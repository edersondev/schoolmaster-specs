# Data Model: Backend Student and Reporting Foundation

## Modeling Principles

- `School` remains the v1 tenant root.
- School-owned records use `school_id` directly or inherit school ownership through a documented same-school parent relationship.
- Public identifiers crossing API boundaries use UUIDs.
- Student self-view is resolved from the authenticated user and active same-school `StudentProfile`; clients do not choose another student profile on self-view endpoints.
- Private teacher content and generated report files must not expose storage paths, scanner internals, or cross-tenant file existence.
- `ReportRun` metadata remains durable after output files expire.

## Entities

### StudentProfile

- **Purpose**: Existing school-owned student profile linked to a user identity for student self-view.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - `user_id`
  - `status` (`active`, `inactive`)
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - belongs to `User`
  - has many `LearningSetAssignment`
  - has many `GradeRecord`
  - has many `AttendanceRecord`
- **Validation rules**:
  - authenticated student self-view requires an active profile in the resolved school
  - inactive or cross-tenant profiles cannot access student self-view endpoints
  - no public student-profile creation or update behavior is introduced in this slice

### LearningSet

- **Purpose**: Existing school-owned instructional sequence for one academic period, visible to a student only through assignment.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - `academic_period_id`
  - `title`
  - `published_at`
  - `status` (`draft`, `published`, `inactive`, `archived`)
- **Relationships**:
  - belongs to `School`
  - belongs to owner `User`
  - belongs to `AcademicPeriod`
  - has many `LearningSetEntry`
  - has many `LearningSetAssignment`
- **Validation rules**:
  - student timeline exposes only same-school learning sets assigned to the authenticated student's active profile
  - student timeline requires the documented academic-period filter
  - timeline-visible learning sets must be `published`
  - ordering is by publish date for learning sets and sequence for entries

### LearningSetEntry

- **Purpose**: Ordered student-visible reference from a learning set to content or a questionnaire.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - `learning_set_id`
  - `entry_type` (`content_item`, `questionnaire`)
  - `entry_reference_id`
  - `sequence`
  - `title`
- **Relationships**:
  - belongs to `School`
  - belongs to `LearningSet`
  - references `TeacherContentItem` or `Questionnaire`
- **Validation rules**:
  - exposed only through an assigned same-school learning set
  - sequence determines student display order inside the learning set
  - content entries expose only student-approved metadata

### LearningSetAssignment

- **Purpose**: Existing direct assignment between a learning set and one selected student profile.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - `learning_set_id`
  - `student_profile_id`
  - `status` (`active`, `inactive`)
  - `assigned_at`
- **Relationships**:
  - belongs to `School`
  - belongs to `LearningSet`
  - belongs to `StudentProfile`
- **Validation rules**:
  - student timeline and content download require an active same-school assignment
  - inactive, missing, or cross-tenant assignments do not authorize student access

### TeacherContentItem

- **Purpose**: Existing school-owned instructional content metadata and private file reference.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - `title`
  - `content_type`
  - `file_size_bytes`
  - `storage_path`
  - `scan_status` (`pending`, `clean`, `failed`)
  - `status` (`draft`, `active`, `inactive`, `archived`)
- **Relationships**:
  - belongs to `School`
  - belongs to owner `User`
  - may be referenced by many `LearningSetEntry`
- **Validation rules**:
  - student metadata includes only fields documented by `TeacherContentStudentMetadata`
  - student download requires same-school ownership, valid assignment, available operational status, and `scan_status = clean`
  - `pending` or `failed` content is not downloadable
  - private storage paths and scan details are never exposed through student responses
- **State transitions relevant to this slice**:
  - `scan_status`: `pending -> clean`, `pending -> failed`
  - content becomes student-downloadable only when assigned and clean

### GradeRecord

- **Purpose**: Existing teacher-recorded grade for a student profile in an academic period.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - `student_profile_id`
  - `academic_period_id`
  - `recorded_by_user_id`
  - `grade_value`
  - `grade_label`
  - `status` (`active`, `inactive`)
  - `recorded_at`
- **Relationships**:
  - belongs to `School`
  - belongs to `StudentProfile`
  - belongs to `AcademicPeriod`
  - belongs to recorder `User`
- **Validation rules**:
  - student self-view returns only records for the authenticated student's same-school profile
  - optional academic-period filter must reference a same-school period
  - inactive records are exposed only if the published contract and authorization rules allow them; otherwise normal lists show active records

### AttendanceRecord

- **Purpose**: Existing teacher-recorded attendance for a student profile in an academic period.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - `student_profile_id`
  - `academic_period_id`
  - `recorded_by_user_id`
  - `attendance_date`
  - `attendance_status` (`present`, `absent`, `late`, `excused`, `remote`, `suspended`)
  - `status` (`active`, `inactive`)
- **Relationships**:
  - belongs to `School`
  - belongs to `StudentProfile`
  - belongs to `AcademicPeriod`
  - belongs to recorder `User`
- **Validation rules**:
  - student self-view returns only records for the authenticated student's same-school profile
  - optional academic-period filter must reference a same-school period
  - attendance status values remain the documented catalog

### AcademicPeriod

- **Purpose**: Existing school-owned period used to filter student timelines and constrain report requests.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - `name`
  - `status`
  - date range fields
- **Relationships**:
  - belongs to `School`
  - has many `LearningSet`
  - has many `GradeRecord`
  - has many `AttendanceRecord`
  - is referenced by `ReportFilters`
- **Validation rules**:
  - student learning timeline requires a same-school academic period
  - student grade and attendance filters, when present, must reference a same-school period
  - report requests require a same-school `academic_period_id`

### ReportRun

- **Purpose**: School-owned asynchronous report request and durable history record.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `requested_by_user_id`
  - `report_type` (`attendance`, `grades`, `academic_structure`, `school_activity`)
  - `filter_summary`
  - `output_formats` (`pdf`, `csv`)
  - `status` (`requested`, `generated`, `failed`, `inactive`)
  - `generated_at` (nullable)
  - `output_expires_at` (nullable)
  - `outputs_available`
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - belongs to requester `User`
  - has generated `ReportOutput` files when status is `generated`
- **Validation rules**:
  - report type must be one of the launch-scope report types
  - filters must include a same-school `academic_period_id`
  - optional filters must be relevant to the selected report type and same-school
  - platform-wide filters are outside this slice
  - report list returns only same-school report runs visible to the requester
- **State transitions**:
  - `requested -> generated`
  - `requested -> failed`
  - `generated -> inactive`
  - output availability changes from available to expired based on `output_expires_at` without deleting `ReportRun` metadata

### ReportOutput

- **Purpose**: Private tenant-scoped generated report file associated with one `ReportRun`.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `report_run_id`
  - `format` (`pdf`, `csv`)
  - `storage_path`
  - `generated_at`
  - `expires_at`
  - `status` (`available`, `expired`, `deleted`)
- **Relationships**:
  - belongs to `School`
  - belongs to `ReportRun`
- **Validation rules**:
  - files are downloadable only when same-school, generated, requested format exists, and unexpired
  - output files expire 90 days after generation
  - expired files return the documented expired-output response
  - expired downloads do not create or mutate output files
  - private paths and storage keys are never exposed through the API

### User

- **Purpose**: Existing actor identity used for authenticated student self-view and school-scoped report permissions.
- **Validation rules in this slice**:
  - inactive users cannot use student or report operations
  - student operations require an active same-school `StudentProfile`
  - report operations require school-scoped report permission
  - platform-scope administration does not imply student self-view or school report access

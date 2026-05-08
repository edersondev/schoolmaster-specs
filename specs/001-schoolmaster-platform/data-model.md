# Data Model: SchoolMaster Platform Foundation

## Modeling Principles

- `School` is the primary tenant boundary for school-scoped records.
- Public and cross-boundary entities use UUID identifiers.
- Business status uses explicit active or inactive state where applicable.
- Historical academic data favors reversible deactivation over destructive
  deletion when records participate in reporting or audit flows.

## Entities

### School

- **Purpose**: Tenant root for school-scoped data and administration.
- **Core fields**:
  - `id` (UUID)
  - `name`
  - `code`
  - `status` (`active`, `inactive`)
  - `contact_email`
  - `contact_phone`
  - `address_summary`
  - `created_at`
  - `updated_at`
- **Relationships**:
  - has many `User`
  - has many `AcademicYear`
  - has many `Guardian`
  - has many `TeacherContentFolder`
  - has many `TeacherContentItem`
  - has many `Questionnaire`
  - has many `LearningSet`
  - has many `GradeRecord`
  - has many `AttendanceRecord`
  - has many `ReportRun`
- **Validation rules**:
  - name is required
  - code is required and unique
  - status is required
- **State transitions**:
  - `inactive -> active` when provisioning is completed
  - `active -> inactive` when the tenant is suspended or archived

### User

- **Purpose**: Authenticated platform identity for all actors.
- **Core fields**:
  - `id` (UUID)
  - `school_id` (nullable for platform-only user)
  - `full_name`
  - `email`
  - `status` (`active`, `inactive`)
  - `last_login_at`
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School` when school-scoped
  - belongs to many `Role`
  - has one `StudentProfile` when actor is student
  - may author many `TeacherContentItem`
  - may author many `Questionnaire`
  - may author many `LearningSet`
- **Validation rules**:
  - email is required and unique within platform identity rules
  - school association is required for school-scoped roles
  - inactive users cannot authenticate into protected workflows
- **State transitions**:
  - `active -> inactive` disables access while retaining history

### Role

- **Purpose**: Named authorization grouping.
- **Core fields**:
  - `id` (UUID)
  - `scope` (`platform`, `school`)
  - `name`
  - `status` (`active`, `inactive`)
- **Relationships**:
  - belongs to many `Permission`
  - belongs to many `User`
- **Validation rules**:
  - name is required
  - scope is required
  - school-scoped roles cannot grant platform-only permissions

### Permission

- **Purpose**: Individual allowed capability.
- **Core fields**:
  - `id` (UUID)
  - `code`
  - `name`
  - `scope` (`platform`, `school`)
- **Relationships**:
  - belongs to many `Role`
- **Validation rules**:
  - code is unique and required
  - scope is required

### AcademicYear

- **Purpose**: Top-level academic cycle per school.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `name`
  - `start_date`
  - `end_date`
  - `status` (`planned`, `active`, `closed`, `inactive`)
- **Relationships**:
  - belongs to `School`
  - has many `AcademicPeriod`
- **Validation rules**:
  - date range is required and valid
  - one school cannot have overlapping active academic years without an
    explicit business rule

### AcademicPeriod

- **Purpose**: Sub-division of an academic year for academic operations.
- **Core fields**:
  - `id` (UUID)
  - `academic_year_id`
  - `school_id`
  - `name`
  - `sequence`
  - `start_date`
  - `end_date`
  - `status` (`planned`, `active`, `closed`, `inactive`)
- **Relationships**:
  - belongs to `AcademicYear`
  - belongs to `School`
  - has many `GradeRecord`
  - has many `AttendanceRecord`
  - has many `LearningSet`
- **Validation rules**:
  - period range must fit within the parent academic year
  - sequence must be unique within the academic year

### StudentProfile

- **Purpose**: School enrollment profile for a student user.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `user_id`
  - `registration_number`
  - `status` (`active`, `inactive`)
  - `current_academic_year_id`
- **Relationships**:
  - belongs to `School`
  - belongs to `User`
  - belongs to many `Guardian` through student-guardian association
  - has many `GradeRecord`
  - has many `AttendanceRecord`
- **Validation rules**:
  - registration number is required and unique within a school
  - student profile requires a school-scoped student user

### Guardian

- **Purpose**: Responsible contact tied to one or more students.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `full_name`
  - `relationship_type`
  - `contact_email`
  - `contact_phone`
  - `status` (`active`, `inactive`)
- **Relationships**:
  - belongs to `School`
  - belongs to many `StudentProfile`
- **Validation rules**:
  - full name is required
  - at least one contact method should be present

### TeacherContentFolder

- **Purpose**: Folder hierarchy for teacher-managed instructional assets.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - `parent_folder_id` (nullable)
  - `name`
  - `status` (`active`, `inactive`)
- **Relationships**:
  - belongs to `School`
  - belongs to `User` as owner
  - may belong to a parent `TeacherContentFolder`
  - has many child `TeacherContentFolder`
  - has many `TeacherContentItem`
- **Validation rules**:
  - folder name is required
  - parent folder must belong to the same school

### TeacherContentItem

- **Purpose**: Uploaded or authored instructional asset.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - `folder_id`
  - `title`
  - `content_type`
  - `storage_reference`
  - `status` (`draft`, `active`, `inactive`, `archived`)
- **Relationships**:
  - belongs to `School`
  - belongs to `User` as owner
  - belongs to `TeacherContentFolder`
  - belongs to many `LearningSet` through ordered entries
- **Validation rules**:
  - title is required
  - folder must belong to the same school
  - uploaded file metadata must meet allowed upload constraints

### Questionnaire

- **Purpose**: Teacher-authored assessment or activity.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - `title`
  - `status` (`draft`, `active`, `inactive`, `archived`)
- **Relationships**:
  - belongs to `School`
  - belongs to `User` as owner
  - belongs to many `LearningSet` through ordered entries
- **Validation rules**:
  - title is required
  - only authorized teachers or administrators may manage questionnaires

### LearningSet

- **Purpose**: Chronological instructional sequence for student consumption.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - `academic_period_id`
  - `title`
  - `status` (`draft`, `published`, `inactive`, `archived`)
- **Relationships**:
  - belongs to `School`
  - belongs to `User` as owner
  - belongs to `AcademicPeriod`
  - has many ordered `LearningSetEntry`
- **Validation rules**:
  - title is required
  - ordered entries must preserve a unique sequence within the learning set
  - referenced items must belong to the same school
- **State transitions**:
  - `draft -> published` when ready for student visibility
  - `published -> inactive` when withdrawn without permanent deletion

### LearningSetEntry

- **Purpose**: Ordered item within a learning set.
- **Core fields**:
  - `id` (UUID)
  - `learning_set_id`
  - `entry_type` (`content_item`, `questionnaire`)
  - `entry_reference_id`
  - `sequence`
- **Relationships**:
  - belongs to `LearningSet`
- **Validation rules**:
  - sequence is required and unique within learning set
  - referenced item type must match the referenced entity

### GradeRecord

- **Purpose**: Student evaluation outcome during an academic period.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `student_profile_id`
  - `academic_period_id`
  - `recorded_by_user_id`
  - `grade_label`
  - `grade_value`
  - `status` (`active`, `inactive`)
  - `recorded_at`
- **Relationships**:
  - belongs to `School`
  - belongs to `StudentProfile`
  - belongs to `AcademicPeriod`
  - belongs to `User` as recorder
- **Validation rules**:
  - student and period must belong to the same school
  - recorder must have permission to manage grades

### AttendanceRecord

- **Purpose**: Student attendance status for an instructional moment.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `student_profile_id`
  - `academic_period_id`
  - `recorded_by_user_id`
  - `attendance_date`
  - `attendance_status`
  - `status` (`active`, `inactive`)
- **Relationships**:
  - belongs to `School`
  - belongs to `StudentProfile`
  - belongs to `AcademicPeriod`
  - belongs to `User` as recorder
- **Validation rules**:
  - attendance date must fall within the relevant academic period or approved
    operational rules
  - recorder must have permission to manage attendance

### ReportRun

- **Purpose**: Generated report request or output for school oversight.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `requested_by_user_id`
  - `report_type`
  - `filter_summary`
  - `status` (`requested`, `generated`, `failed`, `inactive`)
  - `generated_at`
- **Relationships**:
  - belongs to `School`
  - belongs to `User` as requester
- **Validation rules**:
  - requester must have permission to access the selected report type
  - report filters must remain inside the requester's tenant scope

## Relationship Summary

- `School` is the root for tenant-scoped data.
- `AcademicYear` contains many `AcademicPeriod`.
- `User` gains behavior through `Role` and `Permission`.
- `StudentProfile` links student identities to school operations.
- `Guardian` and `StudentProfile` form a many-to-many relationship.
- `LearningSet` is composed of ordered `LearningSetEntry` items that reference
  either `TeacherContentItem` or `Questionnaire`.
- `GradeRecord`, `AttendanceRecord`, and `ReportRun` are school-scoped
  operational records tied to authorization and academic structure.

## Cross-Entity Rules

- School-scoped users may only operate on records owned by their school.
- System administrators may provision or review schools but do not bypass
  module-specific authorization rules without explicit permissions.
- Inactive schools or users cannot participate in normal operational workflows.
- Grades, attendance, learning sets, and reports must align with the selected
  academic period and tenant context.

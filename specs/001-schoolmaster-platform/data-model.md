# Data Model: SchoolMaster Platform Foundation

## Modeling Principles

- `School` is the primary tenant boundary for school-scoped records.
- Public and cross-boundary entities use UUID identifiers.
- Business status uses explicit active or inactive state where applicable.
- Historical academic data favors reversible deactivation over destructive
  deletion when records participate in reporting or audit flows.
- Recoverable tenant-owned records include backend soft-delete support even
  when the public API primarily exposes business status.

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
  - has many `LearningSetAssignment`
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
  - `school_id` (nullable; required for school-scoped roles)
  - `scope` (`platform`, `school`)
  - `name`
  - `status` (`active`, `inactive`)
- **Relationships**:
  - belongs to many `Permission`
  - belongs to many `User`
- **Validation rules**:
  - name is required
  - scope is required
  - school-scoped roles require `school_id`
  - school-scoped roles cannot grant platform-only permissions

### Permission

- **Purpose**: Individual allowed capability.
- **Core fields**:
  - `id` (UUID)
  - `code`
  - `name`
  - `scope` (`platform`, `school`)
  - `status` (`active`, `inactive`)
- **Relationships**:
  - belongs to many `Role`
- **Validation rules**:
  - code is unique and required
  - scope is required
  - inactive permissions cannot be added to new role definitions

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
  - `declared_content_type`
  - `detected_content_type`
  - `file_size_bytes`
  - `scan_status` (`pending`, `clean`, `failed`)
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
  - uploaded files must be PDF, image, text, or office document types
  - uploaded files must not exceed 25 MB
  - executables and archives are rejected
  - uploaded files must not become available until `scan_status` is `clean`

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
  - has many `QuestionnaireQuestion`
- **Validation rules**:
  - title is required
  - only authorized teachers or administrators may manage questionnaires

### QuestionnaireQuestion

- **Purpose**: Individual question inside a teacher-authored questionnaire.
- **Core fields**:
  - `id` (UUID)
  - `questionnaire_id`
  - `question_type` (`multiple_choice`, `true_false`, `short_text`)
  - `prompt`
  - `options` (required for `multiple_choice`, nullable otherwise)
  - `correct_answer` (nullable)
  - `sequence`
- **Relationships**:
  - belongs to `Questionnaire`
- **Validation rules**:
  - prompt is required
  - sequence is required and unique within the questionnaire
  - multiple-choice questions require at least two options
  - question type must be one of the v1 supported types

### LearningSet

- **Purpose**: Chronological instructional sequence for student consumption.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - `academic_period_id`
  - `title`
  - `published_at`
  - `status` (`draft`, `published`, `inactive`, `archived`)
- **Relationships**:
  - belongs to `School`
  - belongs to `User` as owner
  - belongs to `AcademicPeriod`
  - has many ordered `LearningSetEntry`
  - has many `LearningSetAssignment`
- **Validation rules**:
  - title is required
  - ordered entries must preserve a unique sequence within the learning set
  - referenced items must belong to the same school
  - published learning sets require at least one active student assignment
  - student timelines order learning sets by `published_at`
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

### LearningSetAssignment

- **Purpose**: Direct assignment of a learning set to a student profile.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `learning_set_id`
  - `student_profile_id`
  - `assigned_by_user_id`
  - `status` (`active`, `inactive`)
  - `assigned_at`
- **Relationships**:
  - belongs to `School`
  - belongs to `LearningSet`
  - belongs to `StudentProfile`
  - belongs to `User` as assigner
- **Validation rules**:
  - learning set and student profile must belong to the same school
  - student profile must be active when assigned
  - learning set academic period must belong to the student's current school
    context
  - active assignments authorize student metadata visibility for published
    learning sets in the assigned academic period

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
  - grade value must be a numeric decimal from 0 to 100
  - grade label is optional and presentation-only

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
  - attendance status must be one of `present`, `absent`, `late`, `excused`,
    `remote`, or `suspended`

### ReportRun

- **Purpose**: Generated report request or output for school oversight.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `requested_by_user_id`
  - `report_type`
  - `filter_summary`
  - `output_formats` (`pdf`, `csv`)
  - `status` (`requested`, `generated`, `failed`, `inactive`)
  - `generated_at`
  - `output_expires_at`
  - `outputs_available`
- **Relationships**:
  - belongs to `School`
  - belongs to `User` as requester
- **Validation rules**:
  - requester must have permission to access the selected report type
  - report filters must remain inside the requester's tenant scope
  - `academic_period_id` is required for every report request
  - `student_profile_id`, `user_id`, `status`, `start_date`, and `end_date`
    are optional filters where relevant to the selected report type
  - generated outputs are available in PDF and CSV after status is `generated`
  - generated output files expire 90 days after generation while `ReportRun`
    metadata remains available
  - expired output files require a new `ReportRun` with the same filters to
    generate fresh files

## Relationship Summary

- `School` is the root for tenant-scoped data.
- `AcademicYear` contains many `AcademicPeriod`.
- `User` gains behavior through `Role` and `Permission`.
- `StudentProfile` links student identities to school operations.
- `Guardian` and `StudentProfile` form a many-to-many relationship.
- `LearningSet` is composed of ordered `LearningSetEntry` items that reference
  either `TeacherContentItem` or `Questionnaire`.
- `LearningSetAssignment` links a `LearningSet` directly to selected
  `StudentProfile` records for v1 student access.
- `GradeRecord`, `AttendanceRecord`, and `ReportRun` are school-scoped
  operational records tied to authorization and academic structure.

## Cross-Entity Rules

- School-scoped users may only operate on records owned by their school.
- System administrators may provision or review schools but do not bypass
  module-specific authorization rules without explicit permissions.
- Inactive schools or users cannot participate in normal operational workflows.
- Grades, attendance, learning sets, and reports must align with the selected
  academic period and tenant context.
- Teacher content items must remain private until upload validation and malware
  scanning complete successfully.
- Students may download teacher content only when it is in their school, part
  of an assigned learning set, and the content scan status is `clean`.
- Student learning timelines require an academic period filter, order assigned
  learning sets by publish date, and order entries by teacher-defined sequence.
- Report generation is asynchronous for all launch-scope report types; report
  outputs are tenant-bound and available only after the run is `generated`.
- Generated report output files are retained for 90 days after generation, then
  expire while `ReportRun` metadata remains available for audit and history.
- Expired report output files are not regenerated during download; users must
  request a new `ReportRun` with the same filters to generate fresh files.
- P2 questionnaire, grade, attendance, and learning set assignment values must
  use the fixed v1 catalogs and ranges defined in the feature specification.
- School-scoped role assignments are effective only when the role, user, and
  resolved tenant are active and belong to the same school.
- `School`, `User`, `Role`, `AcademicYear`, `AcademicPeriod`,
  `StudentProfile`, `Guardian`, `TeacherContentFolder`, `TeacherContentItem`,
  `Questionnaire`, `QuestionnaireQuestion`, `LearningSet`,
  `LearningSetAssignment`, `GradeRecord`, `AttendanceRecord`, and `ReportRun`
  are recoverable records and use soft-delete support unless a later module
  plan documents an approved permanent deletion path.
- `Permission` records are controlled capability definitions. They use status
  for lifecycle control and are not normally tenant-owned business records.

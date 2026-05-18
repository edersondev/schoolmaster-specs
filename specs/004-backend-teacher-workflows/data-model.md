# Data Model: Backend Teacher Workflow Foundation

## Modeling Principles

- `School` remains the v1 tenant root.
- School-owned records use `school_id` directly or inherit school ownership through a documented same-school parent relationship.
- Public identifiers crossing API boundaries use UUIDs.
- Status fields preserve active/inactive business behavior; recoverable records may use soft deletes behind the public contract.
- Teacher workflows must not introduce class, course, section, group, or roster entities in this slice.
- Uploaded instructional files are private tenant-scoped assets and are unavailable until malware scan status is `clean`.

## Entities

### TeacherContentFolder

- **Purpose**: Internal or pre-existing school-owned organization container for teacher content references in this slice.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - `name`
  - `status` (`active`, `inactive`)
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - belongs to owner `User`
  - has many `TeacherContentItem`
- **Validation rules**:
  - public folder CRUD is outside this slice unless OpenAPI is expanded
  - this slice may validate folder references but must not create public folder lifecycle behavior
  - any submitted `folder_id` must belong to the resolved school and be available to the teacher
  - inactive or cross-tenant folders cannot receive new content
- **State transitions**:
  - `active -> inactive` hides the folder from normal organization while preserving content history

### TeacherContentItem

- **Purpose**: Instructional content metadata and private uploaded file owned by one school.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - `folder_id` (nullable)
  - `title`
  - `content_type` (`pdf`, `image`, `text`, `office_document`)
  - `declared_content_type`
  - `detected_content_type`
  - `file_size_bytes`
  - `storage_path`
  - `scan_status` (`pending`, `clean`, `failed`)
  - `status` (`draft`, `active`, `inactive`, `archived`)
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - belongs to owner `User`
  - optionally belongs to `TeacherContentFolder`
  - may be referenced by many `LearningSetEntry`
- **Validation rules**:
  - `title`, `content_type`, and `file` are required on creation
  - file must be PDF, image, text, or office document up to 25 MB
  - executables and archives are rejected
  - declared and detected content type must be compatible
  - uploaded file is private and tenant-scoped
  - content is unavailable to learning-set or file-access workflows until `scan_status` is `clean`
- **State transitions**:
  - `scan_status`: `pending -> clean`, `pending -> failed`
  - `status`: `draft -> active`, `active -> inactive`, `active -> archived`

### Questionnaire

- **Purpose**: Teacher-authored assessment or activity for instructional use.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - `title`
  - `status` (`draft`, `active`, `inactive`, `archived`)
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - belongs to owner `User`
  - has many `QuestionnaireQuestion`
  - may be referenced by many `LearningSetEntry`
- **Validation rules**:
  - `title` and at least one question are required on creation
  - questionnaire belongs to the resolved school and owner teacher
  - inactive or archived questionnaires cannot be newly added to learning sets
- **State transitions**:
  - `draft -> active`
  - `active -> inactive`
  - `active -> archived`

### QuestionnaireQuestion

- **Purpose**: Ordered question inside a questionnaire.
- **Core fields**:
  - `id` (UUID)
  - `questionnaire_id`
  - `question_type` (`multiple_choice`, `true_false`, `short_text`)
  - `prompt`
  - `options`
  - `correct_answer`
  - `sequence`
- **Relationships**:
  - belongs to `Questionnaire`
- **Validation rules**:
  - `question_type`, `prompt`, and `sequence` are required
  - `multiple_choice` requires at least two options
  - unsupported question types are rejected
  - sequence must be unique within the questionnaire

### LearningSet

- **Purpose**: Ordered instructional sequence for selected students in one academic period.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - `academic_period_id`
  - `title`
  - `published_at`
  - `status` (`draft`, `published`, `inactive`, `archived`)
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - belongs to owner `User`
  - belongs to `AcademicPeriod`
  - has many `LearningSetEntry`
  - has many `LearningSetAssignment`
- **Validation rules**:
  - `academic_period_id`, `title`, at least one entry, and at least one student assignment are required on creation
  - academic period must belong to the resolved school and be active
  - entries must reference same-school available teacher content or questionnaires
  - assigned student profiles must be active and same-school
  - invalid references reject the entire request without partial persistence
- **State transitions**:
  - `draft -> published`
  - `published -> inactive`
  - `published -> archived`

### LearningSetEntry

- **Purpose**: Ordered reference from a learning set to content or a questionnaire.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `learning_set_id`
  - `entry_type` (`content_item`, `questionnaire`)
  - `entry_reference_id`
  - `sequence`
- **Relationships**:
  - belongs to `School`
  - belongs to `LearningSet`
  - references `TeacherContentItem` or `Questionnaire`
- **Validation rules**:
  - referenced item must belong to the same school
  - content item must have `scan_status = clean`
  - referenced item must be active or otherwise available under the published contract
  - sequence must be unique within the learning set

### LearningSetAssignment

- **Purpose**: Direct assignment of a learning set to one selected student profile.
- **Core fields**:
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
  - student profile must be active and same-school
  - duplicate assignment for the same learning set and student profile is rejected
  - assignment is created atomically with the learning set

### GradeRecord

- **Purpose**: Teacher-recorded grade for a student profile in an academic period.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `student_profile_id`
  - `academic_period_id`
  - `recorded_by_user_id`
  - `grade_value`
  - `grade_label`
  - `status` (`active`, `inactive`)
  - `recorded_at`
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - belongs to `StudentProfile`
  - belongs to `AcademicPeriod`
  - belongs to recorder `User`
- **Validation rules**:
  - student profile and academic period must belong to the resolved school
  - student profile must be active
  - academic period must be active for normal creation
  - recorder must be a permitted teacher in the resolved school
  - `grade_value` must be numeric from 0 to 100

### AttendanceRecord

- **Purpose**: Teacher-recorded attendance for a student profile in an academic period.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `student_profile_id`
  - `academic_period_id`
  - `recorded_by_user_id`
  - `attendance_date`
  - `attendance_status` (`present`, `absent`, `late`, `excused`, `remote`, `suspended`)
  - `status` (`active`, `inactive`)
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - belongs to `StudentProfile`
  - belongs to `AcademicPeriod`
  - belongs to recorder `User`
- **Validation rules**:
  - student profile and academic period must belong to the resolved school
  - student profile must be active
  - academic period must be active for normal creation
  - recorder must be a permitted teacher in the resolved school
  - attendance status must match the documented catalog

### AcademicPeriod

- **Purpose**: Existing school-owned period that bounds teacher operations.
- **Validation rules in this slice**:
  - must belong to the resolved school
  - must be active for learning set, grade, and attendance creation
  - closed periods are read-only unless a future correction workflow is specified

### StudentProfile

- **Purpose**: Existing school-owned student profile used as the target of learning set assignment, grades, and attendance.
- **Validation rules in this slice**:
  - must belong to the resolved school
  - must be active for new learning set assignments, grade creation, and attendance creation
  - this slice must not create student self-service behavior or undocumented student-profile APIs

# Data Model: Backend Teacher Workflow Lifecycle and Corrections

## Modeling Principles

- `School` remains the v1 tenant root.
- Affected teacher workflow records are school-owned through `school_id`.
- Public identifiers crossing API boundaries use UUIDs.
- Teacher content, questionnaires, learning sets, grades, and attendance use lifecycle states `active`, `inactive`, and `deleted`.
- Restore transitions return records to `inactive`; activation from `inactive` is a separate documented transition.
- Creating/owning teachers may manage their own records; school administrators may manage same-school records.
- Closed-period grade and attendance corrections are school-administrator-only.
- Grade and attendance correction reasons are required free text from 10 to 500 characters inclusive.
- Grade and attendance imports are create-only, JSON-payload-only, school-administrator-only, capped at 500 rows, and all-or-nothing.
- New learning-set assignment writes use roster membership context; existing direct selected-student assignments remain readable as legacy records.
- Content and questionnaire edits that change historical student-facing meaning are rejected after use.
- Successful and denied teacher content download attempts are audited with tenant-safe metadata only.
- Current student self-view shows `active` records only; `inactive` and `deleted` records are hidden except where OpenAPI explicitly documents historical labels.

## Entities

### TeacherContentItem

- **Purpose**: School-owned instructional content metadata and private uploaded file.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - optional folder reference where already supported
  - title and description metadata
  - content type and declared/detected media metadata
  - private storage reference
  - file size
  - scan status (`pending`, `clean`, `failed`)
  - lifecycle status (`active`, `inactive`, `deleted`)
  - soft-delete metadata where deleted
  - timestamps
- **Relationships**:
  - belongs to `School`
  - belongs to owner `User`
  - may be referenced by `LearningSetEntry`
  - has lifecycle/audit events
- **Validation rules**:
  - detail, update, lifecycle, restore, and download require active permitted school context
  - teacher management requires creator/owner authority; school administrators may manage same-school content
  - downloads require `scan_status = clean`, same-school authority, and an active allowed lifecycle state
  - used content cannot receive edits that change historical student-facing meaning
  - restore returns to `inactive`, not directly to `active`
  - private storage paths and file contents are never returned in JSON responses
- **State transitions**:
  - `active -> inactive`
  - `inactive -> active`
  - `active|inactive -> deleted`
  - `deleted -> inactive`

### Questionnaire

- **Purpose**: School-owned teacher-authored assessment or activity with ordered v1-supported questions.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - title and description metadata
  - lifecycle status (`active`, `inactive`, `deleted`)
  - soft-delete metadata where deleted
  - timestamps
- **Relationships**:
  - belongs to `School`
  - belongs to owner `User`
  - has many `QuestionnaireQuestion`
  - may be referenced by `LearningSetEntry`
  - has lifecycle/audit events
- **Validation rules**:
  - teacher management requires creator/owner authority; school administrators may manage same-school questionnaires
  - submitted question order and supported v1 question types must remain valid
  - used questionnaires cannot receive edits that change historical student-facing meaning
  - restore returns to `inactive`
- **State transitions**:
  - `active -> inactive`
  - `inactive -> active`
  - `active|inactive -> deleted`
  - `deleted -> inactive`

### QuestionnaireQuestion

- **Purpose**: Ordered question inside a questionnaire.
- **Core fields**:
  - `id` (UUID)
  - `questionnaire_id`
  - question type (`multiple_choice`, `true_false`, `short_text`)
  - prompt
  - options where applicable
  - answer metadata where applicable
  - sequence
- **Relationships**:
  - belongs to `Questionnaire`
- **Validation rules**:
  - unsupported question types are rejected
  - sequence is unique within the questionnaire
  - question edits that change historical student-facing meaning are rejected after the questionnaire is used by a published or assigned learning set

### LearningSet

- **Purpose**: School-owned instructional sequence for an academic period.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `owner_user_id`
  - `academic_period_id`
  - title and description metadata
  - lifecycle status (`active`, `inactive`, `deleted`)
  - soft-delete metadata where deleted
  - timestamps
- **Relationships**:
  - belongs to `School`
  - belongs to owner `User`
  - belongs to `AcademicPeriod`
  - has many `LearningSetEntry`
  - has roster-aware `LearningSetAssignment`
  - may have legacy direct selected-student assignments from earlier slices
- **Validation rules**:
  - new assignment writes must use approved roster membership context
  - existing direct selected-student assignments remain readable as legacy records only
  - teacher management requires creator/owner authority and active roster/teacher assignment eligibility where applicable; school administrators may manage same-school learning sets
  - updates that affect student-visible meaning must preserve audit/correction history and obey immutability rules
  - restore returns to `inactive`
- **State transitions**:
  - `active -> inactive`
  - `inactive -> active`
  - `active|inactive -> deleted`
  - `deleted -> inactive`

### LearningSetEntry

- **Purpose**: Ordered reference from a learning set to a content item or questionnaire.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `learning_set_id`
  - entry type (`content_item`, `questionnaire`)
  - entry reference UUID
  - sequence
- **Relationships**:
  - belongs to `School`
  - belongs to `LearningSet`
  - references `TeacherContentItem` or `Questionnaire`
- **Validation rules**:
  - referenced item must be same-school
  - referenced content must be clean before student-facing use
  - inactive, deleted, or historically incompatible references are rejected where OpenAPI documents rejection

### LearningSetAssignment

- **Purpose**: Roster-aware audience relationship for a learning set.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `learning_set_id`
  - class section/roster reference
  - roster membership references or resolved roster membership snapshot where approved
  - lifecycle status (`active`, `inactive`, `deleted`) if exposed as a managed record
  - timestamps
- **Relationships**:
  - belongs to `School`
  - belongs to `LearningSet`
  - references `ClassSection/Roster`
  - references active roster memberships for student audience
- **Validation rules**:
  - new writes use active same-school roster membership context
  - legacy direct selected-student assignments are not created
  - cross-tenant, inactive, deleted, or unauthorized roster references are rejected

### GradeRecord

- **Purpose**: School-owned academic record for one student profile and academic period.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `student_profile_id`
  - `academic_period_id`
  - `recorded_by_user_id`
  - current grade value
  - optional grade label
  - lifecycle status (`active`, `inactive`, `deleted`)
  - soft-delete metadata where deleted
  - timestamps
- **Relationships**:
  - belongs to `School`
  - belongs to `StudentProfile`
  - belongs to `AcademicPeriod`
  - belongs to original recorder `User`
  - has many `CorrectionRecord`
  - has lifecycle/audit events
- **Validation rules**:
  - original recorder, tenant, student, academic period, original value, correction actor history, and audit history are immutable
  - open-period teacher corrections require creator/owner authority
  - closed-period corrections are school-administrator-only and require a reason
  - restore returns to `inactive`
  - current student-visible responses show active records only and hide inactive/deleted records except where OpenAPI explicitly documents historical labels

### AttendanceRecord

- **Purpose**: School-owned attendance record for one student profile, academic period, and attendance date.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `student_profile_id`
  - `academic_period_id`
  - `recorded_by_user_id`
  - attendance date
  - current attendance status (`present`, `absent`, `late`, `excused`, `remote`, `suspended`)
  - lifecycle status (`active`, `inactive`, `deleted`)
  - soft-delete metadata where deleted
  - timestamps
- **Relationships**:
  - belongs to `School`
  - belongs to `StudentProfile`
  - belongs to `AcademicPeriod`
  - belongs to original recorder `User`
  - has many `CorrectionRecord`
  - has lifecycle/audit events
- **Validation rules**:
  - original recorder, tenant, student, academic period, date, correction actor history, and audit history are immutable
  - open-period teacher corrections require creator/owner authority
  - closed-period corrections are school-administrator-only and require a reason
  - restore returns to `inactive`
  - unsupported attendance statuses are rejected
  - current student-visible responses show active records only and hide inactive/deleted records except where OpenAPI explicitly documents historical labels

### CorrectionRecord

- **Purpose**: Tenant-safe history entry for accepted grade and attendance correction workflows.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - target record type and UUID
  - original value reference
  - new value
  - correction reason (free text, 10-500 characters)
  - actor user ID
  - academic period ID where applicable
  - student profile ID where applicable
  - created timestamp
  - student visibility marker where documented
- **Relationships**:
  - belongs to `School`
  - belongs to actor `User`
  - references grade or attendance target where applicable
- **Validation rules**:
  - correction reason is required free text from 10 to 500 characters inclusive
  - prior corrections are never erased
  - private correction notes are not exposed to student self-view unless OpenAPI explicitly documents a safe field
  - cross-tenant target references are rejected before details are exposed

### ImportRun

- **Purpose**: Tenant-safe record of a create-only JSON grade or attendance import attempt.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - actor user ID
  - import type (`grades`, `attendance`)
  - row count
  - status (`accepted`, `rejected`)
  - accepted row count
  - rejected row count
  - tenant-safe error summary
  - created timestamp
- **Relationships**:
  - belongs to `School`
  - belongs to actor `User`
  - may reference accepted `GradeRecord` or `AttendanceRecord` records
  - has audit events
- **Validation rules**:
  - actor must be an authorized school administrator
  - request must be a JSON payload; CSV, spreadsheet, archive, and file-upload imports are outside v1
  - import rows must create new records; rows that target existing records for update or correction are rejected
  - row count must be 500 or fewer
  - all rows validate before any state changes
  - one invalid row rejects the whole import
  - error summaries must not expose unauthorized cross-tenant identifiers or full payloads

### AuditEvent

- **Purpose**: Tenant-safe record of lifecycle, correction, download, import, authorization, validation, and conflict outcomes.
- **Core fields**:
  - `id` (UUID)
  - `school_id` when resolved
  - actor user ID when available
  - target record type and UUID when resolved
  - action
  - outcome
  - reason category where safe
  - tenant-safe metadata summary
  - created timestamp
- **Validation rules**:
  - successful and denied teacher content download attempts are audited
  - updates, lifecycle transitions, delete/restore, imports, import rejection outcomes, corrections, correction rejections, blocked cross-tenant attempts, and conflicts are audited
  - audit data must not store private file contents, private storage paths, credentials, full request payloads, or unauthorized cross-tenant details

## Cross-Entity Rules

- Active permitted school context is resolved before lookup, validation, authorization, persistence, file delivery, audit, or response shaping.
- Platform users do not receive implicit access to school-owned teacher workflow lifecycle or correction behavior.
- Current student self-view sees active records only and permitted historical labels where explicitly documented, never inactive/deleted records as current, private correction notes, or unauthorized actor metadata.
- Concurrent lifecycle, correction, delete/restore, download authorization, and import writes must resolve without partial state and return documented conflict or validation responses.

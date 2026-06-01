# Data Model: Backend Classroom Roster Foundation

## Modeling Principles

- `School` remains the v1 tenant root.
- School-owned roster foundation records use `school_id` as the concrete tenant column.
- Public identifiers crossing API boundaries use UUIDs.
- `ClassSection/Roster` is the primary v1 teaching structure; course, classroom, section, and group values are structured metadata blocks on the same record, each with optional `code` and `name` only, not separate internal tables or lifecycle resources.
- `ClassSection/Roster.code` is required and unique per school and academic period; display names may repeat.
- New `ClassSection/Roster` records start as `active`; inactive creation is rejected, `inactive` is reached only through inactivation, and inactive records cannot be reactivated in v1.
- School administrators manage class sections, roster memberships, and teacher assignments under an active permitted school context.
- Teachers may list and retrieve only their own active teacher assignments where OpenAPI documents that visibility.
- Platform administrators do not receive implicit access to school-owned roster records.
- Roster membership batch changes are all-or-nothing and capped at 100 requested changes per request.
- Effective dates for roster lifecycle changes must fall within the selected academic period and be today-or-past only using the resolved school's configured timezone, with application default timezone fallback only when the school has no configured timezone.
- Roster membership creation and teacher assignment creation require explicit effective start dates.
- Roster membership creation requires active same-school enrollment covering the membership effective start date.
- Teacher assignment creation requires an active same-school teacher-compatible role on the assignment effective start date.
- Roster membership end effective dates and teacher assignment deactivation effective dates must be on or after their effective start dates.
- Lifecycle reasons are required for roster inactivation, roster membership ending, and teacher assignment deactivation.
- Concurrent conflicting roster, membership, teacher-assignment, lifecycle, and dependency-sensitive writes are resolved transactionally so one request succeeds and conflicting requests return a documented conflict response without partial state.
- Existing direct teacher workflow assignments remain readable as legacy records; new roster-aware workflow writes use roster memberships after those workflows are approved.
- Lifecycle transitions preserve audit history and record actor, target record, school, outcome, and tenant-safe metadata.
- List operations use default page size 25 and maximum page size 100, support only `academicPeriodId` and `status` filters, and do not support include expansion or sort behavior unless OpenAPI later documents it.

## Entities

### School

- **Purpose**: Tenant root that owns class sections, roster memberships, teacher assignments, student profiles, academic periods, and related history.
- **Core fields used in this slice**:
  - `id` (UUID)
  - lifecycle status
  - configured timezone where available
  - tenant configuration required for school context resolution
  - `created_at`
  - `updated_at`
- **Relationships**:
  - has many academic periods
  - has many student profiles
  - has many teacher users through school role assignments
  - has many class sections/rosters
  - has many roster memberships through class sections/rosters
  - has many teacher assignments
- **Validation rules**:
  - school-scoped operations require active permitted school context
  - missing, inactive, mismatched, or unauthorized school context fails before roster data lookup, validation, authorization, persistence, or response shaping
  - platform access does not bypass school-owned roster authorization

### AcademicPeriod

- **Purpose**: School-owned period that scopes class sections, roster memberships, and teacher assignments.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - academic year reference
  - name/code from existing academic-period model
  - status
  - start and end dates
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to school
  - has many class sections/rosters
  - constrains roster memberships
  - constrains teacher assignments
- **Validation rules**:
  - active class sections, memberships, and teacher assignments require an active, same-school, compatible academic period
  - closed, inactive, missing, cross-tenant, or incompatible academic periods reject new active roster state unless OpenAPI documents an exception
  - academic period compatibility must respect student enrollment history where membership eligibility depends on it

### ClassSection/Roster

- **Purpose**: Primary school-owned academic-period-scoped teaching structure used to organize roster memberships and teacher assignments.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `academic_period_id`
  - `code` (required; unique per school and academic period)
  - `name` (repeatable display name)
  - optional structured course metadata block with optional `code` and `name` only
  - optional structured classroom metadata block with optional `code` and `name` only
  - optional structured section metadata block with optional `code` and `name` only
  - optional structured group metadata block with optional `code` and `name` only
  - `status` (`active`, `inactive`)
  - lifecycle reason for inactivation
  - `created_by_user_id`
  - `updated_by_user_id`
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to school
  - belongs to academic period
  - has many roster memberships
  - has many teacher assignments
  - has many audit events
- **Validation rules**:
  - creation and update require authorized school administrator permission under active school context
  - creation validates the academic period reference and must create `active` records only; requests to create inactive records are rejected
  - update is metadata-only and accepts documented code, name, and structured course/classroom/section/group metadata fields only
  - `code` must be unique within the same school and academic period
  - names may repeat
  - optional course/classroom/section/group metadata blocks must follow OpenAPI-documented shapes and may include only optional `code` and `name`
  - separate internal Course/Classroom/Section/Group tables are not created in v1
  - lifecycle status is limited to `active` and `inactive`, but lifecycle status changes use the dedicated status operation
  - inactive records cannot be reactivated in v1; administrators create a new roster if needed
  - unsupported lifecycle transitions, inactive-roster reactivation, future effective dates, effective dates outside the selected academic period, missing inactivation reason, and dependency conflicts are rejected
  - inactivation is rejected while any active roster membership or active teacher assignment exists
  - concurrent duplicate-code writes are resolved transactionally so only one request succeeds

### RosterMembership

- **Purpose**: School-owned relationship between a `ClassSection/Roster` and a `StudentProfile`.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `class_section_id`
  - `student_profile_id`
  - `academic_period_id`
  - `status` (`active`, `ended`)
  - effective start date
  - effective end date where ended
  - reason required when ended
  - `created_by_user_id`
  - `ended_by_user_id`
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to school
  - belongs to class section/roster
  - belongs to student profile
  - belongs to academic period
  - has many audit events
- **Validation rules**:
  - creation and ending require authorized school administrator permission under active school context
  - creation requires an explicit effective start date
  - student profile must be active, same-school, not deleted, not transferred out, and backed by active same-school enrollment covering the membership effective start date
  - overlapping active memberships for the same student, roster, and academic period are rejected without changing the existing membership
  - batch membership changes are all-or-nothing and capped at 100 requested changes; one invalid member or oversized batch rejects the entire request without partial changes
  - ending a membership changes current membership state while preserving history
  - ending a membership requires a reason
  - effective dates must fall within the selected academic period and must be today-or-past only using the resolved school's configured timezone, with application default timezone fallback only when the school has no configured timezone
  - effective end date must be on or after the membership effective start date
  - status is limited to `active` and `ended`
  - inactive or transferred students cannot participate in new roster-based workflows where prohibited
  - concurrent conflicting membership writes are resolved transactionally so only one request succeeds

### TeacherAssignment

- **Purpose**: School-owned relationship between a teacher-compatible user and a `ClassSection/Roster` for an academic period.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `class_section_id`
  - `teacher_user_id`
  - `academic_period_id`
  - `status` (`active`, `inactive`)
  - effective start date
  - effective end date where inactive
  - reason required when deactivated
  - `created_by_user_id`
  - `updated_by_user_id`
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to school
  - belongs to class section/roster
  - belongs to academic period
  - belongs to teacher user
  - has many audit events
- **Validation rules**:
  - creation and deactivation require authorized school administrator permission under active school context
  - creation requires an explicit effective start date
  - teacher user must be active, same-school, not deleted, and assigned an active same-school teacher-compatible role on the assignment effective start date
  - target `ClassSection/Roster` and academic period must be active and same-school
  - duplicate active assignments for the same teacher, class section/roster, and academic period are rejected without changing the existing assignment
  - teacher assignment deactivation requires a reason
  - effective dates must fall within the selected academic period and must be today-or-past only using the resolved school's configured timezone, with application default timezone fallback only when the school has no configured timezone
  - effective end date must be on or after the assignment effective start date
  - status is limited to `active` and `inactive`
  - teachers, lead teachers, department managers, students, guardians, and platform administrators without school authorization cannot manage teacher assignments
  - teachers may list and retrieve only their own active assignments where OpenAPI documents that visibility
  - concurrent conflicting teacher-assignment writes are resolved transactionally so only one request succeeds

### StudentProfile

- **Purpose**: Existing school-owned learner profile whose lifecycle and enrollment history determine roster membership eligibility.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - lifecycle status
  - enrollment history references
  - transfer status where applicable
  - soft-delete metadata where applicable
- **Relationships**:
  - belongs to school
  - has many roster memberships
  - has enrollment history
  - may have existing grades, attendance, learning-set assignments, guardian associations, and reports
- **Validation rules**:
  - must be active, same-school, not deleted, not transferred out, and backed by active same-school enrollment covering the membership effective start date before active membership creation
  - historical grades, attendance, learning-set assignments, guardian associations, report references, and audit history remain retained after roster membership changes

### User

- **Purpose**: Authenticated actor and teacher identity used for school administrator management actions and teacher assignment eligibility.
- **Core fields used in this slice**:
  - `id` (UUID)
  - status
  - school role assignments
  - teacher-compatible role marker through existing role/permission model
  - soft-delete metadata where applicable
- **Relationships**:
  - may act as school administrator for management actions
  - may be teacher user in teacher assignments
  - has many audit events as actor
- **Validation rules**:
  - management writes require active user status and school administrator permission
  - teacher assignment target must be active, same-school, not deleted, and have an active same-school teacher-compatible role on the assignment effective start date
  - platform administration does not imply school-owned roster management access

### LegacyDirectAssignment

- **Purpose**: Existing direct relationship between teacher workflow records and selected student profiles.
- **Persistence representation**:
  - represented by existing learning-set assignment, grade, and attendance data from earlier teacher workflow slices
  - no new direct-assignment write model is introduced in this slice
- **Relationships**:
  - relates existing teacher workflow records to student profiles
  - may be visible through existing student self-view and reporting behavior
- **Validation rules**:
  - remains readable for historical compatibility
  - new roster-aware workflow writes must use roster memberships once those workflows are approved
  - migration, backfill, or removal of legacy reads requires a future specification and OpenAPI update

### AuditEvent

- **Purpose**: Tenant-safe record of roster foundation activity.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `actor_user_id`
  - target record type and UUID
  - action
  - outcome
  - lifecycle reason where present
  - metadata summary
  - `created_at`
- **Relationships**:
  - belongs to school
  - belongs to actor user where authenticated
  - may reference class section/roster, roster membership, teacher assignment, student profile, or teacher user
- **Validation rules**:
  - record creation, update, roster inactivation, membership add/end, teacher assignment add/deactivate, duplicate conflict, all-or-nothing batch rejection, blocked cross-tenant attempts, and lifecycle conflict outcomes
  - must include actor user ID when available, action, outcome, lifecycle reason when present, and tenant-safe summary metadata; canonical school ID and target identifiers are required whenever the request resolved them before rejection or completion
  - do not store private student, teacher, credential, full request payload, or unauthorized cross-tenant details beyond documented tenant-safe metadata

## Cross-Entity Rules

- School-scoped roster operations must resolve active permitted school context before lookup, validation, authorization, duplicate checks, persistence, audit creation, or response shaping.
- `ClassSection/Roster.school_id`, `RosterMembership.school_id`, `TeacherAssignment.school_id`, `StudentProfile.school_id`, `TeacherUser.school_id`, and `AcademicPeriod.school_id` must match for write operations.
- `ClassSection/Roster.academic_period_id`, `RosterMembership.academic_period_id`, and `TeacherAssignment.academic_period_id` must remain compatible.
- Batch roster membership changes are atomic across all requested members and reject requests with more than 100 requested changes.
- Duplicate conflict checks must run inside the same school and academic period scope and must be transactionally enforced.
- Historical membership and teacher assignment records remain retained when records become inactive or ended.
- New `ClassSection/Roster` records start as `active`; inactive creation and inactive-roster reactivation are rejected.
- Roster inactivation, roster membership ending, and teacher assignment deactivation require lifecycle reasons.
- Roster membership creation and teacher assignment creation require explicit effective start dates.
- Roster membership creation requires active same-school enrollment covering the membership effective start date.
- Teacher assignment creation requires an active same-school teacher-compatible role on the assignment effective start date.
- Roster membership end effective dates and teacher assignment deactivation effective dates must be on or after their effective start dates.
- Effective dates for lifecycle changes, membership creation, and teacher assignment creation must be inside the selected academic period and cannot be future dates according to the resolved school's configured timezone, with application default timezone fallback only when the school has no configured timezone.
- Concurrent lifecycle or dependency-sensitive writes must not produce partial state.
- List operations use pagination with default page size 25 and maximum page size 100, support only `academicPeriodId` and `status` filters, and reject unsupported filters, include expansion, sort values, and page sizes above 100.
- Inactive schools block school-scoped roster foundation behavior before module-specific validation runs.
- No separate top-level or internal Course/Classroom/Section/Group resource may be exposed or created by this slice.

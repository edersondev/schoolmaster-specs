# Data Model: Backend Student Profile and Enrollment Management

## Modeling Principles

- `School` remains the v1 tenant root.
- School-owned records use `school_id` directly or inherit school ownership through a documented same-school parent relationship.
- Public identifiers crossing API boundaries use UUIDs.
- Student profile lifecycle events are retained for audit and support review.
- Transfer records preserve source-school history and do not move historical academic records across tenants.
- Guardian associations must remain same-school unless a later specification documents a cross-school guardian model.

## Entities

### StudentProfile

- **Purpose**: School-owned learner profile used by guardian associations, teacher assignments, grades, attendance, reports, and student self-view.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `user_id` (nullable when a profile exists before login account linkage)
  - `student_number` or equivalent same-school profile identifier
  - identity fields required by OpenAPI
  - contact or demographic fields required by OpenAPI
  - `enrollment_status` (`active`, `inactive`, `transferred`)
  - `enrolled_at`
  - `status_effective_at`
  - `created_at`
  - `updated_at`
  - soft-delete metadata where recovery is approved by contract
- **Relationships**:
  - belongs to `School`
  - optionally belongs to `User`
  - has many `EnrollmentHistory`
  - has many `GuardianAssociation`
  - has many `LearningSetAssignment`
  - has many `GradeRecord`
  - has many `AttendanceRecord`
  - may be referenced by `ReportRun` filters
- **Validation rules**:
  - creation requires an active resolved school context
  - `student_number` or equivalent identifier must be unique within the resolved school where documented
  - `school_id` cannot be changed through update, status, or transfer operations
  - profile list/detail responses expose only documented fields
  - inactive or transferred profiles cannot participate in new active workflows where the contract prohibits it
- **State transitions**:
  - `active -> inactive` through `updateStudentProfileStatus`
  - `active -> transferred` only through `transferStudentProfile`
  - `inactive -> active` only if OpenAPI explicitly documents reactivation behavior
  - `transferred -> active` is not approved in this slice for the source profile
  - generic status update operations do not set `transferred`

### EnrollmentHistory

- **Purpose**: Append-only school-owned lifecycle record explaining current and historical student profile status.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `student_profile_id`
  - `event_type` (`created`, `activated`, `inactivated`, `transferred_out`, `destination_profile_linked`)
  - `from_status`
  - `to_status`
  - `effective_at`
  - `reason`
  - `actor_user_id`
  - `metadata_summary`
  - `created_at`
- **Relationships**:
  - belongs to `School`
  - belongs to `StudentProfile`
  - belongs to actor `User`
- **Validation rules**:
  - every lifecycle-changing operation writes one history record atomically with the profile change
  - history records are same-school with the source profile
  - history is not publicly editable in this slice
  - metadata must not expose private cross-tenant data

### GuardianAssociation

- **Purpose**: Same-school link between a guardian and student profile.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - `student_profile_id`
  - `guardian_id`
  - `relationship_type`
  - `status` (`active`, `inactive`)
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - belongs to `StudentProfile`
  - belongs to `Guardian`
- **Validation rules**:
  - profile creation may create associations only when every referenced guardian is active and same-school
  - invalid guardian references reject the whole profile creation request
  - guardian self-service access is not introduced in this slice

### Guardian

- **Purpose**: Existing school-owned responsible contact that may be associated with student profiles.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - contact and relationship fields already approved by guardian management contracts
  - `status`
- **Relationships**:
  - belongs to `School`
  - has many `GuardianAssociation`
- **Validation rules**:
  - guardian references in this slice must resolve to active same-school guardians
  - cross-tenant, inactive, missing, or duplicate references fail validation

### StudentTransfer

- **Purpose**: Optional explicit transfer metadata record when transfer details need to be queryable beyond append-only enrollment history.
- **Core fields**:
  - `id` (UUID)
  - `school_id` (source school)
  - `student_profile_id` (source profile)
  - `destination_school_id` (nullable unless contract requires it)
  - `destination_student_profile_id` (nullable)
  - `effective_at`
  - `reason`
  - `actor_user_id`
  - `created_at`
- **Relationships**:
  - belongs to source `School`
  - belongs to source `StudentProfile`
  - may reference destination `School`
  - may reference destination `StudentProfile`
  - belongs to actor `User`
- **Validation rules**:
  - transfer requires active source profile in the resolved source school
  - destination-school behavior requires explicit permission for the destination context
  - transfer must not copy source-school grades, attendance, learning sets, private content, guardian links, or report outputs
  - source profile becomes `transferred` and source history remains retained

### School

- **Purpose**: Tenant root for student profile and enrollment records.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `status`
  - tenant identity fields
- **Relationships**:
  - has many `StudentProfile`
  - has many `EnrollmentHistory`
  - has many `Guardian`
- **Validation rules**:
  - profile operations require an active resolved school
  - destination school references, when documented, must be active and authorized for the requester

### User

- **Purpose**: Authenticated actor and optional student login identity linked to a `StudentProfile`.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `status`
  - role and school relationship data exposed by approved auth/admin contracts
- **Relationships**:
  - may be linked to one or more school-specific `StudentProfile` records
  - may be actor on `EnrollmentHistory`
- **Validation rules**:
  - actor must have school-scoped student administration permission for the resolved school
  - platform-scope access does not imply school-scoped student enrollment authority
  - student self-view eligibility follows the active same-school profile rules already established by `005`

### AcademicPeriod

- **Purpose**: Existing school-owned academic period that may constrain enrollment effective dates where OpenAPI documents it.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - date range fields
  - `status`
- **Relationships**:
  - belongs to `School`
  - may be referenced by enrollment metadata if documented
- **Validation rules**:
  - effective dates must be consistent with documented academic-period rules when the operation references a period
  - this slice does not create class, course, section, or roster membership

## Cross-Entity Rules

- Student profile creation, guardian association creation, and initial enrollment history creation must be atomic.
- Status update and transfer operations must update the profile and write enrollment history atomically.
- Historical `GradeRecord`, `AttendanceRecord`, `LearningSetAssignment`, `ReportRun`, and guardian association references remain owned by the source school after inactivation or transfer.
- No operation in this slice may expose private teacher content, report outputs, or cross-tenant academic history.

# Data Model: Backend Administration Lifecycle Management

## Modeling Principles

- `School` remains the v1 tenant root.
- School-owned records use `school_id` directly or inherit school ownership through a documented same-school parent relationship.
- Public identifiers crossing API boundaries use UUIDs.
- School lifecycle is platform-scoped; school-owned resource lifecycle is school-scoped.
- Delete behavior in this slice means recoverable soft delete.
- Lifecycle transitions preserve dependent history and record actor, reason, effective date, and prior state.
- Cross-tenant access remains deny-by-default unless a future specification documents a support or platform override.

## Entities

### School

- **Purpose**: Platform-managed tenant root whose lifecycle determines whether school-scoped protected workflows may proceed.
- **Core fields used in this slice**:
  - `id` (UUID)
  - tenant identity fields documented by OpenAPI
  - `status` (`active`, `inactive`)
  - soft-delete metadata where approved by contract
  - `created_at`
  - `updated_at`
- **Relationships**:
  - has many school-owned users, roles, academic years, academic periods, guardians, student profiles, teacher records, reports, and lifecycle history
- **Validation rules**:
  - school lifecycle operations require platform-scoped school administration permission
  - inactive schools reject protected school-scoped workflows before module-specific logic runs
  - restoration must validate uniqueness and eligibility before the school becomes available for operational use
- **State transitions**:
  - `active -> inactive` through documented deactivation
  - `inactive -> active` through documented activation
  - `active|inactive -> soft_deleted` through documented recoverable delete
  - `soft_deleted -> active|inactive` through documented restore semantics

### User

- **Purpose**: Authenticated identity that may participate in school-scoped administration through roles and status.
- **Core fields used in this slice**:
  - `id` (UUID)
  - identity fields documented by OpenAPI
  - `status` (`active`, `inactive`)
  - role assignment references
  - school relationship data where school-scoped
  - soft-delete metadata where approved by contract
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to one or more schools through documented school-role assignments
  - has many role assignments
  - may be actor on lifecycle history
  - may be linked to student profile records from prior slices
- **Validation rules**:
  - school-scoped user detail and lifecycle operations require active permitted school context
  - update cannot change immutable public identifier, unauthorized school relationship, or role scope
  - lifecycle operations must not introduce invitation, password setup, password reset, recovery, lock recovery, token refresh, or direct per-user permission assignment behavior
  - deleting or deactivating a user must respect dependency rules for active sessions, role ownership, student profile links, and audit history where OpenAPI documents them

### Role

- **Purpose**: Authorization grouping with platform or school scope and assigned permissions.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id` for school-scoped roles
  - name and description fields documented by OpenAPI
  - `scope` (`platform`, `school`)
  - `status` (`active`, `inactive`)
  - permission assignment references
  - soft-delete metadata where approved by contract
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School` when school-scoped
  - has many permission assignments
  - has many user assignments
- **Validation rules**:
  - role scope cannot be changed through update
  - platform permissions cannot be assigned to school roles through school-scoped operations
  - deactivation or deletion must reject transitions that would leave active users without required authorization state
  - restoration must validate name uniqueness, permission eligibility, and active parent school context

### AcademicYear

- **Purpose**: School-owned academic cycle that constrains academic periods, enrollment context, teacher workflows, student views, and reports.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - name or label
  - start date
  - end date
  - `status` (`active`, `inactive`)
  - soft-delete metadata where approved by contract
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - has many `AcademicPeriod`
  - may be referenced by enrollment, attendance, grade, learning-set, and report workflows
- **Validation rules**:
  - date updates must preserve documented date range rules
  - lifecycle transitions must reject changes that invalidate child periods or historical academic records
  - school ownership cannot change
  - restore must validate active parent school, uniqueness, and date compatibility

### AcademicPeriod

- **Purpose**: School-owned subdivision of an academic year used by teacher, enrollment, student, and report workflows.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - `academic_year_id`
  - name or label
  - sequence
  - start date
  - end date
  - `status` (`active`, `inactive`)
  - soft-delete metadata where approved by contract
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - belongs to `AcademicYear`
  - may be referenced by grades, attendance, learning sets, enrollment history, and reports
- **Validation rules**:
  - parent academic year cannot change across school boundaries
  - period dates must remain inside the parent academic year
  - sequence must remain unique within the parent year where documented
  - lifecycle transitions must reject dependency-blocked changes that would invalidate existing records

### Guardian

- **Purpose**: School-owned responsible contact record associated with same-school student profiles where documented.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - contact and relationship fields documented by OpenAPI
  - `status` (`active`, `inactive`)
  - soft-delete metadata where approved by contract
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - has many same-school guardian-student associations
- **Validation rules**:
  - update cannot change school ownership or create cross-tenant associations
  - lifecycle transitions must preserve historical student profile associations
  - deactivation or deletion may block new guardian association use while retaining authorized history
  - restore must validate active parent school and association eligibility

### LifecycleHistory

- **Purpose**: Audit-relevant record of administrative lifecycle transitions for support review and restore decisions.
- **Core fields**:
  - `id` (UUID)
  - `school_id` for school-owned records, nullable for platform-scoped school lifecycle records where documented
  - `resource_type`
  - `resource_id`
  - `operation` (`updated`, `activated`, `deactivated`, `deleted`, `restored`, `bulk_lifecycle`)
  - `from_status`
  - `to_status`
  - `effective_at`
  - `reason`
  - `actor_user_id`
  - `metadata_summary`
  - `created_at`
- **Relationships**:
  - belongs to actor `User`
  - optionally belongs to `School` for school-owned resource lifecycle
- **Validation rules**:
  - lifecycle-changing operations write history atomically with state changes
  - metadata must not store plaintext credentials, bearer tokens, private content, report output contents, or unauthorized cross-tenant details
  - history is not publicly editable in this slice

### BulkLifecycleRequest

- **Purpose**: Contract-defined request to apply one lifecycle action to a bounded set of same-scope records.
- **Core fields**:
  - resource type
  - action
  - record identifiers
  - effective date where required
  - reason where required
  - actor context derived from authentication
- **Validation rules**:
  - one resource type per request
  - one lifecycle action per request
  - one platform or school scope per request
  - maximum unique record count defined by OpenAPI
  - duplicate identifiers rejected
  - all-or-nothing semantics: any invalid selected record prevents all selected changes

## Cross-Entity Rules

- School-owned operations must resolve active permitted school context before detail, update, lifecycle, dependency, or bulk lookup.
- Platform-scoped school lifecycle operations must not expose school-owned records or grant implicit school-scoped access.
- Updates cannot mutate immutable ownership fields such as `school_id`, role scope, academic parent ownership across tenants, or public UUID identifiers.
- Lifecycle transitions must validate dependency eligibility before state change.
- Soft-deleted records remain retained for authorized history, conflict checks, uniqueness checks, and restore eligibility.
- Restored records must satisfy the same tenant, uniqueness, parent-state, and dependency rules as active records.
- Bulk lifecycle operations must use the same per-record authorization, tenant, validation, dependency, and history rules as individual lifecycle operations.

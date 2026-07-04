# Data Model: Backend School Administration Foundation

## Modeling Principles

- `School` remains the v1 tenant root.
- School-owned records use `school_id` directly or inherit school ownership through a documented same-school parent relationship.
- Public identifiers crossing API boundaries use UUIDs.
- Status fields preserve active/inactive business behavior; recoverable records may use soft deletes behind the public contract.
- Platform-scope and school-scope authorization data must not be mixed.

## Entities

### School

- **Purpose**: Platform-provisioned tenant root.
- **Core fields**:
  - `id` (UUID)
  - `name`
  - `cnpj`
  - `status` (`active`, `inactive`)
  - `contact_email`
  - `contact_phone`
  - `created_at`
  - `updated_at`
- **Validation rules**:
  - `cnpj` is required, unique, stored without punctuation, exactly 14 digits,
    and must pass Brazilian CNPJ check-digit validation
  - masked CNPJ display such as `56.563.930/0001-08` is frontend-only
  - school CNPJ is create-only in the current administration UI
- **State transitions**:
  - `active -> inactive` prevents future school-owned operations while
    preserving historical tenant records

### User

- **Purpose**: Authenticated identity for platform and school actors.
- **Core fields**:
  - `id` (UUID)
  - `school_id` (nullable for platform-only users)
  - `full_name`
  - `email`
  - `status` (`active`, `inactive`)
  - `last_login_at`
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School` when school-scoped
  - belongs to many `Role`
  - may have one `StudentProfile`
- **Validation rules**:
  - `full_name` is required on creation
  - `email` is required and must not create ambiguous platform identity
  - school-scoped users require a valid resolved school context
  - submitted roles must be active, compatible with school scope, and owned by the same school where applicable
- **State transitions**:
  - `active -> inactive` disables protected workflows while preserving history

### Role

- **Purpose**: Named authorization grouping for platform or school capabilities.
- **Core fields**:
  - `id` (UUID)
  - `school_id` (nullable for platform roles, required for school roles)
  - `scope` (`platform`, `school`)
  - `name`
  - `status` (`active`, `inactive`)
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School` for school-scoped roles
  - belongs to many `Permission`
  - belongs to many `User`
- **Validation rules**:
  - `scope`, `name`, and `permission_ids` are required on creation
  - school roles require the resolved active school context
  - role permissions must be active and compatible with the role scope
  - school roles cannot include platform-only permissions
- **State transitions**:
  - `active -> inactive` prevents future assignment while preserving historical user-role context

### Permission

- **Purpose**: Capability definition assigned through roles.
- **Core fields**:
  - `id` (UUID)
  - `code`
  - `name`
  - `scope` (`platform`, `school`)
  - `status` (`active`, `inactive`)
- **Relationships**:
  - belongs to many `Role`
- **Validation rules**:
  - `code` is unique and required
  - only active permissions are available for new role definitions
  - direct per-user permission assignment is not allowed in v1

### AcademicYear

- **Purpose**: Top-level academic cycle for one school.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `name`
  - `start_date`
  - `end_date`
  - `status` (`planned`, `active`, `closed`, `inactive`)
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - has many `AcademicPeriod`
- **Validation rules**:
  - `name`, `start_date`, and `end_date` are required
  - `end_date` must be after `start_date`
  - academic year belongs to the resolved active school
  - overlapping active-year behavior must follow the platform specification and OpenAPI before exposed as configurable behavior
- **State transitions**:
  - `planned -> active`
  - `active -> closed`
  - any non-historical state may become `inactive` when removed from operational use

### AcademicPeriod

- **Purpose**: Ordered subdivision of an academic year used by later grades, attendance, learning sets, and reports.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `academic_year_id`
  - `name`
  - `sequence`
  - `start_date`
  - `end_date`
  - `status` (`planned`, `active`, `closed`, `inactive`)
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - belongs to `AcademicYear`
- **Validation rules**:
  - `academic_year_id`, `name`, `sequence`, `start_date`, and `end_date` are required
  - parent academic year must belong to the same resolved school
  - period date range must fit within the parent academic year
  - `sequence` must be unique within the parent academic year
- **State transitions**:
  - `planned -> active`
  - `active -> closed`
  - any non-historical state may become `inactive` when removed from operational use

### Guardian

- **Purpose**: Responsible contact record associated with one or more students in the same school.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `full_name`
  - `relationship_type`
  - `contact_email`
  - `contact_phone`
  - `status` (`active`, `inactive`)
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to `School`
  - belongs to many `StudentProfile`
- **Validation rules**:
  - `full_name` and `relationship_type` are required
  - contact fields must match the OpenAPI schema when present
  - student profile references, when provided, must be active and owned by the same school
  - invalid student profile references reject the entire guardian create request
- **State transitions**:
  - `active -> inactive` preserves association history while removing the guardian from normal operational use

### StudentProfile

- **Purpose**: School-owned student enrollment profile referenced by guardian association.
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
  - belongs to many `Guardian`
- **Validation rules**:
  - this slice may validate existing `StudentProfile` identifiers for guardian association
  - this slice must not create student self-service behavior or undocumented student profile APIs

### Tenant Context

- **Purpose**: Active school scope required for school-owned administration workflows.
- **Core fields**:
  - resolved `school_id`
  - authenticated user identity
  - requester roles and permissions
  - school status
- **Validation rules**:
  - missing, inactive, mismatched, or unauthorized context rejects the request before school-owned data access
  - platform access is not an implicit school-scope bypass

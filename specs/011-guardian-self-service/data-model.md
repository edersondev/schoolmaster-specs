# Data Model: Backend Guardian Self-Service

## Modeling Principles

- `School` remains the v1 tenant root.
- Guardian self-service records and source records are school-owned through `school_id`.
- Public identifiers crossing API boundaries use UUIDs.
- Guardian self-service requires both an explicit school-administrator-created guardian-user link and an active school-approved guardian-student association.
- Guardian self-service is read-only in v1.
- Guardian academic visibility is summary-only and requires an explicit same-school academic period.
- Guardian contact visibility is limited to the authenticated guardian's own contact fields, school-approved relationship labels, and the student's primary school-approved contact details.
- Target-specific requests for missing, unassociated, inactive, or cross-tenant students return the same not-found envelope.
- Guardian responses never expose school-only notes, private correction reasons, full correction history, internal actor metadata, report outputs, private teacher content, private file paths, malware-scan internals, other guardian records, non-primary student contact details, or unauthorized cross-tenant identifiers.

## Entities

### Guardian

- **Purpose**: Existing school-owned responsible contact record.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - name fields
  - contact fields documented as guardian-owned
  - lifecycle status (`active`, `inactive`, `deleted` where supported by administration lifecycle)
  - timestamps
- **Relationships**:
  - belongs to `School`
  - may have one or more `GuardianUserLink` records
  - may have many `GuardianStudentAssociation` records
- **Validation rules**:
  - guardian self-service requires an active same-school guardian record
  - inactive or deleted guardian records cannot access current guardian self-service views
  - guardian-owned contact fields may be returned only through approved guardian contact view responses
  - other guardian records are hidden from a guardian actor

### GuardianUserLink

- **Purpose**: School-owned proof that an authenticated user account may act as a specific guardian in one school context.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `guardian_id`
  - `user_id`
  - status (`active`, `inactive`, `deleted` where supported)
  - created_by_user_id
  - created_at
  - deactivated_at where applicable
- **Relationships**:
  - belongs to `School`
  - belongs to `Guardian`
  - belongs to authenticated `User`
  - created by a school administrator `User`
- **Validation rules**:
  - link must be explicitly created by a school administrator
  - link must be active, same-school, and point to an active user and active guardian
  - permitted same-school school administrators must be able to create and deactivate the link through a documented admin contract path
  - automatic contact matching and invitation completion cannot create or substitute for this link in v1
  - missing or inactive link denies guardian self-service before student data is returned

### GuardianStudentAssociation

- **Purpose**: School-approved same-school relationship between a guardian and a student profile.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `guardian_id`
  - `student_profile_id`
  - relationship label or type
  - approval/status (`active`, `inactive`, `deleted` where supported)
  - timestamps
- **Relationships**:
  - belongs to `School`
  - belongs to `Guardian`
  - belongs to `StudentProfile`
- **Validation rules**:
  - association must be active, same-school, and linked to the authenticated guardian through `GuardianUserLink`
  - inactive or deleted associations deny current guardian visibility
  - destination-school guardian access after transfer requires a separate active destination-school association
  - source-school current access ends when a student is transferred unless OpenAPI explicitly documents limited historical labels

### StudentProfile

- **Purpose**: Existing school-owned learner profile whose limited summary may be guardian-visible.
- **Core fields visible to guardian self-service**:
  - `id` (UUID)
  - documented display name fields
  - documented enrollment summary fields
  - documented status label where guardian-visible
  - primary school-approved contact details where contact view is requested
- **Relationships**:
  - belongs to `School`
  - has guardian associations
  - has academic records through grades, attendance, learning-set sources, and academic periods
- **Validation rules**:
  - guardian visibility requires an active same-school `GuardianStudentAssociation`
  - target-specific requests for missing, unassociated, inactive, or cross-tenant students return the same not-found envelope
  - inactive, transferred, or deleted student profiles are hidden from current guardian views unless OpenAPI explicitly documents limited historical labels
  - profile detail responses do not expose disciplinary records, school-only notes, report outputs, private file references, or internal identifiers not in contract

### AcademicPeriod

- **Purpose**: Existing school-owned period required to scope guardian academic summaries.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - name or label
  - status
  - date range
- **Relationships**:
  - belongs to `School`
  - referenced by `GradeSummary`, `AttendanceSummary`, and `LearningSetSummary`
- **Validation rules**:
  - academic summary requests require an explicit same-school academic period
  - missing, inactive, unsupported, cross-tenant, or unauthorized periods are rejected according to OpenAPI
  - period validation occurs before academic summary records are exposed

### GradeSummary

- **Purpose**: Guardian-facing summary derived from existing school-owned grade records.
- **Core fields**:
  - `student_profile_id` (UUID)
  - `academic_period_id` (UUID)
  - documented current grade summary values
  - documented status labels where approved
  - generated or calculated timestamp where approved
- **Relationships**:
  - derived from same-school active grade records
  - scoped to one `StudentProfile` and one `AcademicPeriod`
- **Validation rules**:
  - detailed grade rows are not guardian-visible in v1
  - private correction reasons, prior values, full correction history, original recorder metadata, and internal actor metadata are hidden
  - summaries include only current active records for the permitted student and academic period

### AttendanceSummary

- **Purpose**: Guardian-facing summary derived from existing school-owned attendance records.
- **Core fields**:
  - `student_profile_id` (UUID)
  - `academic_period_id` (UUID)
  - documented attendance totals or status values
  - generated or calculated timestamp where approved
- **Relationships**:
  - derived from same-school active attendance records
  - scoped to one `StudentProfile` and one `AcademicPeriod`
- **Validation rules**:
  - detailed attendance rows are not guardian-visible in v1
  - private correction reasons, prior values, full correction history, original recorder metadata, and internal actor metadata are hidden
  - summaries include only current active records for the permitted student and academic period

### LearningSetSummary

- **Purpose**: Limited guardian-facing summary of student learning progress or status.
- **Core fields**:
  - `student_profile_id` (UUID)
  - `academic_period_id` (UUID)
  - documented progress/status values where approved
  - documented counts or labels where approved
- **Relationships**:
  - derived from active same-school learning sets or assignments visible for the permitted student
  - scoped to one `StudentProfile` and one `AcademicPeriod`
- **Validation rules**:
  - no private teacher content downloads
  - no questionnaire answer keys
  - no student activity submission capabilities
  - no private teacher-only notes, file paths, or malware-scan details

### ContactVisibilityRule

- **Purpose**: Contract-governed rule set for guardian-facing contact fields.
- **Visible field groups**:
  - authenticated guardian's own documented contact fields
  - school-approved relationship labels
  - student's primary school-approved contact details
- **Hidden field groups**:
  - other guardian records
  - non-primary student contact details
  - restricted emergency handling details
  - school-only contact notes
  - custody, legal, or dispute records
- **Validation rules**:
  - field visibility must be encoded in OpenAPI schemas before backend exposure
  - contact responses require active guardian-user link and active guardian-student association
  - no contact update behavior is included in v1

### AuditEvent

- **Purpose**: Tenant-safe record of guardian self-service access and denial outcomes.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - actor user ID when available
  - guardian ID when safely resolved
  - target student profile ID when safely resolved
  - action
  - outcome
  - safe reason category
  - timestamp
  - tenant-safe summary metadata
- **Relationships**:
  - belongs to `School`
  - may reference actor `User`
  - may reference `Guardian`
  - may reference `StudentProfile`
- **Validation rules**:
  - audit entries do not store private payloads, school-only notes, credentials, file paths, report outputs, or unauthorized cross-tenant details
  - denied attempts and blocked cross-tenant attempts are audited with safe reason categories only

### School

- **Purpose**: Tenant root that controls guardian, student, academic, contact, and audit visibility.
- **Core fields**:
  - `id` (UUID)
  - tenant status
  - school metadata
- **Relationships**:
  - owns guardian records, user links, associations, student profiles, academic periods, academic records, contact fields, and audit events through `school_id`
- **Validation rules**:
  - every guardian self-service operation requires an active permitted school context
  - missing, mismatched, inactive, or unauthorized school context fails before school-owned data access
  - platform users do not receive implicit guardian self-service or support visibility

## State and Visibility Transitions

| Record | Current visible state | Hidden/denied states | Notes |
|--------|-----------------------|----------------------|-------|
| GuardianUserLink | `active` | inactive, deleted, missing, cross-tenant | Must be explicitly created by a school administrator |
| Guardian | `active` | inactive, deleted, missing, cross-tenant | Required before any guardian self-service response |
| GuardianStudentAssociation | `active` | inactive, deleted, missing, cross-tenant | Required per target student |
| StudentProfile | active same-school current profile | inactive, transferred, deleted, missing, cross-tenant, unassociated | Target-specific denied cases return same not-found envelope |
| AcademicPeriod | explicit active same-school period where supported | missing, inactive, unsupported, cross-tenant, unauthorized | Required for academic summary requests |
| Academic summaries | active current summary values | detailed rows, correction history, inactive/deleted source records | Derived summary only |
| Contact view | guardian-owned contact, relationship label, primary student contact | other guardians, non-primary contacts, restricted notes | Read-only in v1 |

## Validation Summary

- Resolve active permitted school context before lookup, authorization, summary aggregation, contact shaping, or response shaping; audit-safe denial recording may still occur before return when only request metadata and safe tenant context details are available.
- Validate authenticated active user and explicit active same-school `GuardianUserLink`.
- Validate active same-school `GuardianStudentAssociation` before returning target-specific student information.
- Validate explicit same-school academic period for academic summary requests.
- Apply uniform not-found envelope for missing, unassociated, inactive, or cross-tenant target students.
- Reject unsupported filters, includes, sort values, undocumented request fields, invalid payload shapes, inactive references, deleted references, cross-tenant references, and unsupported periods.
- Shape responses only through OpenAPI-approved field sets and API Resources.

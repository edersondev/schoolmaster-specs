# Data Model: Student Guardian Tabs

This feature updates existing school-owned student and guardian records. No new
database table is planned.

## StudentCreateWorkflow

**Purpose**: Route-local administrator workflow for creating one student and
optional guardian associations.

**Fields**:

- `studentDraft`: Student fields approved by `StudentProfileCreateRequest`.
- `guardianEntries`: Ordered collection of zero, one, or two `GuardianEntry`
  records.
- `activeTab`: Student or Guardians.
- `tabErrors`: Student and Guardians error summaries.
- `status`: Idle, loading, ready, submitting, succeeded, validation-error,
  forbidden, tenant-mismatch, inactive-school, conflict, unavailable, or error.
- `isDirty`: True when student or guardian fields differ from initial state.
- `requestKey`: Active school, route, and submit attempt identity used to ignore
  stale results.

**Rules**:

- Requires authenticated session and active permitted school context.
- Student tab remains available to actors with student create authority.
- Guardians tab capture is available only when guardian management authority is
  present.
- Submitted payload contains zero to two guardian entries.
- A successful submission clears dirty state before navigation or success
  feedback.
- Draft values survive tab switching and validation failures.

## StudentDraft

**Purpose**: Approved student profile fields captured in the Student tab.

**Fields**:

- `userId`
- `registrationNumber`
- `firstName`
- `lastName`
- `dateOfBirth`
- `contactEmail`
- `contactPhone`
- `currentAcademicYearId`
- `status`
- `enrolledAt`

**Rules**:

- Required fields follow the published student creation contract:
  registration number, first name, last name, and enrolled date.
- Student fields are validated independently but submitted with guardian
  entries as one request.
- Tenant ownership is never entered by the user.

## GuardianEntry

**Purpose**: One guardian entry captured in the Guardians tab.

**Fields**:

- `entryId`: Local-only stable key for tab rendering and error mapping.
- `mode`: New guardian or existing guardian.
- `relationshipType`: Required relationship label for the student association.
- `newGuardian`: `NewGuardianDraft` when mode is new guardian.
- `existingGuardian`: `ExistingGuardianReference` when mode is existing
  guardian.
- `fieldErrors`: Entry-scoped validation feedback.

**Rules**:

- Exactly one mode is selected per entry.
- Relationship type is required for both modes.
- Duplicate relationship labels are allowed across the two entries.
- Maximum two entries per student create workflow.
- Duplicate existing guardian references are rejected.
- A third entry cannot be added or submitted.

## NewGuardianDraft

**Purpose**: New same-school guardian record to create during student creation.

**Fields**:

- `fullName`
- `contactEmail`
- `contactPhone`

**Rules**:

- Full name is required.
- Contact email must be valid when supplied.
- Contact phone must match approved guardian contact rules when supplied.
- Backend validation remains authoritative for duplicate contact conflicts.

## ExistingGuardianReference

**Purpose**: Active same-school guardian selected for association to the new
student.

**Fields**:

- `guardianId`
- `displayName`
- `relationshipType`
- `contactSummary`
- `status`

**Source**: Approved `listGuardians` response in the active school context.

**Rules**:

- Only active same-school guardians can be selected.
- Lookup requests use documented guardian list filters and pagination only.
- Selected references submit UUIDs, not display labels.
- Cross-school, inactive, missing, deleted, or duplicate references reject the
  whole student create request.

## GuardianStudentAssociation

**Purpose**: School-owned relationship between the created student and one
guardian.

**Fields**:

- `studentProfileId`
- `guardianId`
- `relationshipType`
- `status`

**Rules**:

- Created only after the student and guardian records pass validation.
- At most two active associations are created for the student through this
  workflow.
- Creation is atomic with student creation and any new guardian records.
- No association crosses school boundaries.

## GuardianRouteRedirect

**Purpose**: Frontend routing behavior for former standalone guardian
administration routes.

**Fields**:

- `sourceRoute`: Former guardian list, create, detail, lifecycle, or bulk route.
- `targetRoute`: Create Student.
- `preservedContext`: Active school and safe return/navigation context only.

**Rules**:

- Sidebar metadata does not expose Guardians.
- Direct route access redirects to Create Student.
- Redirect does not load guardian list data.
- Unauthorized or missing-school state follows existing protected-route and
  active-school behavior.

## SafeFeedback

**Purpose**: Normalized user and diagnostic feedback for tabbed student creation
with guardians.

**Fields**:

- `type`: Validation, unauthorized, forbidden, tenant-mismatch,
  inactive-school, not-found, conflict, maximum-guardians,
  duplicate-guardian, temporary-unavailable, stale, or unknown.
- `tab`: Student, Guardians, or workflow.
- `fieldErrors`
- `messageKey`
- `operationId`
- `requestId`

**Rules**:

- Field errors must identify the affected tab and control.
- Diagnostics may include safe operation and request identifiers only.
- Feedback must not expose full payloads, contact details, tokens, permission
  payloads, or cross-tenant existence.

## State Transitions

```text
StudentCreateWorkflow:
idle -> ready
ready -> submitting
submitting -> succeeded
submitting -> validation-error
submitting -> forbidden
submitting -> tenant-mismatch
submitting -> conflict
submitting -> unavailable
validation-error -> ready
forbidden -> ready
tenant-mismatch -> ready
conflict -> ready
unavailable -> ready
ready -> discarded
```

```text
GuardianEntry:
empty -> new-guardian
empty -> existing-guardian
new-guardian -> removed
existing-guardian -> removed
new-guardian -> validation-error
existing-guardian -> validation-error
```

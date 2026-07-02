# Research: Student Self-Service UI

## Decision: Consume only approved student self-service OpenAPI contracts

**Rationale**: The aggregate contract already includes student learning-set
timeline, student teacher-content download, student grade list, and student
attendance list operations. This UI slice can deliver useful student value
without new backend behavior.

**Alternatives considered**:

- Add frontend-only routes to inferred backend behavior: rejected because
  OpenAPI is the contract source of truth.
- Reuse teacher, guardian, or report routes for student views: rejected because
  those routes carry different actor authority and visibility rules.
- Add student report or transcript behavior now: rejected because no
  student-facing report contract is approved in this slice.

## Decision: Use current active academic period only

**Rationale**: The student learning-set timeline requires an academic period,
and the clarified scope excludes manual period switching. Using current active
academic period avoids inventing a period-picker source or allowing arbitrary
query parameters.

**Alternatives considered**:

- Manual period picker: rejected until a student-approved period source and UX
  contract exists.
- Free URL/query `academic_period_id`: rejected because it encourages
  undocumented period selection and extra edge cases.
- Unscoped grade and attendance defaults: rejected because the overview and
  current student workspace should stay aligned to current active context.

## Decision: Show distinct no-student-profile state

**Rationale**: Authenticated accounts can lack an active linked student profile
in the active school. A distinct state gives clear recovery and prevents
student self-service requests that would otherwise rely on backend denials for
expected account-link absence.

**Alternatives considered**:

- Treat as forbidden: rejected because it hides a recoverable profile-link
  problem behind a generic denial.
- Show empty workspace: rejected because it falsely implies the student has no
  records rather than no active profile.

## Decision: Default student workspace opens Assigned Learning Sets

**Rationale**: Assigned Learning Sets is the P1 student story and required
contract scope. It is the main student task and the source for authorized
content access.

**Alternatives considered**:

- Default to academic overview: rejected because overview is P4 and limited to
  counts/statuses.
- Restore last student route: rejected for this slice because it adds
  persistence and stale authorization complexity not required by the spec.

## Decision: Keep questionnaire entries read-only

**Rationale**: The student learning-set response may include questionnaire
entries, but this feature excludes response submit/review UI. Read-only
display preserves learning-set sequence without expanding into assessment
workflow scope.

**Alternatives considered**:

- Add basic response submit/review: rejected because it changes scope,
  testing, and authorization boundaries.
- Hide questionnaire entries: rejected because it would make returned
  learning-set sequence incomplete.

## Decision: Limit academic overview to counts and statuses

**Rationale**: Approved student self-service responses expose lists and
student-visible fields. Calculated GPA, attendance rate, rankings, trend
analytics, transcripts, or report outputs are not approved for this slice.

**Alternatives considered**:

- Calculate grade average and attendance rate client-side: rejected because it
  invents academic meaning not defined by contract.
- Hide overview entirely: rejected because counts/statuses can provide safe
  orientation without new backend behavior.

## Decision: Use route-local composables and loaded-list-backed detail views

**Rationale**: Student screens need stale-response protection and safe detail
behavior without standalone detail endpoints. Route-local composables keep
state minimal and avoid storing sensitive student data globally.

**Alternatives considered**:

- Pinia store for all student records: rejected because it persists more
  student data across routes than needed.
- Direct service calls in route components: rejected by service isolation and
  component-boundary rules.
- Standalone detail requests: rejected because no standalone student
  learning-set, grade, or attendance detail route is approved.

## Decision: Verify safe diagnostics as part of feature acceptance

**Rationale**: Student self-service flows handle student identity, academic
records, content metadata, file availability, and tenant boundaries.
Diagnostics may include safe operation IDs and generic failure categories, but
must not persist private file paths, storage keys, token values, role internals,
scan internals, guardian data, other-student data, or cross-tenant details.

**Alternatives considered**:

- Rely on existing logging conventions only: rejected because this feature
  introduces student-specific file and academic-record surfaces.
- Check only denied states: rejected because successful and empty states also
  carry student context.

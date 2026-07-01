# Research: Teacher Workflow Workspace

## Decision: Consume approved teacher workflow OpenAPI contracts and gate missing scope support through contract review

**Rationale**: The aggregate OpenAPI already includes teacher content,
questionnaire, learning-set, grade, attendance, teacher assignment, download,
correction, restore, status, delete, and import operations needed by much of
this UI slice. Contract review also shows that roster-aware learning-set
creation and scoped learning-set, grade, and attendance list filters must be
added or explicitly confirmed before those UI controls ship. The frontend must
not use undocumented filters or direct selected-student learning-set writes.

**Alternatives considered**:

- Add frontend-specific endpoints without OpenAPI: rejected because the
  constitution requires OpenAPI-first behavior.
- Implement client-side filtering over broad unscoped lists: rejected because
  teacher workspace screens default to explicit period and roster scope and
  should not depend on broad data loads.
- Implement UI against inferred backend shapes: rejected because frontend must
  consume only published contracts.

## Decision: Add a distinct teacher workspace module and keep admin observation under admin-system

**Rationale**: Teacher-facing workflows have different navigation, default
scope, and action authority than school-administrator observation. A teacher
module keeps route ownership clear while admin-only observation, imports, and
closed-period corrections reuse existing admin-system patterns.

**Alternatives considered**:

- Put all screens under admin-system: rejected because teacher-facing routes
  should not appear as administration screens.
- Build one shared route tree for both roles: rejected because it increases
  permission and action leakage risk.

## Decision: Default teacher workspace scope to current active academic period and active teacher rosters

**Rationale**: Feature 022 established academic-period route/query state and
teacher-assignment context. Defaulting teacher workspace screens to the current
active period and active teacher rosters gives teachers a useful first view
while avoiding broad unscoped teacher data loads.

**Alternatives considered**:

- Require manual period and roster selection before any data loads: rejected
  because it slows the common active-period workflow.
- Load all teacher-owned records across periods by default: rejected because it
  weakens scope clarity, increases stale-response risk, and makes tests harder
  to anchor to roster authorization.

## Decision: Route-local composables coordinate sensitive workflow state

**Rationale**: Teacher workflow data includes student, teacher, file, grade,
attendance, correction, and import details. Route-local composables keep pages
thin, centralize stale-response protection, and avoid persisting sensitive
state in broad Pinia stores.

**Alternatives considered**:

- One Pinia store for the teacher workspace: rejected because it would persist
  more sensitive details than necessary and blur service boundaries.
- Direct service calls in pages: rejected by frontend architecture and service
  isolation rules.

## Decision: Admin-observed screens are read/detail plus admin-only imports and closed-period corrections

**Rationale**: The feature wording says admin-observed, not full duplicate
teacher management. Restricting administrators to read/detail observation,
imports, and closed-period corrections prevents action leakage while preserving
approved administrative oversight.

**Alternatives considered**:

- Full same-school administrator management for all records: rejected because
  it could override teacher ownership beyond the clarified scope.
- Import/correction only with no observation: rejected because administrators
  need context to interpret import and correction outcomes.

## Decision: Grade and attendance imports use structured JSON payload entry only

**Rationale**: Backend lifecycle rules approve create-only JSON imports with a
500-row maximum and all-or-nothing validation. UI should mirror that contract
and avoid building CSV, spreadsheet, archive, or file-upload import behavior
that is outside v1.

**Alternatives considered**:

- CSV or spreadsheet upload: rejected because contracts do not approve those
  formats.
- Manual row-entry table only: rejected because it obscures the JSON payload
  contract and adds complex editing behavior not required by the feature.

## Decision: Exclude student questionnaire response review and grading

**Rationale**: This workspace covers questionnaire authoring and learning-set
attachment. Student response review, grading, and response file downloads are
separate assessment workflows and would expand actor, data visibility, and
verification requirements beyond this feature.

**Alternatives considered**:

- Include response review and grading where routes exist: rejected because it
  mixes assessment response workflows into the teacher workspace foundation.
- Show response summaries only: rejected because even summary visibility
  requires separate response-scope product decisions.

## Decision: Safe diagnostics are a feature verification requirement

**Rationale**: Teacher workflow screens handle students, teacher identifiers,
private file metadata, correction history, and import payloads. Diagnostics may
include safe operation and request identifiers, but must not persist or print
student, teacher, guardian, token, permission, role, private file path, full
upload metadata, full import payload, correction private note, or cross-tenant
details.

**Alternatives considered**:

- Rely on existing logging conventions only: rejected because this feature
  adds file, correction, and import flows with sensitive payloads.
- Check only denial states: rejected because successful and validation flows
  also handle sensitive values.

## Decision: Reuse existing shell, CRUD, lifecycle, denial, and conflict patterns

**Rationale**: Features 016 through 022 established protected layout,
list/detail/form composition, status tags, confirmation dialogs, denial
mapping, conflict handling, service isolation, route/query scope restoration,
and no-sensitive-data diagnostics. Reuse keeps teacher and administrator
surfaces consistent and limits new primitives to teacher scope, uploads,
downloads, corrections, and JSON imports.

**Alternatives considered**:

- Build standalone teacher UI patterns: rejected because it would duplicate
  existing foundations.
- Put business rules into components: rejected because rules must come from
  specifications, OpenAPI, services, and composables.

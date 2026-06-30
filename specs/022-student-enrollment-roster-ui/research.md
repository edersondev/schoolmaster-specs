# Research: Student Enrollment and Classroom Roster UI

## Decision: Consume existing student and roster OpenAPI contracts without frontend-driven contract changes

**Rationale**: The aggregate OpenAPI already includes student profile,
student status, transfer, class-section, roster membership, and
teacher-assignment operations. The UI slice can proceed as contract
consumption if planning confirms those operations satisfy the feature.

**Alternatives considered**:

- Add new frontend-specific endpoints: rejected because OpenAPI already
  documents the required resource surfaces and the constitution requires
  contract-first behavior.
- Implement UI against inferred backend shapes: rejected because frontend must
  consume only published contracts.

## Decision: Student profile UI supports list, create, detail, status, and transfer only

**Rationale**: Approved OpenAPI operations include `listStudentProfiles`,
`createStudentProfile`, `getStudentProfile`, `updateStudentProfileStatus`, and
`transferStudentProfile`. There is no approved general
`updateStudentProfile` operation. The UI must not expose profile edit controls
that would require undocumented behavior.

**Alternatives considered**:

- Add general edit UI now: rejected because it would force undocumented
  transport behavior.
- Block all student mutations: rejected because create, status, and transfer
  are approved operations.

## Decision: Keep teacher-facing own-assignment screens out of feature 022

**Rationale**: Roadmap item 7 is administrative. Teacher-facing work belongs to
the Teacher Workflow Workspace. This feature may consume teacher-assignment
operations for school administrator management, but it must not add a teacher
route or workspace.

**Alternatives considered**:

- Add teacher read-only assignment page: rejected because it belongs to a
  later teacher-facing feature.
- Build shared services only with no admin assignment UI: rejected because the
  current feature explicitly includes teacher assignment admin screens.

## Decision: Default roster and assignment lists to the current active academic period

**Rationale**: Roster and teacher-assignment operations are academic-period
scoped and support `academicPeriodId` filtering. Defaulting to the current
active period gives administrators a useful first view while route/query state
preserves explicit period selection for refresh, sharing, and browser
navigation.

**Alternatives considered**:

- Require period selection before loading: rejected because it slows the common
  active-period workflow.
- Load all periods by default: rejected because contracts filter by period and
  cross-period lists are harder to scan and test safely.

## Decision: Expose all-or-nothing membership batch add and end UI capped at 100 requested changes

**Rationale**: The backend contract approves batch add and batch end operations
with `maxItems: 100` and all-or-nothing rejection semantics. The UI should
match the contract directly, prevent oversized selections where possible, and
show whole-request failure clearly when any selected membership is invalid.

**Alternatives considered**:

- Single-student membership controls only: rejected because it hides approved
  batch behavior and makes administrator workflow inefficient.
- Batch add only: rejected because the contract also approves batch ending
  with required reason and effective date.

## Decision: Route-local composables coordinate forms, lists, period scope, and batch state

**Rationale**: Student, roster, membership, transfer, and teacher-assignment
state is route-specific and sensitive. Composables keep route pages thin,
centralize stale-response protection, and avoid creating a broad Pinia store
for state that does not need to survive route changes.

**Alternatives considered**:

- One Pinia store for all student and roster state: rejected because it would
  persist more sensitive detail than necessary and blur service boundaries.
- Direct service calls in pages: rejected by frontend architecture and service
  isolation rules.

## Decision: Safe diagnostics are a feature verification requirement

**Rationale**: Student profile, guardian, roster, teacher, permission, and
cross-tenant data are sensitive school-owned records. Diagnostics may include
safe operation and request identifiers, but must not persist or print student,
guardian, token, permission, role, full request payload, reason, or
cross-tenant details.

**Alternatives considered**:

- Rely on existing logging conventions only: rejected because this feature
  handles multiple sensitive student and roster flows.
- Check only denial states: rejected because successful and validation flows
  also handle sensitive payloads and field values.

## Decision: Reuse existing admin CRUD, lifecycle, denial, and conflict patterns

**Rationale**: Features 016 through 021 established the admin shell, list,
detail, form, confirmation dialog, status tag, denial mapping, conflict
handling, service isolation, and no-secret diagnostics patterns. Reuse keeps
administrator workflows consistent and limits new UI primitives to roster
batch and academic-period scope needs.

**Alternatives considered**:

- Build standalone student/roster UI patterns: rejected because it would
  duplicate existing admin foundations.
- Put business rules into components: rejected because rules must come from
  specifications, OpenAPI, services, and composables.

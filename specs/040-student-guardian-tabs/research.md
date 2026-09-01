# Research: Student Guardian Tabs

## Decisions

### Use student creation as the atomic guardian capture boundary

Extend `createStudentProfile` so the tabbed Create Student workflow can submit
student fields plus zero to two guardian entries in one request.

**Rationale**: The user asked for guardian creation in the same Create Student
form. A single backend transaction prevents partial success where the student is
created but one guardian or association fails.

**Alternatives considered**:

- Frontend calls `createStudentProfile`, then `createGuardian` once or twice:
  rejected because failures can leave partial state and require compensating
  cleanup.
- Add a new workflow-only endpoint: rejected because the existing
  `createStudentProfile` boundary already owns atomic student creation and
  guardian associations.
- Keep standalone Guardian creation as the primary path: rejected because the
  sidebar item is removed by product decision.

### Support both new guardian creation and existing guardian linking

Each guardian entry may either define a new guardian or reference an active
same-school guardian returned by the approved guardian list operation.

**Rationale**: Siblings and reused guardian contacts are common. Linking avoids
duplicating same-school guardian records while still allowing new guardian
capture during enrollment.

**Alternatives considered**:

- New guardians only: rejected because it duplicates records for families with
  multiple students.
- Existing guardians only: rejected because it blocks first-time family entry
  inside Create Student.

### Enforce zero to two guardians at contract and backend validation

Guardian entries are optional, but a student create request may include no more
than two guardian entries.

**Rationale**: Optional guardians keep enrollment possible when information is
not yet available. Contract-level and backend validation make the product limit
consistent for all clients, not only the SPA.

**Alternatives considered**:

- Require at least one guardian: rejected because schools may enroll a student
  before guardian details are known.
- Enforce the maximum only in the frontend: rejected because hidden clients or
  stale builds could bypass the limit.

### Allow duplicate relationship labels

Allow two guardian entries for the same student to use the same approved
relationship label.

**Rationale**: Relationship labels describe the guardian relationship, not
guardian identity. Two guardians can validly share a label in real school data,
while duplicate identity or duplicate existing guardian references remain
separate validation concerns.

**Alternatives considered**:

- Reject duplicate relationship labels: rejected because it blocks valid family
  and custody cases.
- Warn on duplicate labels: rejected because it adds friction without changing
  a valid submission.

### Require guardian management authority for guardian capture

Student-only creation remains available for actors with student create
authority, but creating or linking guardian entries requires guardian
management authority.

**Rationale**: Guardian contact records are privacy-sensitive school-owned
records. Keeping guardian capture behind guardian management permission matches
existing administration permission boundaries.

**Alternatives considered**:

- Let student creation authority imply guardian capture: rejected because it
  broadens guardian data access without an explicit permission.
- Block Create Student entirely when guardian permission is missing: rejected
  because the clarified feature allows zero guardians.

### Redirect standalone guardian admin routes to Create Student

Remove Guardians from sidebar and redirect bookmarked or direct standalone
guardian administration routes to the approved Create Student workflow.

**Rationale**: Redirecting gives administrators a clear current workflow and
keeps old links from becoming confusing dead ends.

**Alternatives considered**:

- Show a safe unavailable page: rejected because it adds a dead end where a
  clear destination exists.
- Keep direct guardian routes fully accessible: rejected because it weakens the
  product decision to move guardian capture into student creation.

### Keep route-local tabbed form state

The frontend Create Student page owns Student and Guardians tab state locally,
with service-isolated API calls and existing session/shell stores for auth,
permissions, navigation, and active school.

**Rationale**: The draft is short-lived, bound to one route, and should be
discarded or submitted as one workflow. A feature store is unnecessary unless
implementation discovers cross-route state needs.

**Alternatives considered**:

- Persist draft state globally: rejected because it increases sensitive data
  lifetime and adds recovery behavior not requested.
- Put HTTP calls in tab components: rejected because frontend architecture
  requires service isolation.

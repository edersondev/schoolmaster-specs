# Feature Specification: Student Enrollment and Classroom Roster UI

**Feature Branch**: `022-student-enrollment-roster-ui`
**Created**: 2026-06-30
**Status**: Draft
**Input**: User description: "Implement feature 022-student-enrollment-roster-ui: Student Enrollment and Classroom Roster UI for schoolmaster-frontend. Define student profile, enrollment, class-section roster, membership, teacher assignment, and academic-period scoped admin screens. Consume only approved student-enrollment and roster OpenAPI operations. Specify transfer behavior, lifecycle status display, assignment constraints, authorization denials, empty states, validation errors, pagination, and conflict handling. Follow existing admin shell, CRUD foundation, and lifecycle UI patterns from completed frontend features 016 through 021."

## Clarifications

### Session 2026-06-30

- Q: Should student profile UI include general profile edit without an approved update operation? → A: Student profile UI supports list, create, detail, status, and transfer only; no general profile edit.
- Q: Should feature 022 include teacher-facing own-assignment screens? → A: Admin-only UI; teacher own-assignment screens deferred to Teacher Workflow Workspace.
- Q: Should roster membership UI expose batch operations? → A: Batch membership UI required for add and end, capped at 100 per request.
- Q: What academic period should roster and assignment lists use by default? → A: Default to current active academic period; route/query stores selected period.
- Q: What diagnostic safety verification is required for sensitive student and roster flows? → A: Require no-sensitive-data diagnostics verification for student, roster, membership, transfer, and assignment flows.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Manage student profiles and enrollment state (Priority: P1)

A school administrator manages student profiles inside the active school context, reviews current enrollment state, and performs approved enrollment lifecycle actions without seeing or changing another school's records.

**Why this priority**: Student profiles and enrollment status are the foundation for roster membership, guardian visibility, teacher workflows, student summaries, and later academic records.

**Independent Test**: Can be fully tested by signing in as an authorized school administrator, opening student administration, listing same-school students, creating or viewing a student profile, applying approved filters and pagination, submitting valid and invalid profile creation or status changes, and confirming cross-school, unauthorized, inactive, not-found, validation, and conflict states render correctly.

**Acceptance Scenarios**:

1. **Given** an authorized administrator has an active school context, **When** they open the student profile list, **Then** they see only students from that school with documented pagination, empty state, status display, and supported filters.
2. **Given** no student profiles match the current filter or school context, **When** the list loads successfully, **Then** the UI shows an actionable empty state without implying missing permissions or cross-school data.
3. **Given** an administrator creates an approved student profile, **When** validation fails, **Then** field-level and summary feedback identify the correct inputs without submitting undocumented fields.
4. **Given** an administrator changes enrollment lifecycle status using an approved operation, **When** the action succeeds, **Then** the updated status, effective date, and history cues are visible in the student detail view.
5. **Given** a student is inactive, transferred, missing, unauthorized, or outside the active school, **When** the administrator opens or acts on the profile, **Then** the UI shows the documented denial, not-found, inactive, or conflict state without exposing protected student details.

---

### User Story 2 - Record student transfers safely (Priority: P2)

A school administrator records approved student transfer behavior from the student detail surface while preserving source-school history and avoiding cross-tenant data exposure.

**Why this priority**: Transfers affect roster eligibility, active enrollment status, guardian visibility, teacher assignment context, and historical academic records. The UI must make transfer outcomes clear before roster operations depend on them.

**Independent Test**: Can be fully tested by opening an active same-school student profile, submitting an approved transfer action with valid and invalid destination, effective date, and reason inputs, and verifying success, validation, unauthorized destination, conflict, and history-preservation states.

**Acceptance Scenarios**:

1. **Given** an active same-school student is eligible for transfer, **When** an authorized administrator starts transfer, **Then** the UI shows required effective date, reason, and approved destination inputs without exposing source-school academic records to another school.
2. **Given** transfer input is valid and permitted, **When** the transfer is submitted, **Then** the UI marks the source profile as transferred, preserves history visibility for authorized source-school users, and removes the student from new active same-school workflow eligibility where required.
3. **Given** transfer input references an unauthorized, missing, inactive, or incompatible destination, **When** the transfer is submitted, **Then** the UI shows documented validation or denial feedback and keeps the current visible profile state unchanged.
4. **Given** a transferred student appears in student or roster screens, **When** an administrator reviews the record, **Then** the UI clearly distinguishes historical visibility from current active eligibility.

---

### User Story 3 - Maintain class sections and roster membership (Priority: P3)

A school administrator creates and maintains academic-period-scoped class sections or rosters, assigns eligible students to rosters, ends memberships with required lifecycle details, and sees batch conflicts before partial changes are shown.

**Why this priority**: Roster membership connects enrollment to teacher workflows and classroom operations. Administrators need reliable structure, membership, and status feedback before teachers can depend on roster-owned work.

**Independent Test**: Can be fully tested by creating or opening a roster for an active academic period, adding eligible students in a batch capped at 100 requested changes, attempting duplicate or invalid memberships, ending memberships in a batch capped at 100 requested changes, changing roster status where approved, and verifying all-or-nothing batch behavior, pagination, empty states, and conflicts.

**Acceptance Scenarios**:

1. **Given** an active academic period exists in the current school, **When** an administrator creates or updates an approved class section or roster, **Then** the UI captures only approved teaching-structure fields and shows status, period, and lifecycle constraints.
2. **Given** roster records exist across schools or academic periods, **When** an administrator lists rosters, **Then** the list defaults to the current active academic period, stores any selected period in the route or query state, and shows only current-school records visible to the actor with approved academic-period and status filtering.
3. **Given** an administrator adds up to 100 students to a roster in one batch, **When** one or more selected students are inactive, transferred, cross-school, duplicate, already active in an overlapping membership, or not actively enrolled for the effective date, **Then** the UI shows documented validation or conflict feedback and does not show partial success for the all-or-nothing request.
4. **Given** an administrator ends up to 100 roster memberships in one batch, **When** required effective date and reason are submitted, **Then** the UI updates current membership state while preserving historical membership visibility for all successfully ended memberships.
5. **Given** roster inactivation is blocked by active memberships or teacher assignments, **When** the administrator attempts inactivation, **Then** the UI shows conflict feedback and guides the administrator to resolve active dependencies first.

---

### User Story 4 - Assign teachers to class sections or rosters (Priority: P4)

A school administrator assigns eligible same-school teachers to class sections or rosters for an academic period and reviews assignment status. Teacher-facing own-assignment screens are deferred to Teacher Workflow Workspace.

**Why this priority**: Teacher assignment rules define who can operate against roster-scoped future workflows. The admin UI must enforce eligibility, duplicate prevention, and denied-state clarity before teacher-facing features expand.

**Independent Test**: Can be fully tested by assigning an eligible teacher to a roster, attempting assignment with inactive or cross-school teachers, duplicate assignments, incompatible periods, missing reasons, or unauthorized actors, and verifying admin-visible states plus deferred teacher route absence.

**Acceptance Scenarios**:

1. **Given** an authorized school administrator opens roster assignment controls, **When** they search or select teachers, **Then** the UI offers only approved same-school teacher assignment interactions and shows eligibility feedback for unavailable choices.
2. **Given** a teacher is active and eligible for the target roster and academic period, **When** an administrator submits assignment with required effective details, **Then** the assignment appears with active status and period context in the current selected academic-period view.
3. **Given** the assignment would duplicate an active same-teacher, same-roster, same-period assignment or uses an invalid effective date, **When** it is submitted, **Then** the UI shows documented conflict or validation feedback without changing visible assignment state.
4. **Given** a teacher assignment is deactivated where approved, **When** the administrator provides required lifecycle information, **Then** the UI shows inactive assignment state and preserves assignment history.
5. **Given** an actor lacks teacher-assignment management authority, **When** they open roster or assignment surfaces, **Then** management controls are hidden or disabled and forbidden responses show safe denial feedback.

### Edge Cases

- Active school context is missing, inactive, changed during a request, or no longer authorized while student, roster, membership, transfer, or teacher-assignment screens are open.
- Student, roster, membership, transfer, or teacher-assignment responses arrive after the user changes filters, pagination, route, active school, or authentication state.
- Lists return no records because of true empty state, restrictive filters, inactive academic period, or lack of authorization; each case needs distinct safe messaging where contracts allow it.
- No current active academic period exists for the school; roster and assignment lists must show a blocked period-selection state before loading scoped records.
- Student profile creation conflicts with duplicate same-school identifiers, invalid guardian references, unsupported lifecycle state, immutable tenant ownership, or stale record state.
- Transfer input references an unauthorized destination school, incompatible student state, invalid effective date, missing reason, or attempted cross-tenant history exposure.
- Roster membership requests include inactive, transferred, cross-school, deleted, duplicate, overlapping, or not-enrolled students; all-or-nothing batch behavior must stay clear.
- Roster membership add or end selection exceeds 100 requested changes; the UI must prevent submission or show documented oversized-batch feedback.
- Roster membership or teacher assignment effective dates are future dates, outside the selected academic period, or earlier than the record start date.
- Roster inactivation is attempted while active memberships or teacher assignments remain.
- Teacher assignment requests include inactive users, users without teacher-compatible role coverage, cross-school teachers, duplicate active assignments, or unauthorized actors.
- Contract responses return validation, unauthorized, forbidden, tenant-mismatch, inactive-school, not-found, conflict, temporary-unavailable, unsupported filter, unsupported sort, or unsupported page-size states.
- Diagnostics, client-side errors, and test output must not persist or print student, guardian, token, permission, role, full request payload, or cross-tenant details for student, roster, membership, transfer, or teacher-assignment flows.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No new backend behavior is included in this frontend feature. Backend student-enrollment and classroom-roster operations must already be approved, implemented, and verified before frontend runtime exposure.
- **Frontend repository impact**: Implement student administration, enrollment lifecycle, transfer, class-section roster, roster membership, and teacher-assignment admin screens in the existing protected administration experience, reusing established list, detail, form, confirmation, status, denial, and conflict patterns.
- **Specification or contract repository impact**: This specification defines the frontend consumption boundary. A frontend contract artifact must map every UI surface to approved student-enrollment and roster operations, response states, filters, pagination, and authorization behavior. Aggregate OpenAPI changes are required only if approved backend contracts are missing or incompatible.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines this UI boundary first, `schoolmaster-frontend` implements after plan and tasks, and `schoolmaster-backend` changes are allowed only through separate backend or contract work if this specification identifies a missing approved operation.

### API Contract Impact

- **OpenAPI update required**: No planned change for this UI slice if the approved student-enrollment and roster operations already cover required behavior. Yes only if contract review finds missing list, detail, create, update, lifecycle, transfer, membership, teacher-assignment, filter, pagination, or error semantics.
- **Versioned endpoints affected**: Frontend may consume only approved `/api/v1` student profile, enrollment lifecycle, transfer, class-section or roster, roster membership, and teacher-assignment operations documented before implementation begins.
- **JSON response impact**: UI behavior depends on documented success, paginated, validation, unauthorized, forbidden, tenant-mismatch, inactive-school, inactive-record, not-found, conflict, unsupported filter, unsupported sort, unsupported page-size, and temporary-unavailable envelopes. No UI may depend on undocumented fields, status codes, or inferred backend state.
- **Authentication/authorization impact**: All protected screens require authenticated access, active permitted school context, and approved school-scoped student or roster administration permissions. Platform access does not grant implicit school-owned student, roster, membership, transfer, or teacher-assignment management.
- **Compatibility impact**: Frontend delivery is additive. It must not change existing authentication, account lifecycle, administration lifecycle, guardian, teacher workflow, student self-service, reporting, or legacy direct-assignment behavior unless a separate approved specification changes those areas.

### Data & Tenancy Impact

- **Tenant scoping impact**: Student profiles, enrollment history, transfer metadata, class sections or rosters, roster memberships, teacher assignments, academic-period context, and related status display are school-owned and scoped to the active permitted school.
- **Cross-tenant or platform access impact**: Cross-school transfer display must show only approved destination and source metadata. Source-school academic records, private student details, guardian links, learning sets, reports, and teacher assignments remain isolated unless a separate approved contract permits exposure.
- **Soft delete impact**: Soft-deleted, inactive, transferred, or ended records may appear only where approved for history and audit review. Permanent deletion, purge, merge, anonymization, restore, and bulk import are outside this UI slice.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The UI MUST expose only approved student profile, enrollment lifecycle, transfer, class-section or roster, roster membership, and teacher-assignment behavior documented before implementation begins.
- **FR-002**: The UI MUST require an authenticated user and active permitted school context before protected student enrollment or roster screens load data or enable actions.
- **FR-003**: The UI MUST show student profile lists with documented pagination, supported filters, lifecycle status display, loading state, empty state, validation feedback, and safe denial states.
- **FR-004**: The UI MUST allow authorized administrators to list, create, and view student profiles, plus apply approved status and transfer actions. General student profile edit controls MUST remain hidden or blocked unless OpenAPI adds an approved student profile update operation before implementation.
- **FR-005**: The UI MUST display enrollment lifecycle status, effective dates, reason or history cues where approved, and current active eligibility for student records.
- **FR-006**: The UI MUST support approved student transfer workflows, including required transfer inputs, success state, validation feedback, conflict feedback, unauthorized destination feedback, and source-school history preservation cues.
- **FR-007**: The UI MUST clearly distinguish active, inactive, transferred, deleted or unavailable, and historical-only student states anywhere students appear in enrollment or roster workflows.
- **FR-008**: The UI MUST list and manage class sections or rosters only within the active school and approved academic-period scope, default roster and assignment lists to the current active academic period, preserve an explicit selected period in route or query state, and use documented filters, pagination, status display, and empty states.
- **FR-009**: The UI MUST allow authorized administrators to create or update approved class-section or roster fields and display lifecycle constraints, including blocked inactivation when active dependencies remain.
- **FR-010**: The UI MUST allow authorized administrators to add eligible students to rosters and end roster memberships through approved all-or-nothing batch membership behavior capped at 100 requested changes per request.
- **FR-011**: The UI MUST display roster membership status, effective dates, required reason prompts for ending memberships, historical membership visibility, all-or-nothing batch rejection feedback, oversized-batch feedback, and validation for academic-period date constraints.
- **FR-012**: The UI MUST prevent or safely reject unsupported roster membership actions for inactive, transferred, cross-school, duplicate, overlapping, deleted, or not-actively-enrolled students according to documented responses.
- **FR-013**: The UI MUST allow authorized school administrators to assign eligible same-school teachers to class sections or rosters, display assignment status and academic-period context, and show validation or conflict feedback for duplicate, inactive, cross-school, or incompatible assignments.
- **FR-014**: The UI MUST display teacher-assignment deactivation controls only where approved and require documented lifecycle information before submitting deactivation.
- **FR-015**: The UI MUST separate student management, roster management, roster membership management, teacher assignment management, platform administration, guardian access, student self-service, reporting visibility, and deferred teacher own-assignment visibility according to approved authorization behavior.
- **FR-016**: The UI MUST show safe unauthorized, forbidden, tenant-mismatch, inactive-school, inactive-record, not-found, validation, conflict, unsupported filter, unsupported sort, unsupported page-size, and temporary-unavailable states without exposing cross-tenant or protected details.
- **FR-017**: The UI MUST ignore or cancel stale list, detail, transfer, membership, and assignment results when the user changes route, filter, pagination, active school, or authentication state before the response is applied.
- **FR-018**: The UI MUST keep existing admin shell, navigation, CRUD foundation, lifecycle, confirmation, status, denial, and conflict behavior consistent with completed frontend features 016 through 021.
- **FR-019**: The UI MUST NOT introduce teacher workflow correction, guardian self-service, student self-service, report lifecycle, report output, bulk import, billing, messaging, notification-center, permanent purge, or undocumented API behavior in this slice.
- **FR-020**: The UI MUST include test coverage or equivalent verification for successful flows, validation failures, authorization denials, tenant isolation denials, empty states, pagination, stale responses, transfer conflicts, membership conflicts, teacher-assignment conflicts, academic-period constraints, safe error display, and no-sensitive-data diagnostics behavior.

### Key Entities *(include if feature involves data)*

- **StudentProfile**: School-owned learner profile shown to authorized administrators, including identity summary, enrollment status, active eligibility, guardian-compatible summary where approved, and lifecycle history cues.
- **EnrollmentHistory**: Timeline or status context that explains student creation, activation, inactivation, transfer, effective dates, and reasons where approved for display.
- **StudentTransfer**: Approved transfer action and resulting state for a source-school student profile, including effective date, reason, destination metadata where permitted, and historical source-school visibility.
- **ClassSection/Roster**: School-owned academic-period-scoped teaching structure used to group students and teachers, with status, code or name summary, optional approved metadata, and lifecycle constraints.
- **RosterMembership**: Relationship between a student profile and a class section or roster, including active or ended status, effective dates, reason where required, eligibility feedback, all-or-nothing batch add or end behavior capped at 100 requested changes, and historical visibility.
- **TeacherAssignment**: Relationship between an eligible teacher and a class section or roster for an academic period, including active or inactive status, effective dates, reason where required, and actor visibility.
- **AcademicPeriod**: School-owned period that scopes enrollment eligibility, rosters, memberships, teacher assignments, filtering, and effective-date validation; roster and assignment list screens default to the current active period and preserve explicit selection in route or query state.
- **SchoolContext**: Active permitted school boundary used to load, filter, authorize, and display school-owned student and roster records.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of student, enrollment, transfer, roster, membership, and teacher-assignment UI actions can be traced to approved contract behavior before frontend implementation begins.
- **SC-002**: In usability checks, an authorized school administrator can find a student, identify current enrollment status, and open the student detail view in under 2 minutes without assistance.
- **SC-003**: In usability checks, an authorized school administrator can create or update an approved roster, add or end a batch of eligible memberships capped at 100 requested changes, and identify rejected memberships in under 5 minutes without assistance.
- **SC-004**: 100% of tested unauthorized, forbidden, tenant-mismatch, inactive-school, inactive-record, not-found, validation, conflict, and temporary-unavailable responses show safe feedback without exposing cross-school protected details.
- **SC-005**: 100% of tested pagination and filter combinations for student, roster, membership, and teacher-assignment lists show stable loading, empty, populated, unsupported-state, current-period-default, and selected-period-restoration behavior.
- **SC-006**: 100% of tested stale responses caused by route, filter, pagination, active-school, or authentication changes do not overwrite the current visible screen state.
- **SC-007**: At least 90% of representative administrators can correctly distinguish active, inactive, transferred, ended, historical-only, and conflict states after completing the main workflows.
- **SC-008**: Review confirms no UI surface exposes undocumented student self-service, guardian self-service, teacher correction, reporting, bulk import, billing, messaging, permanent purge, or cross-tenant data behavior.
- **SC-009**: Diagnostics verification confirms student, guardian, token, permission, role, full request payload, and cross-tenant details do not appear in client-side diagnostics, error logs, or automated test output for student, roster, membership, transfer, or teacher-assignment flows.

## Assumptions

- Backend student profile and enrollment rules from `006-backend-student-enrollment` and classroom roster rules from `009-classroom-roster-foundation` are the source of approved domain behavior for this frontend slice.
- Student enrollment and roster operations are school-owned and require active permitted school context.
- Frontend implementation will block or hide actions until approved permissions or capability flags are confirmed during planning and implementation.
- Student transfers preserve source-school history and do not copy source-school academic records, private content, guardian links, or report outputs across schools.
- Roster membership eligibility depends on active same-school student enrollment covering the membership effective start date.
- Teacher assignment eligibility depends on an active same-school teacher-compatible user state covering the assignment effective start date.
- Academic-period filters and effective-date constraints follow approved backend contract behavior.
- Schools normally have one current active academic period for roster administration; if none is available, the UI blocks roster and assignment scoped loading until an approved period is selected or created elsewhere.
- Existing admin shell, authentication, administration foundation, administration lifecycle, and account lifecycle UI behavior from completed frontend features 016 through 021 remains the baseline.
- Teacher-facing own-assignment screens, teacher-facing workflow workspace, student self-service, guardian self-service, reporting workspace, bulk import, billing, messaging, and notification features remain outside this slice.

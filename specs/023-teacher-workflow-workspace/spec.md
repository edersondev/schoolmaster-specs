# Feature Specification: Teacher Workflow Workspace

**Feature Branch**: `023-teacher-workflow-workspace`
**Created**: 2026-07-01
**Status**: Ready for Planning
**Input**: User description: "Specify Teacher Workflow Workspace frontend feature. Define teacher-facing and admin-observed screens for content, questionnaires, learning sets, grades, attendance, correction history, downloads, and imports where product scope approves them. Scope must reuse existing admin list/detail/form/status patterns, consume only approved teacher workflow API routes, and document upload, lifecycle, correction, download, empty-state, authorization denial, and conflict behavior before implementation."

## Clarifications

### Session 2026-07-01

- Q: What default scope should teacher workspace screens use when a teacher first opens them? → A: Default to current active academic period and teacher active rosters; store selected period and roster in route query.
- Q: What scope should admin-observed teacher workflow screens expose? → A: School administrators get read/detail observation plus admin-only imports and closed-period corrections; other management stays teacher-owned unless contract requires admin control.
- Q: What import input experience should grade and attendance imports use? → A: Admin import UI accepts structured JSON payload entry only, capped by approved row limit; no CSV, spreadsheet, or file-upload import UI.
- Q: Should this slice include teacher review or grading of student questionnaire responses? → A: Exclude questionnaire response review and grading; only questionnaire authoring and learning-set attachment are in scope.
- Q: How should the frontend behave when learning-set create and scoped learning-set, grade, or attendance list contracts are missing? → A: Block learning-set create and scoped learning-set, grade, and attendance lists until OpenAPI adds roster-aware create and period or roster filters.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Manage teacher materials (Priority: P1)

A teacher manages instructional content and questionnaires in the active school context, including upload status, clean-file download, detail review, allowed edits, lifecycle state, and safe empty or denied states. Questionnaire response review and grading are outside this slice; questionnaire scope is limited to authoring and learning-set attachment. A school administrator can observe same-school records through list and detail views, run admin-only imports, and perform closed-period corrections where approved; other management remains teacher-owned unless the contract requires administrator control.

**Why this priority**: Teacher content and questionnaires are prerequisites for learning sets, student work, downloads, and later correction or reporting views.

**Independent Test**: Can be fully tested by signing in as an authorized teacher with an active school context, opening the teacher workspace, listing content and questionnaires, uploading valid and invalid content, creating or viewing a questionnaire, downloading clean content, attempting unavailable downloads, changing approved lifecycle state, and confirming empty, validation, authorization, tenant-mismatch, scan-pending, conflict, and stale-response states render safely.

**Acceptance Scenarios**:

1. **Given** an authorized teacher has an active school context, **When** they open content or questionnaire lists, **Then** they see only same-school records visible to them with documented pagination, filters, status labels, loading state, and empty state.
2. **Given** a teacher uploads an approved instructional file, **When** the request succeeds, **Then** the UI shows the created item with scan status and prevents student-facing or download actions until the content is clean where required.
3. **Given** uploaded content is pending scan, failed scan, inactive, deleted, unauthorized, or cross-school, **When** a teacher attempts download or attachment to a workflow, **Then** the UI shows safe denial or unavailable feedback without exposing private file paths or cross-tenant details.
4. **Given** a teacher creates or updates an approved questionnaire, **When** validation fails, **Then** field-level and summary feedback identify the invalid question, option, answer, sequence, or metadata without submitting undocumented fields.
5. **Given** a school administrator opens an admin-observed teacher materials view, **When** same-school records are listed or opened, **Then** the administrator sees read/detail observation controls, approved admin-only actions, and clear teacher ownership without taking over teacher-owned management actions unless documented.

---

### User Story 2 - Publish roster-aware learning sets (Priority: P2)

A teacher publishes and maintains learning sets for active roster membership or legacy readable assignment contexts using approved content and questionnaires, with status, student-visible availability, lifecycle constraints, and conflict feedback. Teacher workspace screens default to the current active academic period and the teacher's active rosters, and preserve explicit period or roster selection in route query state. Learning-set create and scoped learning-set lists remain blocked until OpenAPI adds roster-aware create behavior and period or roster filters.

**Why this priority**: Learning sets connect teacher-authored materials to classroom work. They must honor roster eligibility, academic-period scope, clean content, and historical compatibility before students can rely on assigned work.

**Independent Test**: Can be fully tested by opening an assigned roster or teacher workspace, verifying create and scoped list controls are blocked while roster-aware create or scoped list contracts are missing, then enabling the controls only after approved OpenAPI support exists, creating a learning set with clean content and active questionnaires, selecting an approved roster-aware audience, submitting valid and invalid dependency combinations, opening detail, changing approved lifecycle state, and confirming legacy direct-assignment records remain readable without enabling new unsupported direct-assignment writes.

**Acceptance Scenarios**:

1. **Given** a teacher has an active teacher assignment to a roster in the selected academic period, **When** they create or maintain a learning set, **Then** the UI uses approved roster-aware assignment behavior and shows the resulting audience, status, entries, and academic-period context.
2. **Given** selected content, questionnaires, roster, memberships, teacher assignment, or academic period are inactive, deleted, unclean, cross-school, unauthorized, or incompatible, **When** the teacher submits the learning-set workflow, **Then** the UI shows documented validation or conflict feedback and does not show partial success.
3. **Given** a legacy learning set uses direct selected-student assignments from the earlier teacher workflow foundation, **When** the learning set is viewed, **Then** the UI may display the legacy audience as read-only history and must not offer new direct selected-student assignment writes.
4. **Given** a learning set is active, inactive, deleted, restored, student-visible, or dependency-blocked, **When** the actor opens list or detail views, **Then** status labels and available actions match documented lifecycle and visibility rules.
5. **Given** OpenAPI does not document roster-aware learning-set create behavior or period/roster list filters, **When** a teacher opens learning-set create or scoped list surfaces, **Then** the UI blocks those controls and does not send undocumented filters or legacy direct-student assignment writes.

---

### User Story 3 - Record and correct grades and attendance (Priority: P3)

A teacher records grades and attendance for students in their approved roster context, reviews current values and history allowed for their role, and submits approved corrections while preserving audit-safe correction history. Grade and attendance scoped lists remain blocked until OpenAPI adds period or roster filters. A school administrator can observe same-school records, run approved imports, and perform closed-period corrections where approved.

**Why this priority**: Grades and attendance are core academic records. Teachers and administrators need clear current values, correction rules, lifecycle state, and denial handling before student self-service and reporting depend on the data.

**Independent Test**: Can be fully tested by verifying grade and attendance scoped lists are blocked while period or roster filters are missing, opening grade and attendance lists for a selected roster or academic period only after approved filters exist, creating valid records, submitting invalid values, opening details, submitting corrections with valid and invalid reasons, reviewing correction history, attempting closed-period corrections as teacher and administrator, and verifying status, authorization, validation, conflict, and stale-response behavior.

**Acceptance Scenarios**:

1. **Given** a teacher has an active same-school roster assignment and selected academic period, **When** they record a valid grade or attendance entry for an eligible student, **Then** the UI updates the current list or detail view with approved current value, status, recorder context, and period context.
2. **Given** a grade value, attendance status, student, roster membership, teacher assignment, or academic period is invalid, closed, inactive, unauthorized, or cross-school, **When** the actor submits the form, **Then** the UI shows documented validation, denial, or conflict feedback without changing visible state.
3. **Given** a current grade or attendance record is eligible for correction, **When** the actor submits an approved correction reason and new value, **Then** the UI shows the updated current value and correction history summary allowed for that actor.
4. **Given** a teacher attempts a closed-period correction, **When** the correction is submitted, **Then** the UI shows the documented denial; an authorized school administrator may submit approved closed-period corrections where the contract permits.
5. **Given** correction history contains private notes, actor metadata, or tenant-sensitive details not approved for the current actor, **When** history is displayed, **Then** the UI exposes only approved tenant-safe fields.
6. **Given** OpenAPI does not document period or roster filters for grade or attendance lists, **When** the actor opens scoped grade or attendance list surfaces, **Then** the UI blocks scoped loading and does not load broad lists for client-side filtering.

---

### User Story 4 - Run approved imports and review operational feedback (Priority: P4)

A school administrator runs approved create-only grade or attendance imports through structured JSON payload entry and reviews all-or-nothing validation results. Teachers can see import-related outcomes only where product scope and permissions approve observation.

**Why this priority**: Imports improve operational efficiency but must not bypass validation, correction rules, roster eligibility, tenant isolation, or safe error handling.

**Independent Test**: Can be fully tested by opening approved grade or attendance import screens as a school administrator, submitting valid and invalid import payloads, verifying accepted row counts or all-or-nothing rejection details, attempting teacher or unauthorized import access, and confirming no partial success appears for rejected imports.

**Acceptance Scenarios**:

1. **Given** an authorized school administrator has an active school context, **When** they submit an approved grade or attendance structured JSON import within the documented row limit, **Then** the UI shows accepted summary, affected scope, and resulting records where approved.
2. **Given** an import contains one or more invalid, duplicate, unauthorized, cross-school, malformed, closed-period, or correction-attempt rows, **When** it is submitted, **Then** the UI shows tenant-safe row feedback and no partial success state.
3. **Given** a teacher, student, guardian, platform user without school authority, or unauthorized administrator opens import routes or controls, **When** access is evaluated, **Then** import controls remain hidden or disabled and denied responses show safe feedback.

### Edge Cases

- Active school context is missing, inactive, changed during a request, or no longer authorized while teacher workflow screens are open.
- Teacher workspace responses arrive after the actor changes route, filter, pagination, selected academic period, selected roster, active school, or authentication state.
- Content, questionnaire, learning-set, grade, attendance, correction, download, or import lists return no records because of true empty state, restrictive filters, missing roster assignment, missing academic period, scan state, lifecycle state, or lack of authorization.
- No current active academic period or no active teacher assignment exists for the teacher; workspace surfaces must block scoped actions and explain the unavailable state where approved.
- Uploads are unsupported, oversized, mismatch declared and detected type, fail sanitization, fail scan, remain pending scan, or are rejected by tenant or permission checks.
- A teacher attempts to manage another teacher's records without approved ownership, assignment, or school-administrator authority.
- Used content or questionnaires receive edit attempts that would change historical student-facing meaning.
- Learning-set entries reference inactive, deleted, unclean, cross-school, unauthorized, missing, duplicate, or dependency-conflicting content, questionnaires, rosters, memberships, or academic periods.
- Legacy direct selected-student assignments appear in existing records but new direct selected-student assignment writes are not approved.
- OpenAPI lacks roster-aware learning-set create behavior or scoped learning-set, grade, or attendance list filters; matching UI controls must remain blocked until contract support exists.
- Grade or attendance forms reference ineligible students, inactive roster memberships, closed periods, unsupported values, duplicate records, stale state, or unauthorized targets.
- Correction reasons are missing, blank, shorter than 10 characters, longer than 500 characters, or submitted by an actor without closed-period authority.
- Download requests target pending-scan, failed-scan, inactive, deleted, unauthorized, cross-school, or unavailable content.
- Import submissions exceed approved row limits, use unsupported format, include CSV, spreadsheet, or file-upload attempts, include correction attempts, duplicate existing records, or contain one invalid row in an all-or-nothing request.
- Contract responses return validation, unauthorized, forbidden, tenant-mismatch, inactive-school, inactive-record, not-found, conflict, unsupported filter, unsupported sort, unsupported page-size, too-large import, scan-unavailable, temporary-unavailable, or stale-record states.
- Student questionnaire response review, teacher grading of questionnaire responses, student file-response downloads, and response correction workflows remain outside this slice.
- Diagnostics, client-side errors, and automated test output must not persist or print student, teacher, guardian, token, permission, role, private file path, full upload metadata, full import payload, correction private note, or cross-tenant details.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No new backend behavior is included in this frontend feature. Backend teacher workflow foundation, lifecycle, correction, download, and import operations must already be approved, implemented, and verified before frontend runtime exposure.
- **Frontend repository impact**: Implement teacher-facing workspace surfaces and admin-observed same-school teacher workflow screens for teacher content, questionnaires, learning sets, grades, attendance, correction history, downloads, and imports where approved, reusing established protected shell, list, detail, form, status, confirmation, denial, and conflict behavior. Admin-observed screens provide read/detail observation plus admin-only imports and closed-period corrections unless a contract explicitly grants broader administrator management.
- **Specification or contract repository impact**: This specification defines the frontend consumption boundary. A frontend contract artifact must map every UI surface to approved teacher workflow operations, response states, filters, pagination, permissions, lifecycle behavior, and authorization outcomes. Aggregate OpenAPI changes are required before implementation for roster-aware learning-set create behavior and scoped learning-set, grade, and attendance list filters.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines this UI boundary first, `schoolmaster-frontend` implements after plan and tasks, and `schoolmaster-backend` changes are allowed only through separate backend or contract work if this specification identifies a missing approved operation.

### API Contract Impact

- **OpenAPI update required**: Yes before implementing learning-set create and scoped learning-set, grade, or attendance list controls: OpenAPI must document roster-aware learning-set create behavior and period or roster filters for learning-set, grade, and attendance lists. No UI may use undocumented filters, broad client-side filtering, or legacy direct-student learning-set writes as a workaround.
- **Versioned endpoints affected**: Frontend may consume only approved `/api/v1/teacher-content`, `/api/v1/questionnaires`, `/api/v1/learning-sets`, `/api/v1/grades`, `/api/v1/attendance`, and `/api/v1/teacher-assignments` operations needed for this slice. Student questionnaire-response review, grading, and student response file-download operations are excluded from this feature.
- **JSON response impact**: UI behavior depends on documented success, paginated, validation, unauthorized, forbidden, tenant-mismatch, inactive-school, inactive-record, not-found, conflict, correction-history, import-validation, download, unsupported filter, unsupported sort, unsupported page-size, scan-unavailable, and temporary-unavailable envelopes. No UI may depend on undocumented fields, status codes, inferred backend state, private file paths, or hidden actor metadata.
- **Authentication/authorization impact**: All protected screens require authenticated access and active permitted school context. Teacher-facing surfaces require approved teacher workflow authority and roster or ownership scope where applicable. Admin-observed surfaces require same-school school-administrator authority. Platform access does not grant implicit school-owned teacher workflow access.
- **Compatibility impact**: Frontend delivery is additive. It must not change existing authentication, administration, student enrollment, roster administration, student self-service, guardian self-service, reporting, or legacy direct-assignment read behavior unless a separate approved specification changes those areas.

### Data & Tenancy Impact

- **Tenant scoping impact**: Teacher content, questionnaires, learning sets, learning-set entries, assignments, grades, attendance, correction history, import summaries, download availability, teacher assignments, academic-period context, and related status display are school-owned and scoped to the active permitted school.
- **Cross-tenant or platform access impact**: Cross-school records, private files, private correction notes, import payloads, student identifiers, teacher identifiers, and assignment details remain isolated unless a separate approved support or reporting contract permits exposure.
- **Soft delete impact**: Active, inactive, deleted, restored, hidden, historical-only, and unavailable records may appear only where approved for teacher or administrator review. Permanent deletion, purge, anonymization, legal hold, and unsupported restore behavior are outside this UI slice.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The UI MUST expose only approved teacher content, questionnaire, learning-set, grade, attendance, correction, download, import, and teacher-assignment behavior documented before implementation begins.
- **FR-002**: The UI MUST require an authenticated user and active permitted school context before protected teacher workflow screens load data or enable actions.
- **FR-003**: The UI MUST distinguish teacher-facing workspace screens from admin-observed same-school teacher workflow screens and show controls according to approved actor permissions, ownership, roster assignment, and school-administrator authority.
- **FR-003a**: Admin-observed teacher workflow screens MUST provide same-school read/detail observation plus admin-only imports and closed-period corrections, and MUST NOT expose teacher-owned create, update, lifecycle, download, or learning-set management controls to school administrators unless the approved contract explicitly requires that administrator action.
- **FR-004**: The UI MUST list teacher content and questionnaires with documented pagination, supported filters, lifecycle status, scan status, ownership cues, loading state, empty state, validation feedback, stale-response handling, and safe denial states.
- **FR-005**: The UI MUST support approved teacher content upload behavior, including required metadata, allowed file feedback, upload progress or pending state where approved, scan status, scan-gated availability, validation errors, and safe rejection of unsupported or unavailable files.
- **FR-006**: The UI MUST support approved questionnaire create, detail, update, lifecycle, and restore behavior, including supported question types, sequence display, validation feedback, historical-meaning conflict feedback, and safe unsupported-action states.
- **FR-006a**: The UI MUST NOT expose student questionnaire response review, teacher grading of questionnaire responses, student response file downloads, or response correction workflows in this slice.
- **FR-007**: The UI MUST support clean authorized teacher content download behavior and show documented denial feedback for pending-scan, failed-scan, inactive, deleted, unauthorized, cross-school, or unavailable content without exposing private file paths.
- **FR-008**: The UI MUST list and maintain learning sets within approved school, academic-period, roster, teacher-assignment, ownership, and lifecycle scope using documented filters, pagination, detail, entries, audience, status display, empty states, and conflict feedback.
- **FR-008a**: Teacher workspace list and detail surfaces MUST default to the current active academic period and the teacher's active rosters, MUST preserve explicit selected academic period and roster in route query state, and MUST block scoped data loading when no approved period or roster context is available.
- **FR-008b**: Scoped learning-set lists MUST remain blocked until OpenAPI documents approved period or roster filters; the UI MUST NOT load broad learning-set lists for client-side filtering.
- **FR-009**: The UI MUST use approved roster-aware learning-set assignment behavior for new writes and MUST keep legacy direct selected-student assignments read-only when they appear in existing records.
- **FR-009a**: Learning-set create controls MUST remain blocked until OpenAPI documents roster-aware create behavior; the UI MUST NOT create new legacy direct selected-student assignment writes as a workaround.
- **FR-010**: The UI MUST display learning-set dependency and lifecycle constraints, including blocked actions caused by inactive, deleted, unclean, cross-school, unauthorized, missing, or incompatible content, questionnaires, rosters, memberships, teacher assignments, or academic periods.
- **FR-011**: The UI MUST support approved grade and attendance list, create, detail, lifecycle, and restore behavior with documented academic-period scope, roster or student eligibility, status display, current value, recorder context, loading, empty, validation, denial, and conflict states.
- **FR-011a**: Scoped grade and attendance lists MUST remain blocked until OpenAPI documents approved period or roster filters; the UI MUST NOT load broad grade or attendance lists for client-side filtering.
- **FR-012**: The UI MUST support approved grade and attendance correction workflows, including current value display, new value input, 10-500 character reason requirements, actor authority, closed-period denial or administrator authority, correction history summary, validation feedback, and private-note redaction.
- **FR-013**: The UI MUST support approved grade and attendance import workflows for authorized school administrators only, including documented format, row limit, all-or-nothing validation, accepted summary, rejected summary, tenant-safe row feedback, and no partial-success display for rejected imports.
- **FR-013a**: Grade and attendance import UI MUST accept structured JSON payload entry only, MUST stay within the approved row limit, and MUST NOT expose CSV, spreadsheet, archive, or file-upload import controls in this slice.
- **FR-014**: The UI MUST hide or disable import controls for teachers, students, guardians, platform users without school authority, and unauthorized administrators, and MUST render safe denial feedback for direct route or response denial.
- **FR-015**: The UI MUST show safe unauthorized, forbidden, tenant-mismatch, inactive-school, inactive-record, not-found, validation, conflict, unsupported filter, unsupported sort, unsupported page-size, scan-unavailable, import-validation, download-denied, stale-record, and temporary-unavailable states without exposing cross-tenant or protected details.
- **FR-016**: The UI MUST ignore or cancel stale teacher workflow results when the user changes route, filter, pagination, selected academic period, selected roster, active school, or authentication state before the response is applied.
- **FR-017**: The UI MUST keep existing protected shell, navigation, list, detail, form, confirmation, status, denial, pagination, filter, and conflict behavior consistent with completed frontend features 016 through 022.
- **FR-018**: The UI MUST NOT introduce unsupported student self-service authoring, guardian self-service, reporting workspace behavior, platform support access, billing, messaging, notification-center behavior, advanced assessment content types, permanent purge, legal hold, data anonymization, or undocumented API behavior in this slice.
- **FR-019**: The UI MUST include test coverage or equivalent verification for successful flows, validation failures, authorization denials, tenant isolation denials, upload and scan states, download denials, lifecycle conflicts, learning-set dependency conflicts, correction reason rules, closed-period correction authority, import all-or-nothing behavior, empty states, pagination, stale responses, safe error display, and no-sensitive-data diagnostics behavior.

### Key Entities *(include if feature involves data)*

- **TeacherContentItem**: School-owned instructional content metadata and private file availability state, including title, description, owner, lifecycle status, scan status, download eligibility, and dependency cues.
- **Questionnaire**: School-owned teacher-authored activity with supported question structure, owner, lifecycle status, sequence display, usage dependencies, and historical-meaning constraints.
- **LearningSet**: School-owned instructional sequence for an academic period with ordered entries, owner, lifecycle status, audience summary, student-visible availability, and dependency conflicts.
- **LearningSetAssignment**: Audience relationship for a learning set; new writes use approved roster-aware context while legacy direct selected-student assignments remain read-only where present.
- **GradeRecord**: School-owned academic record for one student and academic period, including current value, original recorder context, lifecycle status, correction eligibility, and correction history summary.
- **AttendanceRecord**: School-owned attendance record for one student, academic period, and date, including current status, original recorder context, lifecycle status, correction eligibility, and correction history summary.
- **CorrectionRecord**: Tenant-safe history entry for an accepted grade or attendance correction, including original value reference, new value, actor visibility allowed for the viewer, required reason summary, timestamp, and student visibility impact.
- **ImportRun**: Tenant-safe summary of a grade or attendance import, including target resource, actor, accepted count, rejected count, all-or-nothing outcome, and safe validation details.
- **TeacherAssignment**: Approved relationship that defines teacher access to roster-aware workflow surfaces for a class section or roster during an academic period.
- **ClassSection/Roster**: Existing school-owned teaching structure whose active memberships define the approved student audience for teacher workflow actions.
- **AcademicPeriod**: School-owned period that scopes teacher assignments, learning sets, grades, attendance, corrections, imports, filters, and closed-period authority.
- **SchoolContext**: Active permitted school boundary used to load, filter, authorize, and display all school-owned teacher workflow records.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of teacher content, questionnaire, learning-set, grade, attendance, correction, download, import, and teacher-assignment UI actions can be traced to approved contract behavior before frontend implementation begins, with missing learning-set create or scoped list contracts producing blocked UI states instead of undocumented requests.
- **SC-002**: In usability checks, an authorized teacher can find a learning set, identify its current status and audience, and open its detail view in under 2 minutes without assistance.
- **SC-003**: In usability checks, an authorized teacher can upload content, identify scan-gated availability, create or find a questionnaire, and add approved clean materials to a learning set in under 6 minutes without assistance.
- **SC-004**: In usability checks, an authorized teacher can record a grade or attendance entry and identify a validation or conflict failure in under 4 minutes without assistance.
- **SC-005**: 100% of tested unauthorized, forbidden, tenant-mismatch, inactive-school, inactive-record, not-found, validation, conflict, scan-unavailable, download-denied, import-validation, and temporary-unavailable responses show safe feedback without exposing cross-school protected details.
- **SC-006**: 100% of tested pagination and filter combinations for teacher content, questionnaires, learning sets, grades, attendance, correction history, and imports show stable loading, empty, populated, unsupported-state, selected-period, and selected-roster behavior where approved.
- **SC-007**: 100% of tested stale responses caused by route, filter, pagination, academic-period, roster, active-school, or authentication changes do not overwrite the current visible screen state.
- **SC-008**: Import verification confirms rejected grade and attendance imports show no partial success state, and accepted imports show approved summary feedback for 100% of tested valid submissions.
- **SC-009**: At least 90% of representative teachers and administrators can correctly distinguish active, inactive, deleted, restored, scan-pending, scan-failed, unavailable, read-only legacy, and conflict states after completing the main workflows.
- **SC-010**: Review confirms no UI surface exposes undocumented student self-service, guardian self-service, reporting workspace, platform support, billing, messaging, permanent purge, legal hold, data anonymization, or cross-tenant behavior.
- **SC-011**: Diagnostics verification confirms student, teacher, guardian, token, permission, role, private file path, full upload metadata, full import payload, correction private note, and cross-tenant details do not appear in client-side diagnostics, error logs, or automated test output for teacher workflow flows.

## Assumptions

- Backend teacher workflow foundation rules from `004-backend-teacher-workflows` and lifecycle, correction, download, and import rules from `010-teacher-workflow-lifecycle` are the source of approved domain behavior for this frontend slice.
- Classroom roster and teacher assignment behavior from `009-classroom-roster-foundation` and feature 022 student enrollment and roster UI constraints govern roster-aware teacher workflow access.
- Teacher workflow operations are school-owned and require active permitted school context.
- Teachers manage records they created or own and records available through approved active teacher assignments; school administrators may observe same-school records through read/detail views and perform admin-only imports or closed-period corrections where approved, but do not receive broader teacher-owned management controls unless required by contract.
- New learning-set assignment writes use roster-aware assignment rules; legacy direct selected-student assignments remain readable only where documented.
- Learning-set create and scoped learning-set, grade, and attendance list controls remain blocked until OpenAPI documents roster-aware learning-set create behavior and period or roster filters.
- Content download is available only through approved operations for clean and authorized content; private file paths never appear in UI state, diagnostics, or errors.
- Grade and attendance corrections require a 10-500 character free-text reason; closed-period corrections are school-administrator-only where approved.
- Grade and attendance imports are school-administrator-only, use approved create-only behavior, and are all-or-nothing.
- Grade and attendance import UI uses structured JSON payload entry only; CSV, spreadsheet, archive, and file-upload import flows remain outside this slice.
- Academic-period filters, roster filters, teacher-assignment eligibility, lifecycle states, correction history visibility, and import limits follow approved backend contract behavior.
- Teacher-facing workspace screens default to the current active academic period and active teacher rosters; explicit period and roster selection is preserved in route query state.
- Existing admin shell, authentication, administration foundation, administration lifecycle, account lifecycle, and student enrollment/roster UI behavior from completed frontend features 016 through 022 remains the baseline.
- Student self-service, guardian self-service, reporting workspace, platform support access, billing, messaging, notifications, permanent purge, legal hold, data anonymization, and unsupported advanced assessment behavior remain outside this slice.

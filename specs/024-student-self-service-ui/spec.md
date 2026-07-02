# Feature Specification: Student Self-Service UI

**Feature Branch**: `024-student-self-service-ui`  
**Created**: 2026-07-01  
**Status**: Ready for Planning  
**Input**: User description: "Specify Student Self-Service UI frontend feature. Define student-facing screens for assigned learning sets, authorized content access, own grades, own attendance, and approved academic or reporting views. Scope must reuse existing protected-shell, list, detail, status, and empty-state patterns, consume only approved student self-service API routes, and document authentication, authorization, unavailable-content, no-active-school, no-current-period, and not-found behavior before implementation."

## Clarifications

### Session 2026-07-01

- Q: How should questionnaire entries returned in assigned learning sets behave in this UI slice? → A: Show questionnaire entries read-only only; no response submit/review UI in this slice.
- Q: How should academic-period scope work in this UI slice? → A: Use current active academic period only; no manual period switch in this slice.
- Q: How should the UI behave when the authenticated account has no active linked student profile in the active school? → A: Show distinct no-student-profile state; no data requests.
- Q: Which screen should the student workspace open by default? → A: Default student workspace opens Assigned Learning Sets.
- Q: What may the academic summary calculate in this UI slice? → A: Summary uses counts/status only; no calculated GPA or attendance rate.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Review Assigned Learning Sets (Priority: P1)

An authenticated student opens the student workspace for the active school and current active academic period, lands on Assigned Learning Sets by default, then reviews learning sets assigned to their own student profile with ordered entries, status labels, safe loading, and empty-state behavior.

**Why this priority**: Assigned learning sets are the primary student-facing entry point for coursework and are the source for authorized content access.

**Independent Test**: Can be fully tested by signing in as a student with an active school and current active academic period, opening assigned learning sets, selecting a learning set from the returned timeline, and confirming only the student's own assigned learning sets and entries appear.

**Acceptance Scenarios**:

1. **Given** an authenticated student has an active permitted school context, an active linked student profile, and a current active academic period, **When** they open the student workspace without a deeper student route, **Then** the workspace defaults to Assigned Learning Sets and shows assigned learning sets from the approved student timeline contract with loading, pagination, status, and empty-state behavior consistent with existing protected list patterns.
2. **Given** a returned learning set contains multiple entries, **When** the student opens the learning set detail surface from the timeline, **Then** entries are shown in the documented sequence with safe status and availability indicators, and questionnaire entries remain read-only.
3. **Given** the student has no assigned learning sets for the current period, **When** the timeline loads successfully, **Then** the UI shows the approved empty-state pattern and does not show teacher, administrator, guardian, or other-student data.
4. **Given** no current academic period is available from approved session or tenant context, **When** the student opens assigned learning sets, **Then** the UI shows the no-current-period state and does not send a timeline request without the required period.
5. **Given** an authenticated account has no active linked student profile in the active school, **When** the user opens any student self-service screen, **Then** the UI shows the no-student-profile state and does not send student self-service data requests.

---

### User Story 2 - Access Authorized Content (Priority: P2)

A student downloads only content that the approved student contract marks as available through an assigned learning set, while pending, failed, inactive, missing, unassigned, or unauthorized content remains unavailable without exposing private file details.

**Why this priority**: Student material access must be useful while preserving assignment scope, malware-scan gating, private storage boundaries, and tenant isolation.

**Independent Test**: Can be fully tested by opening an assigned learning set with clean downloadable content, downloading it, then attempting unavailable, unassigned, cross-school, and missing content states and verifying safe feedback.

**Acceptance Scenarios**:

1. **Given** an assigned content entry is marked `download_available`, **When** the student chooses download, **Then** the UI uses only the approved student content download operation and provides the file response without exposing private storage paths.
2. **Given** an assigned content entry is pending scan, failed scan, unavailable, inactive, deleted, or not marked `download_available`, **When** the student views the entry, **Then** the download action remains hidden or disabled and the unavailable-content state explains that the material cannot be accessed yet.
3. **Given** a direct download attempt returns unauthorized, forbidden, tenant-mismatch, validation, unavailable, or not-found behavior, **When** the UI handles the response, **Then** it shows safe feedback and does not reveal whether another school, student, private path, or unassigned file exists.

---

### User Story 3 - Review Own Grades and Attendance (Priority: P3)

A student reviews their own current grade and attendance records for the active school, using existing list, detail, status, pagination, and empty-state patterns without correction, import, authoring, or other-student controls.

**Why this priority**: Grades and attendance are core academic self-service information and must be student-owned, readable, and safe before broader academic summaries depend on them.

**Independent Test**: Can be fully tested by signing in as a student with grade and attendance records, opening both lists, opening read-only details from loaded records, and verifying no records from another student or school appear.

**Acceptance Scenarios**:

1. **Given** an authenticated student has grade records in the active school, **When** they open the grades screen, **Then** they see only their own grade records using approved pagination, status labels, loading, empty, and read-only detail behavior.
2. **Given** an authenticated student has attendance records in the active school, **When** they open the attendance screen, **Then** they see only their own attendance records using approved pagination, status labels, loading, empty, and read-only detail behavior.
3. **Given** no grade or attendance records are returned for the selected scope, **When** the list loads successfully, **Then** the UI shows a clear empty state without treating the result as an authorization failure.
4. **Given** a grade or attendance record is no longer present in the approved list response, **When** the student attempts to open its read-only detail surface from a stale route, bookmark, or cached state, **Then** the UI shows the not-found state and does not request undocumented detail endpoints.

---

### User Story 4 - View Approved Academic Summaries (Priority: P4)

A student views a simple academic overview or reporting-style summary derived only from approved student self-service data counts and statuses, while report-run creation, report downloads, custom reporting, calculated GPA, calculated attendance rate, and cross-student views remain unavailable.

**Why this priority**: Students need orientation across assignments, grades, attendance, and content availability, but the UI must not imply access to administrative reporting or undocumented report behavior.

**Independent Test**: Can be fully tested by opening the student overview with approved learning-set, grade, and attendance responses, verifying the displayed summary matches only those records, then confirming report-run or download controls are absent.

**Acceptance Scenarios**:

1. **Given** approved student self-service data is available for the active school and current active academic period, **When** the student opens the overview, **Then** the UI summarizes assigned learning-set counts, content-availability counts, grade statuses, and attendance statuses using only the approved student route responses.
2. **Given** no student-facing report request, report-run list, report download, transcript, or custom-report operation is approved, **When** the student opens academic or reporting views, **Then** those controls remain hidden or blocked with a contract-unavailable state.
3. **Given** the active school or current period is missing, inactive, unauthorized, or changed during loading, **When** the overview would otherwise render tenant-owned academic data, **Then** the UI shows the no-active-school or no-current-period state and keeps protected content hidden.

### Edge Cases

- Active school context is missing, inactive, changed during a request, or no longer authorized while student screens are open.
- Authenticated account has no active linked student profile in the active school; student self-service data requests must remain blocked behind a distinct no-student-profile state.
- No current academic period is available, or the current period becomes inactive while assigned learning sets or summaries are open.
- Learning sets, grades, or attendance responses return no records because the student has no assignments, no academic records, restrictive period scope, lifecycle state, or authorization limits.
- A direct link targets a student workspace route, selected learning-set item, grade detail, attendance detail, or content download that is no longer available in the approved response scope.
- Content is pending scan, failed scan, inactive, deleted, missing, unassigned, cross-school, outside the active school, or otherwise unavailable.
- Student self-service responses arrive after the student changes route, pagination, current active academic period, active school, authentication state, or session state.
- The UI receives validation, unauthorized, forbidden, tenant-mismatch, inactive-school, unavailable-content, not-found, unsupported page-size, temporary-unavailable, or stale-response behavior.
- A student attempts to access teacher, administrator, guardian, platform, report-run, custom reporting, correction, import, authoring, or other-student surfaces by URL.
- Client-side diagnostics, automated tests, and visible errors must not expose student identifiers beyond the current student context, teacher identifiers not approved for student display, guardian data, private file paths, storage keys, token values, role internals, scan internals, or cross-tenant details.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No new backend behavior is included in this frontend feature. Student learning timeline, student content download, student grade self-view, and student attendance self-view behavior must already be approved and verified before frontend runtime exposure.
- **Frontend repository impact**: Adds student-facing protected workspace screens for assigned learning sets, selected learning-set detail from returned timeline data, authorized content download, own grades, own attendance, and approved academic summary surfaces. The implementation must reuse existing protected-shell, list, detail, status, pagination, loading, denial, not-found, unavailable, and empty-state patterns.
- **Specification or contract repository impact**: This specification defines the frontend consumption boundary for approved student self-service operations. OpenAPI changes are not required for the listed student routes, but any student-facing report-run, transcript, detailed record endpoint, period picker source, questionnaire response surface, or new summary field requires a separate specification and OpenAPI update before implementation.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines this UI boundary first, `schoolmaster-frontend` implements after plan and tasks, and `schoolmaster-backend` changes are allowed only through separate backend or contract work if a missing approved operation is identified.

### API Contract Impact

- **OpenAPI update required**: No for the currently approved student self-service routes in scope. Yes before exposing any student-facing report runs, report downloads, transcript view, broad academic report, standalone learning-set detail endpoint, standalone grade detail endpoint, standalone attendance detail endpoint, period-selection source, questionnaire response UI, or undocumented summary field.
- **Versioned endpoints affected**: Frontend may consume only `/api/v1/student/learning-sets`, `/api/v1/student/teacher-content/{contentItemId}/download`, `/api/v1/student/grades`, and `/api/v1/student/attendance` for this slice, plus already approved authentication, current-user, permission, and active-school/session context behavior from the authentication foundation.
- **JSON response impact**: UI behavior depends on documented success, paginated, binary download, validation, unauthorized, forbidden where approved, tenant-mismatch, inactive-school, unavailable-content, not-found, unsupported pagination, and temporary-unavailable behavior. No UI may depend on undocumented fields, status codes, inferred backend state, private file paths, scan internals, or hidden actor metadata.
- **Authentication/authorization impact**: All student self-service screens require authenticated access, active permitted school context, and an active same-school linked student profile. Student-facing navigation and controls are visibility aids only; backend authorization remains authoritative.
- **Compatibility impact**: Frontend delivery is additive. It must not change existing authentication, administration, teacher workflow, guardian self-service, reporting workspace, platform support, or backend student self-view behavior unless a separate approved specification changes those areas.

### Data & Tenancy Impact

- **Tenant scoping impact**: Learning sets, learning-set entries, teacher content metadata, teacher content downloads, grades, attendance, academic-period context, and student summary displays are school-owned and scoped to the active permitted school and the authenticated student's own profile.
- **Cross-tenant or platform access impact**: Cross-school records, other-student records, private files, private paths, teacher-only metadata, guardian data, administrator report data, and platform/support data remain inaccessible unless a separate approved contract permits exposure.
- **Soft delete impact**: Active, inactive, deleted, unavailable, and historical records may appear only where approved for student self-view. Permanent deletion, purge, restore controls, correction controls, import controls, legal hold, and anonymization are outside this UI slice.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The UI MUST expose only approved student self-service behavior documented before implementation begins.
- **FR-002**: The UI MUST require an authenticated user and active permitted school context before student self-service screens load data or enable protected actions.
- **FR-003**: The UI MUST require the authenticated user to have an active same-school linked student profile before showing assigned learning sets, content downloads, grades, attendance, or academic summaries; when absent, the UI MUST show a distinct no-student-profile state and MUST NOT send student self-service data requests.
- **FR-004**: If no active permitted school is selected or resolvable, the UI MUST show a no-active-school state, keep tenant-owned student content hidden, and avoid student self-service data requests.
- **FR-005**: If no current active academic period is available from approved context, the UI MUST show a no-current-period state for assigned learning sets, grades, attendance, and any academic summary, and MUST NOT send learning-set, grade, attendance, or summary data requests without an academic period.
- **FR-006**: The UI MUST list assigned learning sets using only the approved student learning-set route, documented required academic-period filter, pagination, returned sequence, and returned student-visible fields.
- **FR-006a**: The default student workspace route MUST open Assigned Learning Sets when the student has authenticated access, active permitted school context, active linked student profile, and current active academic period.
- **FR-007**: The UI MUST provide selected learning-set detail only from data returned by the approved student timeline response unless a standalone student learning-set detail contract is approved later.
- **FR-008**: The UI MUST show assigned learning-set entries with documented entry type, sequence, title, content availability, and safe status behavior, and MUST NOT expose teacher-only authoring controls or unapproved questionnaire response behavior.
- **FR-009**: The UI MUST offer content download only when the approved student metadata indicates the content is `download_available`.
- **FR-010**: The UI MUST use only the approved student content download operation for authorized downloads and MUST show safe unavailable-content or not-found feedback for pending-scan, failed-scan, inactive, deleted, missing, unassigned, cross-school, unauthorized, or otherwise unavailable content.
- **FR-011**: The UI MUST list grade records using only the approved student grades route, documented pagination, required current active academic-period filter, returned student-visible grade value, label, status, and recorded timestamp fields; manual period switching and unscoped grade lists are outside this slice.
- **FR-012**: The UI MUST list attendance records using only the approved student attendance route, documented pagination, required current active academic-period filter, returned student-visible attendance date, current attendance status, and lifecycle status fields; manual period switching and unscoped attendance lists are outside this slice.
- **FR-013**: The UI MUST provide read-only grade and attendance detail surfaces only from records returned by the approved list responses unless standalone student detail contracts are approved later.
- **FR-014**: The UI MUST render true empty states separately from denied, unavailable, no-active-school, no-student-profile, no-current-period, validation, tenant-mismatch, inactive-school, temporary-unavailable, and not-found states.
- **FR-015**: The UI MUST show safe unauthorized, forbidden, tenant-mismatch, inactive-school, no-active-school, no-student-profile, no-current-period, unavailable-content, validation, not-found, unsupported page-size, stale-response, and temporary-unavailable states without exposing cross-tenant or protected details.
- **FR-016**: The UI MUST ignore or cancel stale student self-service results when the student changes route, pagination, current active academic period, active school, authentication state, or session state before the response is applied.
- **FR-017**: The UI MUST reuse existing protected shell, navigation, list, detail, status, pagination, loading, empty-state, denial, unavailable, and not-found behavior from completed frontend features.
- **FR-018**: The UI MUST keep student-facing academic or reporting-style summaries limited to counts and statuses from approved student learning-set, content availability, grade, and attendance responses; calculated GPA, calculated attendance rate, rankings, and trend analytics are outside this slice.
- **FR-019**: The UI MUST NOT expose report-run creation, report-run listing, report downloads, custom report definitions, transcripts, broad academic reports, or report filters to students unless a future specification and OpenAPI contract explicitly approve student-facing access.
- **FR-020**: The UI MUST NOT expose teacher authoring, administrator management, guardian self-service, platform support, grade or attendance correction, import, restore, purge, legal hold, anonymization, messaging, notification-center, billing, or undocumented API behavior in this slice.
- **FR-021**: The UI MUST include test coverage or equivalent verification for successful flows, authentication failures, authorization denials, no active school, no student profile, no current period, own-profile boundaries, tenant isolation denials, unavailable content, not-found states, empty states, pagination, stale responses, safe error display, and no-sensitive-data diagnostics behavior.

### Key Entities *(include if feature involves data)*

- **StudentWorkspace**: Protected student-facing surface for own learning, academic records, content access, and summary orientation within the active school.
- **AssignedLearningSetsLanding**: Default student workspace landing surface for the current active academic period.
- **StudentProfileContext**: The authenticated student's active same-school profile relationship required before self-service data can be shown.
- **SchoolContext**: Active permitted school boundary used to load, authorize, and display all school-owned student self-service records.
- **AcademicPeriodContext**: Current active academic period used to scope assigned learning sets, grades, attendance, and academic summaries; manual period switching is outside this slice.
- **StudentLearningSet**: Student-visible assigned instructional sequence returned by the approved timeline, including title, status, published timestamp, academic period, and ordered entries.
- **StudentLearningSetEntry**: Ordered content or questionnaire entry displayed inside a returned learning set using only student-visible entry fields.
- **StudentContentMetadata**: Student-visible content metadata including title, content type, file size, scan status, and download availability.
- **StudentContentDownload**: Authorized binary file response for content that belongs to the active school, is assigned to the student, and is available through the approved contract.
- **StudentGradeRecord**: Student-visible grade record for the authenticated student's own profile and active school, including current visible value, optional label, lifecycle status, academic period, and recorded timestamp.
- **StudentAttendanceRecord**: Student-visible attendance record for the authenticated student's own profile and active school, including date, current visible status, lifecycle status, and academic period.
- **AcademicSummaryView**: Read-only student summary assembled only from counts and statuses in approved student learning-set, grade, attendance, and content-availability responses; calculated GPA and attendance-rate metrics are outside this slice.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of student learning-set, content-download, grade, attendance, and summary UI actions can be traced to approved student self-service contract behavior before frontend implementation begins.
- **SC-002**: In usability checks, an authenticated student with an active school and current active academic period can find an assigned learning set, identify its status, and open its entry detail in under 2 minutes without assistance.
- **SC-003**: In usability checks, an authenticated student can identify whether assigned content is downloadable or unavailable and complete one authorized download in under 2 minutes without assistance.
- **SC-004**: In usability checks, an authenticated student can find their own grades and attendance and distinguish populated, empty, unavailable, and denied states in under 3 minutes without assistance.
- **SC-005**: 100% of tested unauthorized, forbidden, tenant-mismatch, inactive-school, no-active-school, no-student-profile, no-current-period, unavailable-content, validation, not-found, unsupported page-size, stale-response, and temporary-unavailable responses show safe feedback without exposing cross-school, other-student, private file, or protected actor details.
- **SC-006**: 100% of tested stale responses caused by route, pagination, current active academic period, active school, authentication, or session changes do not overwrite the current visible screen state.
- **SC-007**: Review confirms no UI surface exposes undocumented student reports, report downloads, custom reporting, teacher authoring, administrator management, guardian self-service, platform support, correction, import, restore, purge, messaging, billing, or cross-tenant behavior.
- **SC-008**: At least 90% of representative students can correctly distinguish active assignment, empty assignment, downloadable content, unavailable content, no-current-period, and not-found states after completing the main workflows.
- **SC-009**: Diagnostics verification confirms private file paths, storage keys, token values, role internals, scan internals, guardian data, other-student data, and cross-tenant details do not appear in client-side diagnostics, visible errors, or automated test output for student self-service flows.

## Assumptions

- Backend student and reporting foundation rules from `005-backend-student-reporting` are the source of approved domain behavior for student learning timelines, student content download, student grades, and student attendance.
- Existing authentication and session UI behavior from `017-auth-session-ui` provides protected-route handling, current-user hydration, active school context, no-active-school handling, session-expired handling, unauthorized handling, forbidden handling, inactive-user handling, inactive-school handling, and tenant-mismatch handling.
- Existing frontend features provide reusable protected-shell, list, detail, status, pagination, loading, empty-state, denial, unavailable, and not-found patterns that this feature must reuse rather than redefine.
- Student learning-set, grade, attendance, and summary views require an approved academic-period context; if no current period exists, those data requests remain blocked until an approved context exists.
- Grade and attendance views use the current active academic period and do not expose manual period switching or unscoped list fallback in this slice.
- Student-facing academic or reporting-style summaries are limited to counts and statuses available from approved student self-service responses; administrative report runs, report downloads, custom reports, transcripts, calculated GPA, calculated attendance rate, rankings, and trend analytics are outside this slice.
- Standalone detail endpoints for student learning sets, grades, and attendance are not approved in this slice; direct detail views must use loaded list data or show a safe not-found or contract-unavailable state.
- Student questionnaire response submission, response review, advanced assessment file-response downloads, teacher grading, guardian visibility, and reporting expansion remain outside this slice unless a future approved specification and OpenAPI contract adds them; assigned questionnaire entries returned in learning sets are displayed as read-only navigation context only.

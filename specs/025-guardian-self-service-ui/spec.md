# Feature Specification: Guardian Self-Service UI

**Feature Branch**: `025-guardian-self-service-ui`  
**Created**: 2026-07-02  
**Status**: Ready for Planning  
**Input**: User description: "Specify the Guardian Self-Service UI for authenticated guardians. Define guardian-facing list and detail views for linked students, academic summary views, and contact/profile views using only approved guardian self-service contracts. Include authorization and tenant-safe behavior for missing guardian-student associations, inactive links, cross-tenant access attempts, not-found records, unavailable data, empty states, and student visibility limits. This feature follows the completed Student Self-Service UI and must establish the separate guardian UX, routing, permissions, and contract expectations before frontend implementation."

## Clarifications

### Session 2026-07-02

- Q: How should guardian academic summary choose the required academic period? → A: Use current active academic period only; if absent, show no-academic-period.
- Q: When should the UI show no-guardian-link instead of generic denial? → A: Only when approved session or access behavior safely identifies a missing guardian-user link.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Review Linked Students (Priority: P1)

An authenticated guardian opens the guardian workspace for the active school and sees only students that the school has approved for that guardian through active guardian-user link and active guardian-student association proof.

**Why this priority**: Guardian self-service is only safe and useful when the first screen proves association scope, separates guardian access from student self-view, and avoids exposing other students or schools.

**Independent Test**: Can be fully tested by signing in as a guardian with an active same-school guardian-user link and at least one active student association, opening the guardian workspace, and confirming only approved same-school linked students appear with limited summary fields.

**Acceptance Scenarios**:

1. **Given** an authenticated guardian has an active permitted school context, active guardian-user link, and active same-school association to one or more students, **When** they open the guardian workspace, **Then** the UI lists only guardian-visible students from the approved guardian student list contract with loading, pagination, status, and empty-state behavior.
2. **Given** a guardian has active associations in more than one school, **When** they use one active school context, **Then** the UI shows only students associated with the guardian in that active school.
3. **Given** a guardian has no active approved student association in the active school, **When** the guardian student list loads successfully, **Then** the UI shows a no-linked-students empty state and does not imply whether student records exist outside the guardian's permitted view.
4. **Given** the active school is missing, inactive, or no longer authorized while guardian screens are open, **When** the guardian workspace would otherwise load student data, **Then** the UI shows the appropriate no-active-school, inactive-school, unauthorized, forbidden, or tenant-mismatch state and does not send guardian self-service data requests without a valid school context.

---

### User Story 2 - View Linked Student Detail (Priority: P2)

A guardian opens a linked student's detail view and sees only the limited profile and enrollment summary fields approved for guardian self-service, without school-admin, teacher, student self-view, or other-guardian information.

**Why this priority**: Student detail is the natural follow-up from the guardian list and must preserve the same non-enumerating, summary-only visibility rules before academic and contact views rely on a selected student.

**Independent Test**: Can be fully tested by selecting a linked student from the guardian list, opening the detail view, and verifying the visible profile and enrollment summary match the approved guardian contract while restricted details remain absent.

**Acceptance Scenarios**:

1. **Given** a guardian selects a student returned in their approved list, **When** the student detail loads, **Then** the UI shows only the guardian-visible profile and enrollment summary fields documented for that student.
2. **Given** a direct route, stale selection, bookmarked URL, or typed identifier targets a missing, unassociated, inactive, transferred, deleted, or cross-tenant student, **When** the detail request resolves, **Then** the UI shows the same safe not-found state and does not reveal which condition occurred.
3. **Given** the selected student is removed from the guardian's active association scope while the detail is open, **When** the UI refreshes or receives an access failure, **Then** it clears the protected detail and returns the guardian to a safe not-found or no-linked-students state.

---

### User Story 3 - Review Academic Summaries (Priority: P3)

A guardian reviews summary-only academic information for a linked student using the current active same-school academic period only, including guardian-visible grade summary, attendance summary, and learning-set progress or status where approved.

**Why this priority**: Guardians need academic context to support students, but this UI must remain narrower than student self-service and must not expose detailed grade rows, attendance rows, correction history, teacher content, questionnaires, or report output.

**Independent Test**: Can be fully tested by opening a linked student's academic summary with current active same-school academic period context and verifying only summary values appear, then attempting no-current-period, unassociated-student, and cross-tenant cases.

**Acceptance Scenarios**:

1. **Given** a guardian has a selected linked student and current active same-school academic period context, **When** they open academic summary, **Then** the UI shows guardian-visible grade summary, attendance summary, and learning-set summary values from the approved academic summary contract.
2. **Given** no current active same-school academic period is available from approved context, **When** the guardian opens academic summary, **Then** the UI shows a no-academic-period state and does not request guardian academic data without the required period.
3. **Given** academic summary values are unavailable, empty, or not yet recorded for the current active period, **When** the academic summary loads successfully, **Then** the UI shows a true empty or unavailable-summary state distinct from denied, validation, tenant-mismatch, and not-found states.
4. **Given** a guardian attempts to access detailed grades, detailed attendance, correction history, teacher content downloads, questionnaire answers, student activity submission, reports, rankings, or custom reporting through the guardian workspace, **When** the route or control is evaluated, **Then** those surfaces remain hidden or blocked with a contract-unavailable state.

---

### User Story 4 - Review Contact and Relationship Information (Priority: P4)

A guardian views their own school-maintained guardian contact fields, school-approved relationship label, and the selected student's primary school-approved contact details for a permitted student.

**Why this priority**: Contact visibility is useful for guardians and school offices, but it is privacy-sensitive and must be limited to the approved guardian contact contract.

**Independent Test**: Can be fully tested by opening a linked student's contact view and verifying the authenticated guardian's own contact fields, relationship label, and student primary contact appear while other guardian records, non-primary contacts, emergency handling details, and school-only notes remain hidden.

**Acceptance Scenarios**:

1. **Given** a guardian has a selected linked student, **When** they open the contact view, **Then** the UI shows only the authenticated guardian's own contact fields, the school-approved relationship label, and the student's primary school-approved contact details.
2. **Given** a contact field is not present or not approved for guardian visibility, **When** the contact view renders, **Then** the UI shows a safe missing-value state without inferring private data.
3. **Given** a student has other guardians, non-primary contacts, emergency handling details, or school-only contact notes, **When** one guardian opens the contact view, **Then** those restricted records and fields are absent from the UI.

### Edge Cases

- Active school context is missing, inactive, changed during a request, or no longer authorized while guardian screens are open.
- Authenticated account has no active same-school guardian-user link; guardian self-service data requests must remain blocked behind a distinct no-guardian-link state only when approved session or access behavior safely identifies the missing link, otherwise the UI follows the documented response envelope.
- Guardian-user link, guardian record, guardian-student association, school, student profile, academic period, academic summary, or contact record becomes inactive or deleted after access was previously available.
- Guardian student list returns no records because no active approved associations exist in the active school.
- Direct routes target missing, unassociated, inactive, transferred, deleted, or cross-tenant students, academic periods, academic summaries, or contact views.
- Academic summary requires the current active same-school academic period, but no approved current active period is available.
- Guardian responses arrive after the guardian changes route, selected student, academic period, active school, authentication state, or session state.
- UI receives validation, unauthorized, forbidden, tenant-mismatch, inactive-school, no-active-school, no-guardian-link, no-linked-students, no-academic-period, unavailable-summary, not-found, unsupported page-size, temporary-unavailable, or stale-response behavior.
- A guardian attempts to access student self-service downloads, teacher content, questionnaire activity, detailed academic rows, correction history, report output, school-admin screens, teacher screens, platform support, or another guardian's contact data by URL.
- Visible errors, diagnostics, and automated test output must not expose unassociated student identifiers, other guardian data, non-primary contacts, school-only notes, correction details, teacher-private data, report data, token values, role internals, or cross-tenant details.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No new backend behavior is included in this frontend feature. Guardian student list, guardian student detail, guardian academic summary, and guardian contact view behavior must already be approved, implemented, and contract-compliant before frontend runtime exposure.
- **Frontend repository impact**: Adds guardian-facing protected workspace screens for linked student listing, linked student detail, summary-only academic views, and approved contact/profile views. The implementation must reuse existing protected-shell, list, detail, status, pagination, loading, denial, not-found, unavailable, and empty-state patterns while keeping guardian navigation separate from student self-service.
- **Specification or contract repository impact**: This specification defines the frontend consumption boundary for approved guardian self-service operations. OpenAPI changes are not required for the listed guardian routes, but any guardian-facing write, profile update, association request, detailed academic record, teacher content, report, notification, messaging, or new summary field requires a separate specification and OpenAPI update before implementation.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines this UI boundary first, `schoolmaster-frontend` implements after plan and tasks, and `schoolmaster-backend` changes are allowed only through separate backend or contract work if a missing approved operation is identified.

### API Contract Impact

- **OpenAPI update required**: No for the currently approved guardian self-service routes in scope. Yes before exposing any guardian-facing write, guardian profile update, association request, academic detail row, teacher content download, questionnaire action, report run, report download, notification, messaging, manual academic-period picker or alternate period source, or undocumented summary field.
- **Versioned endpoints affected**: Frontend may consume only `/api/v1/guardian/students`, `/api/v1/guardian/students/{studentProfileId}`, `/api/v1/guardian/students/{studentProfileId}/academics`, and `/api/v1/guardian/students/{studentProfileId}/contacts` for this slice, plus already approved authentication, current-user, permission, active-school, and session context behavior from the authentication foundation.
- **JSON response impact**: UI behavior depends on documented success, paginated, validation, unauthorized, tenant-mismatch, not-found, unsupported pagination, and temporary-unavailable behavior. Target-specific missing, unassociated, inactive, and cross-tenant student cases must be treated as the same safe not-found state. No UI may depend on undocumented fields, status codes, denial reasons, backend internals, or hidden actor metadata.
- **Authentication/authorization impact**: All guardian self-service screens require authenticated access, active permitted school context, an active same-school guardian-user link, and active school-approved association to each visible student. Guardian-facing navigation and controls are visibility aids only; backend authorization remains authoritative.
- **Compatibility impact**: Frontend delivery is additive. It must not change existing authentication, administration, teacher workflow, student self-service, reporting workspace, platform support, or backend guardian self-service behavior unless a separate approved specification changes those areas.

### Data & Tenancy Impact

- **Tenant scoping impact**: Guardian records, guardian-user links, guardian-student associations, student profiles, academic summaries, contact views, academic-period context, and visible relationship labels are school-owned and scoped to the active permitted school.
- **Cross-tenant or platform access impact**: Cross-school students, unassociated students, inactive associations, other guardians, non-primary contacts, teacher-only data, student self-view data, report data, school-admin data, and platform/support data remain inaccessible unless a separate approved contract permits exposure.
- **Soft delete impact**: Active, inactive, deleted, transferred, and unavailable records may appear only where approved for guardian self-service. Permanent deletion, purge, restore controls, correction controls, transfer controls, legal hold, anonymization, custody workflows, and guardian-facing lifecycle actions are outside this UI slice.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The UI MUST expose only approved guardian self-service behavior documented before implementation begins.
- **FR-002**: The UI MUST require an authenticated user and active permitted school context before guardian self-service screens load data or enable protected actions.
- **FR-003**: The UI MUST list guardian-visible students using only the approved guardian student list operation, documented tenant context, pagination, and returned guardian-visible fields.
- **FR-004**: The UI MUST show a true no-linked-students empty state when the guardian student list succeeds with no records, and MUST keep that state distinct from missing school, missing guardian link, denied access, validation, tenant-mismatch, and not-found behavior.
- **FR-005**: The UI MUST provide linked student detail using only the approved guardian student detail operation and MUST show only guardian-visible limited profile and enrollment summary fields.
- **FR-006**: For direct, stale, missing, unassociated, inactive, transferred, deleted, or cross-tenant target student routes, the UI MUST show the same safe not-found state and MUST NOT reveal which protected condition occurred.
- **FR-007**: The UI MUST provide academic summary views using only the approved guardian academic summary operation for a selected linked student and the current active same-school academic period.
- **FR-008**: If no approved current active academic period is available for guardian academic summary, the UI MUST show a no-academic-period state and MUST NOT request guardian academic data without the required period.
- **FR-009**: The UI MUST display only guardian-visible grade summary, attendance summary, learning-set status, and progress fields documented by the approved academic summary response.
- **FR-010**: The UI MUST NOT expose detailed grade rows, detailed attendance rows, private correction reasons, full correction history, internal actor metadata, private teacher content, questionnaire answers, student activity submission, report runs, report downloads, transcripts, rankings, or custom reporting in this slice.
- **FR-011**: The UI MUST provide contact views using only the approved guardian contact operation and MUST show only the authenticated guardian's own contact fields, school-approved relationship label, and the student's primary school-approved contact details.
- **FR-012**: The UI MUST NOT show other guardian records, non-primary student contact details, restricted emergency handling details, school-only notes, custody notes, legal-document details, or unapproved contact fields.
- **FR-013**: The UI MUST render true empty states separately from unauthorized, forbidden, tenant-mismatch, inactive-school, no-active-school, no-guardian-link, no-linked-students, no-academic-period, unavailable-summary, validation, not-found, unsupported page-size, stale-response, and temporary-unavailable states.
- **FR-014**: The UI MUST show no-guardian-link only when approved session or access behavior safely identifies a missing guardian-user link; other guardian access denials MUST follow the documented response envelope without client-side inference.
- **FR-015**: The UI MUST show safe unauthorized, forbidden, tenant-mismatch, inactive-school, no-active-school, no-guardian-link, no-linked-students, no-academic-period, unavailable-summary, validation, not-found, unsupported page-size, stale-response, and temporary-unavailable states without exposing protected student, guardian, contact, academic, report, or cross-tenant details.
- **FR-016**: The UI MUST ignore or cancel stale guardian self-service results when the guardian changes route, selected student, academic period, active school, authentication state, or session state before the response is applied.
- **FR-017**: The UI MUST keep guardian navigation, route names, titles, empty states, and denied states distinct from student self-service so guardians do not appear to have student-only permissions.
- **FR-018**: The UI MUST reuse existing protected shell, navigation, list, detail, status, pagination, loading, empty-state, denial, unavailable, and not-found behavior from completed frontend features.
- **FR-019**: The UI MUST NOT expose school administration, teacher workflow, student self-service, platform support, account lifecycle administration, guardian management, guardian-user-link provisioning, guardian profile maintenance, association requests, correction, import, restore, purge, legal hold, anonymization, messaging, notification-center, billing, or undocumented API behavior in this slice.
- **FR-020**: The UI MUST include test coverage or equivalent verification for successful guardian flows, authentication failures, authorization denials, no active school, safely identified missing guardian link behavior, no linked students, explicit academic period gating, tenant isolation denials, target-specific not-found non-enumeration, unavailable summary, empty states, pagination, stale responses, safe error display, and no-sensitive-data diagnostics behavior.

### Key Entities *(include if feature involves data)*

- **GuardianWorkspace**: Protected guardian-facing surface for linked student summaries, academic summaries, and contact views within the active school.
- **GuardianAccessContext**: The authenticated guardian's same-school access proof, including active account, active guardian-user link, and active guardian record where documented by approved behavior; missing link may become visible to the UI only through approved safe session or access behavior.
- **SchoolContext**: Active permitted school boundary used to load, authorize, and display all school-owned guardian self-service records.
- **GuardianStudentAssociation**: School-approved same-school relationship that allows a guardian to view one linked student in current self-service.
- **GuardianStudentList**: Paginated guardian-visible collection of active same-school student summaries associated with the authenticated guardian.
- **GuardianStudentDetail**: Limited profile and enrollment summary for a permitted active student.
- **AcademicPeriodContext**: Current active same-school academic period required before guardian academic summary data can be requested; manual period switching is outside this slice.
- **GuardianAcademicSummaryView**: Summary-only guardian view containing approved grade, attendance, and learning-set status or progress values for one linked student and academic period.
- **GuardianContactView**: Contact view limited to the authenticated guardian's own contact fields, relationship label, and the student's primary school-approved contact details.
- **GuardianSafeState**: UI state category for denied, unavailable, not-found, empty, stale, or missing-context behavior that avoids protected-record enumeration.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of guardian student list, student detail, academic summary, contact, and safe-state UI actions can be traced to approved guardian self-service contract behavior before frontend implementation begins.
- **SC-002**: In usability checks, an authenticated guardian with an active school and at least one active linked student can open the guardian workspace, find a linked student, and open the student detail in under 2 minutes without assistance.
- **SC-003**: In usability checks, an authenticated guardian can find a linked student's academic summary for the current active academic period and correctly identify grade, attendance, and learning-set summary status in under 3 minutes without assistance.
- **SC-004**: In usability checks, an authenticated guardian can find their own contact view for a linked student and distinguish available, missing, empty, unavailable, and denied contact states in under 2 minutes without assistance.
- **SC-005**: 100% of tested unauthorized, forbidden, tenant-mismatch, inactive-school, no-active-school, no-guardian-link, no-linked-students, no-academic-period, unavailable-summary, validation, not-found, unsupported page-size, stale-response, and temporary-unavailable responses show safe feedback without exposing protected student, guardian, contact, academic, report, or cross-school details.
- **SC-006**: 100% of target-specific missing, unassociated, inactive, transferred, deleted, and cross-tenant student attempts render the same not-found experience without protected-record enumeration.
- **SC-007**: 100% of tested stale responses caused by route, selected student, academic period, active school, authentication, or session changes do not overwrite the current visible screen state.
- **SC-008**: Review confirms no UI surface exposes undocumented guardian writes, guardian profile updates, association requests, detailed academic records, teacher content downloads, questionnaire actions, student self-service downloads, school administration, teacher workflows, platform support, reports, messaging, billing, or cross-tenant behavior.
- **SC-009**: At least 90% of representative guardians can correctly distinguish linked-student list, no-linked-students, student detail, academic summary, contact view, no-academic-period, unavailable-summary, and not-found states after completing the main workflows.
- **SC-010**: Diagnostics verification confirms unassociated student identifiers, other guardian records, non-primary contacts, emergency handling details, school-only notes, correction details, teacher-private data, report data, token values, role internals, and cross-tenant details do not appear in visible errors, client-side diagnostics, or automated test output for guardian self-service flows.

## Assumptions

- Backend guardian self-service rules from `011-guardian-self-service` are the source of approved domain behavior for guardian student listing, guardian student detail, guardian academic summaries, guardian contact views, guardian-user link proof, association proof, and non-enumerating not-found behavior.
- Existing authentication and session UI behavior from `017-auth-session-ui` provides protected-route handling, current-user hydration, active school context, no-active-school handling, session-expired handling, unauthorized handling, forbidden handling, inactive-user handling, inactive-school handling, and tenant-mismatch handling.
- Existing frontend features provide reusable protected-shell, list, detail, status, pagination, loading, empty-state, denial, unavailable, and not-found patterns that this feature must reuse rather than redefine.
- Guardian academic summaries use the current active same-school academic period only. If the frontend has no approved current active period source for guardians, academic summary requests remain blocked behind a no-academic-period or contract-unavailable state until such context exists.
- Guardian self-service is read-only in this slice. Guardian profile maintenance, association requests, consent-signature capture, custody dispute workflows, legal-document handling, messaging, notification-center behavior, report lifecycle expansion, teacher content downloads, billing, and undocumented APIs remain outside scope.
- Guardian visibility is narrower than student self-service. Student-only assigned learning-set details, content downloads, questionnaire activity, detailed grade rows, detailed attendance rows, and student report surfaces are not exposed to guardians unless a future approved specification and OpenAPI contract adds them.

# Feature Specification: Student Guardian Tabs

**Feature Branch**: `040-student-guardian-tabs`
**Created**: 2026-09-01
**Status**: Ready for Planning
**Input**: User description: "Remove the item Guardians from the sidebarmenu. A guardian will be created in the same form of Create student form. Separate them into tabs. In tha Guardians tab will be possible to add more than one guardian, e.g. one father and one mother. Max guardians will be 2 per student."

## Clarifications

### Session 2026-09-01

- Q: Should the Guardians tab create only new guardian records or also link existing same-school guardians? → A: Guardians tab can create new or link existing same-school guardians.
- Q: Which permissions are required to use guardian capture inside Create Student? → A: Require both student create and guardian manage permissions.
- Q: What should direct standalone guardian administration routes do after the sidebar item is removed? → A: Redirect standalone guardian routes to Create Student.
- Q: Are guardians required when creating a student? → A: Guardians optional: allow zero, one, or two.
- Q: Must guardian relationship labels be unique per student? → A: Allow duplicate relationship labels.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Remove Standalone Guardian Navigation (Priority: P1)

A school administrator using the administration sidebar no longer sees Guardians as a standalone navigation item, because guardian capture is now part of student creation.

**Why this priority**: The navigation change is the visible product decision that prevents administrators from starting guardian records outside the student enrollment workflow.

**Independent Test**: Can be fully tested by signing in as an authorized school administrator, opening the administration shell, and confirming the sidebar has no standalone Guardians item while student administration remains reachable.

**Acceptance Scenarios**:

1. **Given** an authorized school administrator has an active permitted school context, **When** the administration sidebar renders, **Then** no standalone Guardians item is shown.
2. **Given** a user has guardian administration permission, **When** the administration sidebar renders, **Then** that permission does not cause a standalone Guardians item to appear.
3. **Given** a user opens a bookmarked or direct standalone guardian administration route, **When** the route is evaluated, **Then** the user is redirected to the approved Create Student workflow without loading guardian list data.

---

### User Story 2 - Create Student With Guardian Tabs (Priority: P2)

A school administrator creates a student from a tabbed create-student form that separates student details from guardian details, so the administrator can complete the student and guardian enrollment context in one workflow by creating new guardians or linking existing same-school guardians.

**Why this priority**: Student creation is the new entry point for guardian capture and must remain clear before multiple guardian support is added.

**Independent Test**: Can be fully tested by opening Create Student as an administrator with both student create and guardian manage permissions, confirming separate Student and Guardians tabs, completing valid student details, creating a new guardian or linking an existing same-school guardian, and submitting the combined workflow.

**Acceptance Scenarios**:

1. **Given** an authorized school administrator with student create and guardian manage permissions starts Create Student, **When** the form opens, **Then** it shows separate Student and Guardians tabs within the same create workflow.
2. **Given** required student fields are incomplete, **When** the administrator moves between tabs or submits the workflow, **Then** validation feedback identifies the affected tab and field without losing entered guardian data.
3. **Given** valid student details and zero, one, or two valid new or existing same-school guardian entries are provided, **When** the administrator submits the workflow, **Then** the student is created and any submitted guardian relationship records are associated to that student.
4. **Given** the administrator leaves the create workflow with unsaved student or guardian changes, **When** navigation is attempted, **Then** existing unsaved-change confirmation behavior applies to the whole tabbed workflow.

---

### User Story 3 - Add Up To Two Guardians For One Student (Priority: P3)

A school administrator can add more than one guardian in the Guardians tab, such as father and mother, while the workflow prevents more than two guardians for one student.

**Why this priority**: Many students have more than one guardian, but the product constraint caps guardian records at two per student to keep student creation simple and predictable.

**Independent Test**: Can be fully tested by adding one guardian, adding a second guardian with a different relationship label, confirming both remain editable before submission, and verifying that a third guardian cannot be added or submitted.

**Acceptance Scenarios**:

1. **Given** the Guardians tab has no guardian entries, **When** the administrator adds a guardian, **Then** one editable guardian entry is shown with required relationship and contact fields.
2. **Given** one guardian entry already exists, **When** the administrator adds another guardian, **Then** a second editable guardian entry is shown and both entries can be reviewed before submission.
3. **Given** two guardian entries already exist, **When** the administrator attempts to add another guardian, **Then** the workflow prevents a third entry and explains that a student can have a maximum of two guardians.
4. **Given** two guardian entries are submitted for one student, **When** both pass validation, **Then** both guardian records are associated to the created student.

### Edge Cases

- The active school context is missing, inactive, changed during the workflow, or no longer authorized while the administrator is creating a student with guardians.
- Guardian data is partially entered and the administrator switches tabs, changes page size, changes active school, signs out, or navigates away.
- The administrator submits zero guardians; student creation remains allowed and no guardian association is created.
- The administrator submits duplicate guardian entries with the same name and contact data or the same existing guardian reference; duplicate relationship labels are allowed and do not make the request invalid.
- One guardian entry is valid and another is invalid; the workflow must not present partial success as complete student creation.
- Guardian contact email or phone fields are blank, malformed, or already used by another same-school guardian record where uniqueness rules apply.
- The administrator lacks guardian management permission but has student creation permission; Create Student remains available for student-only creation, but guardian capture remains unavailable.
- The standalone guardian list, create, lifecycle, and bulk lifecycle routes exist from earlier administration features but are no longer intended as sidebar entry points.
- Contract responses return validation, unauthorized, forbidden, tenant-mismatch, inactive-school, not-found, conflict, temporary-unavailable, unsupported field, or unsupported relationship behavior.
- Visible errors, diagnostics, and automated test output must not expose protected student, guardian, contact, role, permission, token, full request payload, or cross-tenant details.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Backend contract review is required because the new workflow creates or links guardian records from the student creation experience and enforces a maximum of two guardians per student. If existing operations can support this behavior without new backend behavior, planning must document the exact approved operation sequence and backend readiness gate; otherwise backend validation and contract behavior must be updated before frontend implementation.
- **Frontend repository impact**: Update the protected administration navigation to remove the standalone Guardians sidebar item, update the Create Student experience to use Student and Guardians tabs, support one or two guardian entries in the Guardians tab, preserve tabbed form state, and redirect direct guardian administration access to Create Student.
- **Specification or contract repository impact**: Add this behavior specification. OpenAPI must document the two-guardian maximum and approved new-guardian creation plus existing same-school guardian association behavior used by the tabbed student create workflow before implementation begins.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines the updated behavior first. Backend contract, implementation, and verification must pass before frontend implementation when OpenAPI or backend validation changes are needed. `schoolmaster-frontend` implements navigation and tabbed workflow only after backend readiness is recorded. Every affected implementation repository uses the exact feature branch name created by `/speckit-specify`; unaffected repositories receive no branch.

### API Contract Impact

- **OpenAPI update required**: Yes. The contract must document how guardian records are created or existing same-school guardian records are associated from student creation and must define the maximum of two guardians per student. If planning proves existing endpoints already provide this behavior, OpenAPI still needs the missing two-guardian maximum documented where the request or validation contract is defined.
- **Versioned endpoints affected**: `/api/v1/student-profiles` and `/api/v1/guardians` are affected if the workflow uses existing student and guardian creation or association operations. Any combined student-with-guardians operation or changed guardian association behavior must be added under `/api/v1/...` before implementation begins.
- **JSON response impact**: Success responses must allow the frontend to confirm the created student and associated guardian records. Failure responses must clearly distinguish student-field validation, guardian-field validation, maximum-guardian violations, duplicate guardian violations, authorization denials, tenant mismatch, inactive school, not-found references, and conflict states without exposing protected details.
- **Authentication/authorization impact**: Student creation remains protected by approved student administration permissions and active permitted school context. Guardian creation or association inside the student form requires both approved student create permission and guardian manage permission.
- **Compatibility impact**: Removing the sidebar item is a breaking navigation change for administrators who previously opened standalone guardian administration from the sidebar. Existing backend guardian operations may remain for direct access, lifecycle support, or future features, but the product navigation no longer exposes the standalone Guardians entry.

### Data & Tenancy Impact

- **Tenant scoping impact**: Student profiles, guardian records, and guardian-student associations are school-owned and must be created only inside the active permitted school context.
- **Cross-tenant or platform access impact**: Cross-school guardian references, student references, contact data, and association attempts remain blocked. Platform access does not grant implicit school-owned student or guardian creation outside an approved school context.
- **Soft delete impact**: Soft-deleted or inactive guardian records must not be silently reused during student creation unless an approved restore or lifecycle behavior is explicitly selected and documented. Guardian restore, deactivation, bulk lifecycle, user-link management, and guardian self-service remain outside this feature.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The administration sidebar MUST NOT show a standalone Guardians navigation item for any role or permission combination.
- **FR-002**: Direct or bookmarked access to standalone guardian administration screens MUST redirect to the approved Create Student workflow without loading guardian list data through hidden navigation.
- **FR-003**: Create Student MUST present a tabbed workflow with at least a Student tab for student details and a Guardians tab for guardian details.
- **FR-004**: The tabbed workflow MUST preserve entered Student and Guardians tab data while the administrator switches tabs.
- **FR-005**: The Guardians tab MUST allow the administrator to submit the student with zero guardian entries or add one or two guardian entries for the student being created.
- **FR-006**: The Guardians tab MUST prevent adding or submitting more than two guardians for one student.
- **FR-007**: Each guardian entry MUST either capture the guardian identity, relationship, and contact fields documented by the OpenAPI student-create guardian input contract for a new guardian or select an existing same-school guardian reference with an OpenAPI-documented relationship label.
- **FR-008**: The workflow MUST support common two-guardian family cases, including father and mother relationship labels, without requiring those exact labels when another OpenAPI-documented relationship label is valid.
- **FR-009**: The workflow MUST allow two guardian entries to use the same OpenAPI-documented relationship label for one student.
- **FR-010**: The workflow MUST validate student fields and guardian fields with field-level feedback and a clear indication of which tab contains errors.
- **FR-011**: The workflow MUST NOT present student creation as successful unless the required student record and all submitted guardian records or associations have succeeded according to approved behavior.
- **FR-012**: The workflow MUST show maximum-guardian, duplicate guardian, invalid relationship, invalid contact, missing required field, unauthorized, forbidden, tenant-mismatch, inactive-school, not-found, conflict, and temporary-unavailable states using safe feedback.
- **FR-013**: The workflow MUST require an authenticated user and active permitted school context before loading the student create form or enabling guardian entry submission.
- **FR-014**: Guardian creation or association inside Create Student MUST require both student create permission and guardian manage permission. Actors with student create permission but without guardian manage permission MUST still be able to create the student without guardians, while guardian capture remains blocked or hidden.
- **FR-015**: The workflow MUST keep guardian self-service, guardian user-link management, guardian lifecycle actions, bulk guardian lifecycle, student transfer, roster membership, teacher assignment, reports, messaging, billing, and undocumented guardian behavior outside this feature.
- **FR-016**: System MUST define any changed student or guardian creation and association contract in OpenAPI before implementation begins.
- **FR-017**: System MUST preserve consistent JSON responses for success and failure cases used by student creation with guardians.
- **FR-018**: System MUST preserve tenant isolation and document any intentional cross-tenant access path; no cross-tenant access is expected for this feature.
- **FR-019**: System MUST identify all affected repositories and the delivery sequence when implementation spans more than one repository.
- **FR-020**: Verification MUST cover sidebar visibility, direct guardian route behavior, tab switching, zero/one/two guardian submissions, rejected third guardian attempts, validation errors by tab, authorization denials, tenant isolation denials, stale responses, and no-sensitive-data diagnostics behavior.

### Key Entities *(include if feature involves data)*

- **StudentProfile**: School-owned learner profile created by an authorized administrator in the active school.
- **GuardianEntry**: One guardian captured inside the Create Student workflow, either as a new guardian with OpenAPI-documented identity, relationship, and contact details or as an existing same-school guardian reference with an OpenAPI-documented relationship label before submission.
- **GuardianRecord**: School-owned guardian record created or associated through the approved workflow.
- **GuardianStudentAssociation**: School-owned relationship between one student and one guardian, limited to a maximum of two active guardians per student for this workflow.
- **StudentCreateWorkflow**: Tabbed administrator workflow containing Student and Guardians tabs, shared validation state, unsaved-change protection, and final submission outcome.
- **SchoolContext**: Active permitted school boundary used to authorize and scope student creation, guardian creation, and guardian-student association.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of tested administrator sessions show no standalone Guardians item in the administration sidebar after the feature is delivered.
- **SC-002**: In at least one documented usability check using an authorized administrator or administrator-proxy account, the tester can open Create Student, switch between Student and Guardians tabs, and identify where to add guardian data in under 1 minute without assistance.
- **SC-003**: In at least one documented usability check using an authorized administrator or administrator-proxy account, the tester can create a student with two valid guardian entries in under 4 minutes without using a standalone guardian screen.
- **SC-004**: 100% of attempts to add or submit a third guardian for one student are blocked with a clear maximum-two explanation.
- **SC-005**: 100% of tested invalid student or guardian submissions identify the affected tab and field without losing data entered in the other tab.
- **SC-006**: 100% of tested unauthorized, forbidden, tenant-mismatch, inactive-school, not-found, validation, conflict, and temporary-unavailable cases show safe feedback without exposing protected student, guardian, contact, permission, role, token, full request payload, or cross-tenant details.
- **SC-007**: Contract review confirms every student-with-guardians submission path and the maximum-two guardian rule are documented before implementation begins.
- **SC-008**: Review confirms guardian self-service, guardian user-link management, lifecycle actions, bulk lifecycle actions, student transfer, roster membership, teacher assignment, reports, messaging, billing, and undocumented guardian behavior are not exposed by this feature.

## Assumptions

- The target actor is a school administrator creating student profiles in the active school context.
- "Same form" means one user-facing Create Student workflow that supports new guardian creation and existing same-school guardian linking; planning may choose the approved backend operation sequence only after OpenAPI and backend readiness are confirmed.
- Student creation remains allowed with zero guardians.
- A student can have at most two active guardian associations through this workflow.
- Father and mother are examples of relationship labels, not a complete controlled list; the accepted relationship labels or free-text constraints must be documented in OpenAPI for this workflow.
- Relationship labels do not need to be unique per student; duplicate guardian identity or existing guardian reference rules remain separate.
- Existing permission, unsaved-change, safe denial, validation, protected route, and active school behaviors from completed administration UI features remain the baseline.
- Standalone guardian backend operations may remain available for compatibility, direct authorized workflows, or future features, but the administration sidebar no longer exposes Guardians as a top-level item.

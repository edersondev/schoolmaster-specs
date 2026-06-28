# Feature Specification: Administration Lifecycle UI

**Feature Branch**: `020-administration-lifecycle-ui`
**Created**: 2026-06-28
**Status**: Draft
**Input**: User description: "Specify the Administration Lifecycle UI for schoolmaster-frontend. Define detail, update, activate/deactivate, delete/restore, and approved bulk workflows for administrative resources from the Administration Foundation UI, including confirm dialogs, detail pages, status tags, conflict handling, authorization visibility, editable-field rules, soft-delete behavior, audit-sensitive action handling, empty/error states, and OpenAPI operation IDs required before implementation."

## Clarifications

### Session 2026-06-28

- Q: How should bulk lifecycle operations handle records that conflict or fail validation within the selected batch? → A: Bulk lifecycle is all-or-nothing; any conflict or validation failure blocks the batch and shows batch-level feedback.
- Q: Should edit forms allow direct status changes when dedicated lifecycle actions exist? → A: Edit forms exclude status; activate, deactivate, delete, and restore happen only through lifecycle confirmations.
- Q: Should lifecycle effective dates allow future scheduling? → A: Effective date must be today or in the past; lifecycle actions are immediate.
- Q: What permissions are required for role update when permission assignments are editable? → A: Role update requires `roles.view`, `roles.manage`, and `permissions.view`.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Review and Edit Administrative Records (Priority: P1)

An authorized administrator can open a detail page from an existing
administration list, review the full permitted record, edit only fields approved
for that resource, and return to the prior list context without losing filters
or pagination.

**Why this priority**: Detail and update workflows are the minimum lifecycle
maintenance capability. They let administrators correct operational data while
preserving the list/create foundations from Administration Foundation UI.

**Independent Test**: For each approved resource, sign in with the required
view and manage permissions, open a detail route from the list, submit valid
and invalid edits, verify validation and conflict feedback, and return to the
same list query.

**Acceptance Scenarios**:

1. **Given** an authorized administrator on a resource list, **When** they open
   a permitted record detail, **Then** the page shows only fields returned by
   the approved detail operation and shows status, scope, tenant, and audit
   context without exposing inaccessible data.
2. **Given** editable fields are changed with valid values, **When** the
   administrator saves, **Then** one update is submitted to the approved update
   operation, success feedback is shown, stale validation messages clear, and
   the detail page reflects the updated record.
3. **Given** update validation fails, **When** the response is shown, **Then**
   matching controls display actionable messages, entered valid values remain
   available, and protected or cross-tenant details are not revealed.
4. **Given** a record changed after the detail page loaded, **When** saving
   produces a conflict, **Then** the page explains that the record must be
   refreshed or reviewed before retrying and does not silently overwrite newer
   data.
5. **Given** the administrator leaves an edited detail form before saving,
   **When** browser or in-app navigation is attempted, **Then** a confirmation
   is required before discarding changes.

---

### User Story 2 - Perform Single-Record Lifecycle Actions (Priority: P2)

An authorized administrator can activate, deactivate, soft delete, or restore
one eligible administrative record when the current status and approved
contract allow that action.

**Why this priority**: Operational maintenance requires explicit status and
soft-delete controls, but those controls are audit-sensitive and must be
guarded by confirmation, eligibility, and permission rules.

**Independent Test**: For each resource and each eligible status, open the list
or detail action, confirm the action with an effective date and reason, and
verify success, validation, conflict, forbidden, not-found, and tenant-denial
outcomes.

**Acceptance Scenarios**:

1. **Given** a record is eligible for activation, deactivation, deletion, or
   restoration, **When** the administrator opens the action, **Then** a
   confirmation dialog names the resource, action, current status, expected
   outcome, required effective date, and required reason before submission.
2. **Given** the lifecycle action succeeds, **When** the result is returned,
   **Then** the status tag, row actions, detail summary, lifecycle history, and
   list membership update to match the approved response.
3. **Given** the action is no longer eligible, **When** the backend returns a
   conflict, **Then** the UI keeps the record protected, explains the conflict,
   and offers a refresh or return path without claiming success.
4. **Given** the record is deleted, inaccessible, or outside the active school,
   **When** the administrator tries to open or act on it, **Then** the UI shows
   the approved not-found, forbidden, or tenant-denial state without revealing
   whether a hidden record exists.
5. **Given** the action is audit-sensitive, **When** it is submitted, **Then**
   the reason and effective date are required and the UI does not provide a
   shortcut that bypasses the confirmation.

---

### User Story 3 - Apply Approved Bulk Lifecycle Actions (Priority: P3)

An authorized administrator can select multiple eligible records from an
approved list and apply one approved lifecycle action to the selected records
where a bulk operation exists.

**Why this priority**: Bulk maintenance reduces repetitive work after
single-record lifecycle behavior is established, while keeping risk bounded to
resources with explicit bulk contracts.

**Independent Test**: On each approved bulk-enabled list, select valid and
conflicting records, attempt each published bulk action within selection
limits, verify confirmation content, and validate full success, batch-level
validation, conflict, and denial states.

**Acceptance Scenarios**:

1. **Given** a list supports a published bulk lifecycle operation, **When** the
   administrator selects eligible records, **Then** only actions permitted for
   the selected resource type, statuses, tenant context, and permissions are
   shown.
2. **Given** selected records are submitted for one bulk action, **When** the
   confirmation is displayed, **Then** it states the action, selected count,
   affected resource type, effective date, reason, and all-or-nothing batch
   outcome.
3. **Given** a bulk lifecycle operation succeeds for all selected records,
   **When** the result is shown, **Then** all affected rows, status tags,
   selections, and list totals are refreshed or reconciled.
4. **Given** a bulk operation returns validation or conflict failure for the
   selected batch, **When** the result is shown, **Then** no selected record is
   presented as successfully changed, batch-level feedback explains the
   failure, and the selection remains available for review or retry.
5. **Given** the selected record count exceeds the approved limit or contains
   records from another context, **When** submission is attempted, **Then** the
   UI blocks or displays validation without sending undocumented parameters or
   exposing hidden records.

### Edge Cases

- A detail or lifecycle response returns after the user changes route, active
  school, selection, permission state, or session; stale results must not
  replace the current view.
- A lifecycle action is visible from stale client state but the backend denies
  it; backend authorization and eligibility remain authoritative.
- A lifecycle action is submitted with a future effective date; the UI must
  prevent or display validation for the invalid date and must not present it as
  a scheduled action.
- A deleted record cannot be found through an approved list or detail response;
  restore entry points remain hidden until an approved response exposes that
  record to the requester.
- A user tries to delete or deactivate their own administrative user record;
  the UI must surface the approved conflict or forbidden outcome and must not
  assume self-action is allowed.
- A role being edited or deleted is still assigned to users; conflict feedback
  must direct the administrator to refresh or resolve dependencies without
  exposing users outside the permitted scope.
- An academic year or academic period has dependent schedules, rosters, grades,
  or reporting data; conflict feedback must preserve existing data and avoid
  irreversible claims.
- A school switch is requested with an unsaved detail edit or open lifecycle
  confirmation; the user must confirm before discarding changes and no pending
  action may submit into the new school context.
- Bulk selection spans pages; only selected records retained under approved
  list state and active tenant context can be submitted.
- A bulk result updates records that no longer match the current filter; the
  list must reconcile totals and empty states without looping requests.
- Lifecycle reason text includes sensitive personal data; the UI must not echo
  it into diagnostics, logs, or shared feedback beyond the submitted action
  confirmation and approved result display.
- Temporary service failures must preserve the current detail form, selected
  records, lifecycle reason, and list query enough for a deliberate retry.
- Permission definitions remain read-only and must not expose detail, edit,
  lifecycle, delete, restore, or bulk controls.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No new backend behavior is defined by this
  frontend specification. Backend readiness is required for published detail,
  update, lifecycle, delete, restore, and bulk lifecycle operations and their
  conflict, validation, forbidden, not-found, and tenant-denial envelopes.
- **Frontend repository impact**: Extend the existing administration module
  with detail pages, edit forms, status tags, row actions, lifecycle
  confirmations, selection and bulk action surfaces, conflict feedback, and
  tests for schools, users, roles, academic years, academic periods, and
  guardians. Permissions remain read-only.
- **Specification or contract repository impact**: Add this specification and,
  during planning, add a frontend lifecycle consumption contract that maps each
  UI surface to approved OpenAPI operation IDs. OpenAPI changes are required
  only if review discovers missing fields, filters, statuses, errors, or
  operations needed before implementation.
- **Delivery ownership and sequencing**: `schoolmaster-specs` approves the
  lifecycle behavior first. Backend contracts and implementations must be
  confirmed before `schoolmaster-frontend` exposes each action. Frontend work
  may implement independently approved resource slices without enabling
  blocked actions.

### API Contract Impact

- **OpenAPI update required**: No expected contract expansion. Contract review
  must confirm every consumed operation, request field, response field, status,
  conflict envelope, and error envelope before implementation.
- **Versioned endpoints affected**:
  - Schools: `getSchool`, `updateSchool`, `activateSchool`,
    `deactivateSchool`, `deleteSchool`, and `restoreSchool`.
  - Users: `getUser`, `updateUser`, `activateUser`, `deactivateUser`,
    `deleteUser`, `restoreUser`, and `bulkLifecycleUsers`.
  - Roles: `getRole`, `updateRole`, `activateRole`, `deactivateRole`,
    `deleteRole`, `restoreRole`, and `bulkLifecycleRoles`.
  - Academic years: `getAcademicYear`, `updateAcademicYear`,
    `activateAcademicYear`, `deactivateAcademicYear`, `deleteAcademicYear`,
    `restoreAcademicYear`, and `bulkLifecycleAcademicYears`.
  - Academic periods: `getAcademicPeriod`, `updateAcademicPeriod`,
    `activateAcademicPeriod`, `deactivateAcademicPeriod`,
    `deleteAcademicPeriod`, `restoreAcademicPeriod`, and
    `bulkLifecycleAcademicPeriods`.
  - Guardians: `getGuardian`, `updateGuardian`, `activateGuardian`,
    `deactivateGuardian`, `deleteGuardian`, `restoreGuardian`, and
    `bulkLifecycleGuardians`.
  - Existing foundation operations remain required for list navigation,
    selectors, and post-action refreshes: `listSchools`, `listUsers`,
    `listRoles`, `listPermissions`, `listAcademicYears`,
    `listAcademicPeriods`, `listGuardians`, and `listStudentProfiles` where
    already approved.
- **JSON response impact**: The frontend must consume only published success,
  detail, lifecycle outcome, bulk lifecycle outcome, validation, unauthorized,
  forbidden, tenant-mismatch, not-found, conflict, and temporary failure
  envelopes. Diagnostics may include safe operation identifiers and request
  identifiers but not form values, reason text, emails, tokens, tenant data, or
  permission payloads.
- **Authentication/authorization impact**: All routes and actions require
  authentication. View permissions are required for list and detail access.
  Manage permissions are required for update, activate, deactivate, delete,
  restore, and bulk lifecycle actions. Backend authorization remains
  authoritative when client visibility is stale.
- **Compatibility impact**: Additive frontend behavior over approved contracts.
  Account invitation, password setup, password reset, account lock,
  reactivation, guardian user-link management, student management, reporting,
  and platform support workflows remain outside this feature.

### Data & Tenancy Impact

- **Tenant scoping impact**: Schools are platform-managed tenant roots. Users,
  roles, academic years, academic periods, and guardians remain scoped to the
  active permitted school. Tenant headers and active context come only from the
  authenticated session foundation.
- **Cross-tenant or platform access impact**: The UI must not combine records
  across schools, use platform access as a school-owned bypass, infer hidden
  record existence, or submit lifecycle actions for records outside the current
  permitted context.
- **Soft delete impact**: Delete actions are soft-delete actions. Deleted
  records may be restored only when an approved list, detail, or action result
  exposes that record to the requester. Restore controls remain hidden when the
  frontend cannot safely locate a deleted record through approved contracts.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The frontend MUST provide protected detail routes for schools,
  users, roles, academic years, academic periods, and guardians.
- **FR-002**: The frontend MUST keep permission definitions read-only and MUST
  NOT expose permission detail, update, lifecycle, delete, restore, or bulk
  actions.
- **FR-003**: Detail routes MUST require the resource view permission and MUST
  require active school context for tenant-owned resources.
- **FR-004**: Update, activate, deactivate, delete, restore, and bulk lifecycle
  actions MUST require both the resource view permission and the corresponding
  manage permission, except role update MUST also require `permissions.view`
  because the update workflow includes permission assignment controls.
- **FR-005**: Direct visits to unauthorized detail or action routes MUST show
  the established unauthorized, forbidden, inactive-context, tenant-mismatch,
  or not-found state without rendering protected resource data.
- **FR-006**: Detail pages MUST show the current status with a reusable status
  tag and MUST expose actions only when the current record state, permissions,
  tenant context, and approved operation allow them.
- **FR-007**: Update forms MUST submit only editable non-status fields
  published by the resource update contract: school name, contact information,
  and address; user full name, email, and role assignments; role name and
  permission assignments; academic year name and dates; academic period name,
  sequence, and dates; guardian name, relationship type, and contact
  information.
- **FR-008**: Role update MUST require `roles.view`, `roles.manage`, and
  `permissions.view`, MUST allow only approved permission assignments from the
  permitted school-scope permission set, and MUST NOT expose platform role
  scope editing.
- **FR-009**: User update MUST allow role assignment only and MUST NOT expose
  direct per-user permission assignment.
- **FR-010**: Academic period update MUST NOT expose academic-year reassignment
  unless a future approved contract publishes that field.
- **FR-011**: Status transitions MUST be performed only through dedicated
  lifecycle actions when a matching lifecycle operation exists, because those
  operations require confirmation, effective date, and reason.
- **FR-012**: Every lifecycle confirmation MUST require the published effective
  date and reason fields before submission and MUST describe the resource,
  action, current status, and expected outcome.
- **FR-013**: Lifecycle effective dates MUST be today or in the past in the
  resolved school or platform context, and the UI MUST NOT present future
  effective dates as scheduled lifecycle actions.
- **FR-014**: Lifecycle actions MUST support only the published actions:
  activate, deactivate, delete, and restore.
- **FR-015**: Delete actions MUST be presented as soft-delete actions and MUST
  avoid irreversible deletion language unless a future contract explicitly
  approves permanent deletion.
- **FR-016**: Restore actions MUST be shown only for records the requester can
  access through approved contracts or current action results.
- **FR-017**: Bulk lifecycle actions MUST be available only for users, roles,
  academic years, academic periods, and guardians, because those are the
  resources with approved bulk lifecycle operation IDs.
- **FR-018**: Bulk lifecycle submission MUST send one resource type, one
  lifecycle action, 1 to 50 unique selected record identifiers, one effective
  date, and one reason, matching the approved request contract.
- **FR-019**: Schools MUST NOT expose bulk lifecycle actions unless a future
  approved school bulk operation is published.
- **FR-020**: Bulk selection MUST clear or revalidate when filters, page size,
  active school, session, permission state, or resource type changes.
- **FR-021**: List and detail actions MUST preserve approved list query state
  when navigating to detail, edit, confirmation, success, cancellation, or
  return flows.
- **FR-022**: Successful update or lifecycle actions MUST refresh or reconcile
  the affected detail page and list row without assuming undocumented response
  fields.
- **FR-023**: Validation responses MUST map field errors to controls, preserve
  valid entered values, and summarize unmatched errors accessibly.
- **FR-024**: Conflict responses MUST clearly distinguish stale data,
  dependency conflicts, ineligible status transitions, and bulk batch failure
  when the published envelope supports that distinction.
- **FR-025**: Not-found and denied states MUST not reveal whether a record
  exists in another tenant or inaccessible scope.
- **FR-026**: Pending update, lifecycle, delete, restore, and bulk submissions
  MUST prevent duplicate submission while keeping deliberate cancellation or
  retry behavior predictable.
- **FR-027**: Unsaved detail edit forms and open lifecycle confirmations MUST
  require confirmation before browser navigation, sidebar navigation, list
  return, active school switch, or another in-app destination discards them.
- **FR-028**: Stale detail, update, lifecycle, and bulk responses MUST NOT
  overwrite state belonging to a newer route, selection, tenant, session,
  permission, or request context.
- **FR-029**: Lifecycle reason text, form values, email addresses, tenant
  identifiers, tokens, and permission payloads MUST NOT be exposed in
  diagnostics, route query, shared notifications, or logs intended for support.
- **FR-030**: The UI MUST use consistent empty, filtered-empty, unavailable,
  validation, forbidden, tenant, not-found, conflict, and temporary failure
  feedback patterns across all lifecycle-enabled resources.
- **FR-031**: Administration lifecycle pages and controls MUST remain usable at
  390px mobile, 768px tablet, and 1440px desktop viewport widths, including
  row actions, status tags, selection controls, confirmations, and validation
  summaries.
- **FR-032**: All lifecycle controls MUST be keyboard operable, visibly
  focused, programmatically labeled, and announced appropriately during
  loading, success, validation, conflict, and failure states.
- **FR-033**: The feature MUST NOT expose account lifecycle workflows,
  invitation flows, password setup or reset flows, account lock or
  reactivation flows, guardian user-link workflows, student management, or
  undocumented backend routes.
- **FR-034**: Frontend verification MUST cover contract mapping, route
  metadata, permission visibility, direct-route denial, active-school gating,
  detail loading, update validation, unsaved guards, status tags, confirmation
  dialogs, single lifecycle actions, soft-delete and restore visibility, bulk
  selection and outcomes, conflict handling, stale-response protection,
  responsive layout, keyboard operation, and absence of out-of-scope actions.

### Key Entities *(include if feature involves data)*

- **School**: Platform-managed tenant root that can be reviewed, edited,
  activated, deactivated, soft deleted, and restored when the requester has
  approved platform permissions.
- **User**: Identity in the permitted scope with editable profile fields, role
  assignments, status display, and lifecycle controls; account lifecycle
  workflows are excluded.
- **Role**: School-scoped authorization grouping with editable name,
  permission assignments, status display, and lifecycle controls.
- **Permission**: Read-only capability definition used for role assignment;
  no lifecycle UI is allowed for this feature.
- **Academic Year**: School-owned academic cycle with editable dates and status
  display plus lifecycle and approved bulk controls.
- **Academic Period**: School-owned subdivision of an academic year with
  editable sequence, dates, and status display plus lifecycle and approved bulk
  controls.
- **Guardian**: School-owned responsible contact with editable contact and
  relationship fields plus lifecycle and approved bulk controls; user-link
  management is excluded.
- **Lifecycle Action**: One audit-sensitive activate, deactivate, soft-delete,
  or restore action submitted with a today-or-past effective date and reason.
- **Bulk Lifecycle Selection**: A same-resource, same-context set of 1 to 50
  selected record identifiers submitted for one approved lifecycle action.
- **Lifecycle Outcome**: Approved result describing affected resource type,
  record identifier, action, resulting status, and lifecycle history.
- **Administration Detail State**: Route-local detail data, update draft,
  validation messages, action eligibility, loading state, errors, and original
  list return context.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Authorized administrators can open a detail view for every
  lifecycle-enabled resource from its list in no more than two interactions.
- **SC-002**: In moderated usability verification with five representative
  administrators, at least 90% of attempted valid update and single lifecycle
  tasks complete without facilitator guidance.
- **SC-003**: With the application already bootstrapped and mocked services
  settling within 1.5 seconds, every detail route displays usable content,
  empty/not-found feedback, or recoverable error feedback within 2 seconds of
  navigation.
- **SC-004**: Validation verification confirms 100% of documented update and
  lifecycle field errors appear beside or are programmatically associated with
  the relevant control while valid entered values remain intact.
- **SC-005**: Authorization and tenancy verification confirms 100% of tested
  denied, inactive, mismatched, and cross-tenant scenarios render no protected
  resource data and submit no unauthorized action.
- **SC-006**: Conflict verification confirms 100% of stale-record,
  dependency-conflict, ineligible-transition, and bulk batch-failure scenarios
  show conflict feedback without falsely reporting success for any selected
  record.
- **SC-007**: Bulk verification confirms all submitted batches respect the
  approved 1 to 50 selected-record limit, one resource type, one action, one
  today-or-past effective date, and one reason.
- **SC-008**: Responsive and accessibility review at 390px, 768px, and 1440px
  confirms all detail, edit, row action, confirmation, selection, bulk, and
  recovery controls are keyboard operable and visually usable.
- **SC-009**: Contract review maps 100% of frontend requests in this feature to
  approved operation IDs with no undocumented endpoint, field, filter, status,
  action, or error dependency.

## Assumptions

- Features `015-frontend-architecture-baseline`, `016-admin-shell-dashboard`,
  `017-auth-session-ui`, and `018-administration-foundation-ui` are complete
  and provide the reusable architecture, admin shell, authenticated routing,
  permissions, tenant context, lists, create workflows, shared feedback, and
  query-state foundations.
- Feature 017 still owns active-school resolution. If no active permitted
  school can be confirmed, tenant-owned lifecycle pages remain blocked and this
  feature does not introduce school selection.
- Backend operations listed in the API Contract Impact section are approved,
  implemented, and contract-compliant before the corresponding frontend action
  is exposed.
- Manage permissions are the existing resource manage permissions established
  by Administration Foundation UI: `schools.manage`, `users.manage`,
  `roles.manage`, `academic_years.manage`, `academic_periods.manage`, and
  `guardians.manage`.
- Existing list operations remain the source for list refresh, return
  navigation, and row reconciliation after lifecycle actions.
- Restore UI depends on approved visibility of deleted records through
  published responses; this specification does not require inventing a deleted
  record browser if the contract does not expose one.
- Search controls remain out of scope unless already published by the consumed
  operation. The UI must not add undocumented filters to find records for
  update, delete, restore, or bulk workflows.

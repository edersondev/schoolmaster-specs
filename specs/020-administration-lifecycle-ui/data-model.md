# Data Model: Administration Lifecycle UI

This feature adds no database entities. Models below define frontend state,
forms, service mappings, and relationships to approved OpenAPI records.

## AdministrationLifecycleRoute

**Fields**:

- `name`: Stable route name.
- `resource`: School, user, role, academic year, academic period, or guardian.
- `mode`: Detail, edit, lifecycle action, or list with bulk lifecycle.
- `requiresAuth`: Always true.
- `requiresSchoolContext`: False for schools; true for tenant-owned resources.
- `permissions`: Exact permission codes required by the route or action.
- `layout`: Existing admin-system layout.
- `titleKey`, `breadcrumb`, `sidebar`: Localized shell metadata.
- `returnQuery`: Validated list query used for list return navigation.

**Rules**:

- Permission definitions remain list/reference-only and have no lifecycle
  routes.
- Tenant-owned routes cannot load before active school resolution.
- Unauthorized navigation remains hidden, but direct-route denial is handled.
- Route metadata does not carry tenant authorization state.

## AdministrationDetailState

**Fields**:

- `resourceType`: Current resource type.
- `resourceId`: UUID from route params.
- `record`: Normalized detail record.
- `status`: Idle, loading, ready, forbidden, tenant-mismatch, not-found,
  conflict, unavailable, or error.
- `error`: Normalized `AdministrationLifecycleFeedback`.
- `requestKey`: Current operation, route params, tenant generation, and
  permission generation.
- `returnQuery`: Validated originating list query.

**Rules**:

- Only the latest request matching current route and tenant may update state.
- Not-found and forbidden feedback must not reveal cross-tenant existence.
- School change clears tenant-owned detail state before new-school data
  renders.
- Detail state never stores lifecycle reason text after a dialog closes.

## AdministrationUpdateFormState

**Fields**:

- `initialValues`: Current editable non-status values from detail.
- `values`: Current user-entered non-status values.
- `fieldErrors`: Contract field to localized messages.
- `formError`: Optional normalized non-field error.
- `status`: Idle, submitting, succeeded, validation-error, forbidden,
  tenant-mismatch, not-found, conflict, unavailable, or error.
- `isDirty`: Derived comparison with initial values.
- `submitted`: Successful-submit flag that disables leave warning.

**Rules**:

- Status fields are display-only and are never submitted by edit forms.
- Duplicate submit is blocked while submitting.
- Dirty route exit, school switch, and list return require confirmation.
- Successful update clears dirty/error state and reconciles detail/list data.
- Tenant switch clears drafts after discard confirmation.

## AdministrationLifecycleDialogState

**Fields**:

- `resourceType`: Current resource type.
- `resourceId`: Target UUID.
- `resourceLabel`: Safe display label.
- `currentStatus`: Current status displayed before action.
- `action`: Activate, deactivate, delete, or restore.
- `effectiveAt`: Required today-or-past date.
- `reason`: Required 1 to 500 character reason.
- `fieldErrors`: Validation messages for effective date and reason.
- `status`: Closed, open, submitting, succeeded, validation-error, forbidden,
  not-found, conflict, unavailable, or error.

**Rules**:

- Future effective dates are blocked or mapped to validation feedback.
- The dialog never presents lifecycle actions as scheduled future changes.
- Reason text is not written to route query, diagnostics, shared logs, or
  long-lived stores.
- Closing a dirty dialog through navigation requires discard confirmation.

## AdministrationBulkLifecycleSelection

**Fields**:

- `resourceType`: Users, roles, academic years, academic periods, or guardians.
- `selectedIds`: Unique selected UUIDs, minimum 1 and maximum 50 for submit.
- `selectedSummaries`: Safe labels and current statuses for selected records.
- `action`: One selected lifecycle action.
- `effectiveAt`: Required today-or-past date.
- `reason`: Required 1 to 500 character reason.
- `status`: Idle, confirming, submitting, succeeded, validation-error,
  forbidden, conflict, unavailable, or error.

**Rules**:

- Schools have no bulk lifecycle selection.
- Permission definitions have no lifecycle selection.
- Selection is resource-specific and tenant-specific.
- Filter, page-size, active school, session, permission, or resource changes
  clear or revalidate selection.
- Bulk lifecycle is all-or-nothing; failed batches do not mark individual
  selected records as changed.

## AdministrationLifecycleFeedback

**Fields**:

- `kind`: Loading, empty, filtered-empty, validation, unauthorized,
  forbidden, tenant-mismatch, inactive-context, not-found, conflict,
  temporary-unavailable, or unknown.
- `operationId`: Safe operation identifier where useful.
- `requestId`: Safe request identifier where available.
- `messageKey`: Localized message key.
- `fieldErrors`: Optional validation details.

**Rules**:

- Does not include form values, reason text, emails, tokens, tenant payloads,
  permission payloads, or hidden record identifiers.
- Conflict feedback distinguishes stale data, dependency conflict, ineligible
  status transition, and batch-level bulk failure when the published envelope
  supports that distinction.

## Resource Models and Update Forms

### School

**Fields**: `id`, `name`, `code`, `status`, `contactEmail`, `contactPhone`,
`address`.

**Update fields**: `name`, `contactEmail`, `contactPhone`, `address`.

**Lifecycle**: Single-record activate, deactivate, soft delete, and restore.
No bulk lifecycle.

### User

**Fields**: `id`, `schoolId`, `fullName`, `email`, `status`, `roles`.

**Update fields**: `fullName`, `email`, `roleIds`.

**Lifecycle**: Single-record activate, deactivate, soft delete, restore, and
approved all-or-nothing bulk lifecycle. Account lifecycle remains excluded.

### Role

**Fields**: `id`, `schoolId`, `scope`, `name`, `status`, `permissions`.

**Update fields**: `name`, `permissionIds`.

**Rules**: Role update requires `roles.view`, `roles.manage`, and
`permissions.view`. Scope is display-only.

**Lifecycle**: Single-record activate, deactivate, soft delete, restore, and
approved all-or-nothing bulk lifecycle.

### Permission

**Fields**: `id`, `code`, `name`, `scope`, `status`.

**Rules**: Read-only reference data only. No detail, update, lifecycle, delete,
restore, or bulk lifecycle UI.

### AcademicYear

**Fields**: `id`, `schoolId`, `name`, `startDate`, `endDate`, `status`.

**Update fields**: `name`, `startDate`, `endDate`.

**Lifecycle**: Single-record activate, deactivate, soft delete, restore, and
approved all-or-nothing bulk lifecycle.

### AcademicPeriod

**Fields**: `id`, `schoolId`, `academicYearId`, `name`, `sequence`,
`startDate`, `endDate`, `status`.

**Update fields**: `name`, `sequence`, `startDate`, `endDate`.

**Lifecycle**: Single-record activate, deactivate, soft delete, restore, and
approved all-or-nothing bulk lifecycle. Academic-year reassignment remains
blocked.

### Guardian

**Fields**: `id`, `schoolId`, `fullName`, `relationshipType`, `contactEmail`,
`contactPhone`, `status`.

**Update fields**: `fullName`, `relationshipType`, `contactEmail`,
`contactPhone`.

**Lifecycle**: Single-record activate, deactivate, soft delete, restore, and
approved all-or-nothing bulk lifecycle. Guardian user links remain excluded.

## Lifecycle Outcome

**Fields**:

- `resourceType`
- `resourceId`
- `action`
- `status`
- `lifecycleHistory`

**Rules**:

- Reconciles status tags, row actions, detail summary, and list membership.
- Does not authorize UI to infer hidden or cross-tenant record existence.
- Bulk outcomes are used only after all selected records succeed.

# Data Model: System Administrator Master Access

This feature introduces no new tenant root and no new business resource
schema. It clarifies authorization entities, contexts, and audit evidence used
by existing backend and frontend workflows.

## SystemAdministrator

**Purpose**: Master user role that satisfies all feature-specific permission
checks.

**Key Attributes**:

- `role`: Canonical System Administrator role identifier.
- `accountState`: Active, inactive, locked, or other existing account state.
- `sessionState`: Valid, expired, revoked, or otherwise rejected session state.
- `availableSchoolContexts`: Active schools available for selection.
- `selectedSchoolContext`: Active school selected for school-owned work, when
  required.
- `selectedSubjectContext`: Student, guardian, user, or other subject selected
  for identity-owned self-service work, when required.

**Rules**:

- Satisfies every feature-specific permission check.
- Does not bypass authentication, account state, session validity, lockout,
  school state, release state, tenant context, subject context, approval
  workflows, explicit confirmations, support opt-ins, file safety gates,
  closed-period safety checks, or other business controls.
- May select any active school as the resolved school context.
- Cannot receive unscoped school-owned data unless an operation is documented
  as platform-wide.

## FeatureSpecificPermission

**Purpose**: Named capability normally required by a route, action, or backend
operation for non-System Administrator roles.

**Key Attributes**:

- `code`: Stable permission identifier used by existing route metadata,
  policies, or operation checks.
- `scope`: Platform, school, self-service, support, reporting, or other
  existing permission scope.
- `requiredFor`: Routes, actions, operations, or navigation items that require
  the permission.

**Rules**:

- Required for non-System Administrator roles according to existing behavior.
- Considered satisfied for System Administrator.
- Must remain documented so non-System Administrator roles and UI visibility
  continue to behave predictably.

## ResolvedSchoolContext

**Purpose**: Active school tenant target used to scope school-owned operations.

**Key Attributes**:

- `schoolId`: Public school identifier.
- `status`: Active or inactive according to existing school lifecycle rules.
- `source`: User selection or approved session resolution path.

**Rules**:

- Required before System Administrator loads or mutates school-owned resources
  that need a tenant target.
- Must be active for school-owned operational workflows.
- Limits school-owned reads and writes to the selected school unless the
  operation is explicitly platform-wide.
- Switching school context clears stale school-owned data before new data is
  loaded.

## SelectedSubjectContext

**Purpose**: Person-specific target required before System Administrator opens
identity-owned self-service data.

**Key Attributes**:

- `subjectId`: Public identifier for the selected student, guardian, user, or
  other subject.
- `subjectType`: Student, guardian, user, or other documented self-service
  subject type.
- `schoolContext`: School context when the subject belongs to school-owned
  data.

**Rules**:

- Required before identity-owned self-service routes load protected data.
- Must be compatible with the current tenant context when the subject is
  school-owned.
- Prevents loading another user's self-service data without explicit subject
  selection.

## PlatformWideOperation

**Purpose**: Documented operation that intentionally spans more than one school
or platform resource.

**Key Attributes**:

- `operationId`: Stable operation identifier in the shared contract.
- `scope`: Platform-wide.
- `allowedOutput`: Documented cross-school or platform fields.

**Rules**:

- Must be documented as platform-wide before returning unscoped or cross-school
  output.
- Still enforces authentication, account state, session state, release state,
  and applicable approval/safety gates.

## MasterAccessAuditEvidence

**Purpose**: Reviewable evidence that a System Administrator used master access
for a write or lifecycle action.

**Key Attributes**:

- `actorId`: System Administrator public user identifier.
- `action`: Create, update, delete, restore, activate, deactivate, import,
  retry, cancel, approve, revoke, or other state-changing action.
- `targetType`: Resource type affected.
- `targetId`: Public identifier of affected resource where available.
- `schoolContext`: Selected school context when the target is school-owned.
- `subjectContext`: Selected subject context when the target is identity-owned.
- `masterAccessUsed`: True for covered actions.
- `occurredAt`: Existing audit timestamp.
- `outcome`: Success, failure, conflict, denied by non-permission prerequisite,
  or other existing outcome.

**Rules**:

- Required for all System Administrator writes and lifecycle actions.
- Not newly required for read-only navigation, list, or detail views unless an
  existing operation already requires audit evidence.
- Must not disclose cross-tenant resource details beyond the selected context
  and documented audit payload.

## State Transitions

- **Unauthenticated -> Authenticated System Administrator**: Session resolves
  master role and can evaluate protected routes.
- **No School Context -> School Context Selected**: Any active school may be
  selected; school-owned data loads only after selection.
- **School Context A -> School Context B**: School-owned data from A is cleared
  before B's data is loaded.
- **No Subject Context -> Subject Context Selected**: Identity-owned
  self-service data loads only after subject selection.
- **Permission Check -> Master Override**: Feature-specific permission is
  treated as satisfied for System Administrator.
- **Write/Lifecycle Action -> Audit Evidence Recorded**: Master-access marker
  is recorded with existing audit metadata.

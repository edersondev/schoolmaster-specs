# Research: Administration Lifecycle UI

## Decision: Implement a frontend-only lifecycle consumption slice

**Rationale**: Backend Administration Lifecycle Management already publishes
detail, update, single lifecycle, soft-delete, restore, and selected bulk
lifecycle operation IDs in OpenAPI. The frontend feature should consume those
contracts without adding backend behavior or contract mutations.

**Alternatives considered**:

- Expand OpenAPI during frontend planning: rejected because current aggregate
  OpenAPI already contains the needed operation IDs and schemas.
- Implement against undocumented backend routes for missing UI affordances:
  rejected by constitution and roadmap rules.

## Decision: Keep status changes out of edit forms

**Rationale**: Update schemas currently publish `status`, but the clarified
frontend behavior requires audit-sensitive status transitions to go through
dedicated lifecycle confirmations. This keeps reason and effective date
required, avoids silent status changes, and aligns user expectations around
activation, deactivation, soft delete, and restore.

**Alternatives considered**:

- Allow direct status editing in update forms: rejected because it bypasses the
  confirmation workflow and weakens audit intent.
- Allow non-delete status editing only: rejected because it creates two status
  paths with different validation and user feedback.

## Decision: Treat lifecycle actions as immediate with today-or-past dates

**Rationale**: The published lifecycle operations return immediate lifecycle
outcomes and do not publish scheduling semantics. The UI therefore constrains
`effective_at` to today or a past date in the resolved school or platform
context and must not present future lifecycle scheduling.

**Alternatives considered**:

- Allow future effective dates as scheduled transitions: rejected because no
  scheduling contract, queue state, cancellation behavior, or future-status UI
  is published.
- Force current date only: rejected because the contract preserves an effective
  date field for audit-relevant historical effective dates.

## Decision: Use all-or-nothing bulk lifecycle behavior

**Rationale**: Backend lifecycle specification and the clarified frontend spec
define selected bulk lifecycle requests as all-or-nothing. Conflict or
validation failure blocks the batch and shows batch-level feedback; no selected
record is presented as successfully changed unless the operation succeeds.

**Alternatives considered**:

- Show partial success for failed batches: rejected because the contract does
  not publish per-record failure details.
- Hide bulk lifecycle until per-record failures are published: rejected because
  approved all-or-nothing operations already support useful bounded bulk
  maintenance.

## Decision: Use route-local detail, form, dialog, and selection state

**Rationale**: Administration Foundation UI already uses route query and
route-local state for CRUD surfaces, with Pinia reserved for shared session and
shell state. Lifecycle pages can follow the same boundary: detail state,
update drafts, dialog inputs, and bulk selections are local to the current
resource route and active tenant generation.

**Alternatives considered**:

- Add resource Pinia stores for each lifecycle module: rejected because state
  does not need to cross unrelated route boundaries and would increase stale
  tenant risk.
- Store lifecycle dialog inputs in the URL: rejected because reason text and
  action context may contain sensitive data and are not navigation state.

## Decision: Restore controls depend on approved record visibility

**Rationale**: The current frontend feature does not define a deleted-record
browser or new list filters. Restore actions are shown only when an approved
detail, list, or lifecycle result exposes a restorable record to the requester.
If the frontend cannot safely locate the record through approved contracts,
restore remains hidden.

**Alternatives considered**:

- Add a deleted-record list filter: rejected unless a consumed operation
  explicitly publishes that filter and backend behavior is verified.
- Allow manual restore by identifier: rejected because it can reveal hidden or
  cross-tenant record existence.

## Decision: Centralize lifecycle error and privacy handling

**Rationale**: Lifecycle actions include audit reason text, identifiers, tenant
context, and conflict details. A shared error mapper and feedback contract keeps
validation, conflict, forbidden, tenant, not-found, temporary failure, and
unknown outcomes consistent while preventing reason text, emails, tokens,
permission payloads, or tenant data from leaking into diagnostics.

**Alternatives considered**:

- Let each resource page map errors independently: rejected because lifecycle
  failures are security-sensitive and should remain uniform.
- Surface raw backend errors to components: rejected by existing frontend
  architecture and privacy constraints.

## Decision: Keep lifecycle UI in existing administration route modules

**Rationale**: The existing `schools`, `access-administration`, `academics`,
and `guardians` route modules already own resource navigation and permission
metadata. Extending them preserves route grouping and lets lifecycle work land
resource-by-resource without a parallel navigation system.

**Alternatives considered**:

- Create a separate lifecycle route module tree: rejected because it would
  duplicate resource ownership and make permissions harder to audit.
- Embed lifecycle behavior only in list pages: rejected because detail and edit
  workflows need independent, bookmarkable protected routes.

# Data Model: User Recovery UI

Feature 038 adds no database entity, backend persistence, or browser storage.
All entities below are transient frontend state derived from the published API
and discarded when the create workflow or context ends.

## Normalized Recoverable Creation Feedback

| Field | Type | Rules |
|---|---|---|
| `type` | string | Existing normalized conflict category |
| `code` | string | Exactly `recoverable_user_conflict` |
| `status` | integer | Exactly `409` |
| `messageKey` | string | Fixed localized recovery-warning key |
| `recoveryAction` | string | Internal exact action `restore-user` |
| `recoveryUserId` | UUID | Projected only from validated `details.user_id`; never rendered |
| `operationId` | string | Must identify `createUser` for this workflow |
| `requestId` | string/null | Existing safe diagnostic identifier |

The normalized object contains no backend message, raw details object, email,
name, tenant, status, deletion data, role, profile, or audit information. If any
required recovery field is missing or invalid, the normalized result has generic
conflict feedback and no `recoveryUserId` or restore action.

## Recovery Warning Presentation

| Field | Type | Rules |
|---|---|---|
| `visible` | boolean | True only while one current recovery snapshot exists |
| `pending` | boolean | Disables the ordinary restore control during one current restore request |
| `liveMode` | string | Exact polite status/live-region semantics with atomic announcement |
| `messageKey` | string | Fixed localized warning; contains no backend message or identifier |

The warning uses semantic HTML rather than Element Plus `ElAlert`, whose
assertive `role="alert"` would contradict the clarified polite announcement.
Mounting the warning must not call focus, use autofocus, replace the currently
focused form control, or add custom/positive tab order. The restore button
remains an ordinary control in DOM and keyboard order. No UUID may appear in
live-region text, accessible names, descriptions, or component props.

## Recovery Context Snapshot

| Field | Type | Rules |
|---|---|---|
| `userId` | UUID | Validated recovery target; held only in the composable |
| `emailSnapshot` | string | Exact create-form email value when recovery was accepted |
| `schoolId` | UUID | Exact active school that received the response |
| `actorId` | UUID | Current authenticated actor |
| `authorizationGeneration` | string/integer | Changes when session, identity, permissions, or school authority changes |
| `routeName` | string | Must remain `userCreate` until restore success |
| `requestGeneration` | integer | Monotonic stale-response guard |

Any email edit clears the snapshot immediately, even when the edited value later
normalizes to the same address. A school, actor, authorization generation, route,
or newer-request change invalidates the snapshot and all pending results.

The snapshot is never placed in route query, Pinia, local storage, session
storage, analytics, logs, or rendered DOM.

## Restore Confirmation Draft

| Field | Type | Rules |
|---|---|---|
| `effectiveAt` | date | Required; today or earlier |
| `reason` | string | Required; 1–500 characters |
| `pending` | boolean | At most one restore request is active |
| `fieldErrors` | field-message map | Local or published validation feedback |
| `formError` | normalized feedback/null | Safe lifecycle feedback only |

The existing lifecycle composable and dialog own this draft. Local validation,
published `422`, network/no-response, `408`, `429`, and `5xx` outcomes preserve
it. Cancellation, context invalidation, `401`, `403`, `404`, `409`, any other
non-allowlisted HTTP status, or success clears it and invalidates late results.

## Restore Failure Disposition

| Source/status | Preserve draft | Preserve target | Retry mode |
|---|---|---|---|
| Local validation | Yes | Yes | Correct and submit deliberately |
| Published HTTP `422` | Yes | Yes | Correct and submit deliberately |
| Network/no HTTP response (`0`) | Yes | Yes | Deliberate retry only |
| HTTP `408` | Yes | Yes | Deliberate retry only |
| HTTP `429` | Yes | Yes | Deliberate retry only |
| Any HTTP `5xx` | Yes | Yes | Deliberate retry only |
| Published HTTP `401` | No | No | New authenticated create submission required |
| Published HTTP `403`, including tenant denial | No | No | New authorized create submission required |
| Published HTTP `404` | No | No | New create submission required |
| Published HTTP `409` | No | No | New create submission required |
| Any other HTTP status | No | No | Safe default; new create submission required |

Only `200`, `401`, `403`, `404`, `409`, and `422` are published restore
responses. Network/no-response, `408`, `429`, and `5xx` rows define defensive
frontend behavior without promising new endpoint responses or payload shapes.
The policy branches on normalized numeric status rather than broad feedback type
because `unknown` may cover both retryable transport failures and unrelated
non-allowlisted HTTP responses.

## Terminal Recovery Feedback

| Field | Type | Rules |
|---|---|---|
| `type` | enum | unauthorized, forbidden/tenant denial, not found, conflict, or safe unknown |
| `status` | integer | `401`, `403`, `404`, `409`, or another non-allowlisted HTTP status |
| `messageKey` | string | Existing localized safe feedback |
| `requestId` | string/null | Safe request correlation only |
| `operationId` | string | `restoreUser` |

Terminal feedback contains no recovery UUID or submitted reason. It may remain
visible after the recovery action and target reference have been cleared.

## Recovery Workflow State

```text
editing
  -> creating
     -> generic-failure -> editing
     -> recoverable-warning
        -> email/context/route change -> cleared -> editing
        -> confirming-restore
           -> local-validation or 422 -> confirming-restore
           -> restoring
              -> network/0, 408, 429, or 5xx -> confirming-restore
              -> 401, 403, 404, 409, or other HTTP -> cleared + safe-feedback -> editing
              -> restored -> discard-draft + clear-recovery -> user-detail-loading
                 -> detail-ready
                 -> detail-load-failure -> stay userDetail
                    -> retry getUser only or return
```

State invariants:

- Only one current create or restore promise may commit state.
- The warning component never receives the target UUID.
- The warning is a polite atomic status/live region, never moves focus, contains
  no nested assertive alert, and leaves its action in normal keyboard order.
- Restore uses the same school, actor, authorization, route, and email snapshot
  that authorized the recovery warning.
- Success clears the failed creation draft before route navigation.
- The user detail page loads authoritative data with explicit school lookup
  mode; create-form values are not carried forward.
- Once restore succeeds, recovery cannot be reconstructed. Detail-load failure
  remains on `userDetail`; retry invokes only `getUser` and never restore.
- No transition automatically creates, restores, activates, updates, invites,
  or assigns roles beyond the explicit submitted operation.

## Relationships

- One valid **Normalized Recoverable Creation Feedback** initializes at most one
  **Recovery Context Snapshot**.
- One current snapshot may open one **Restore Confirmation Draft**.
- A terminal restore result destroys both snapshot and draft before producing
  **Terminal Recovery Feedback**.
- A successful restore destroys all create/recovery state and navigates to the
  existing user detail workflow, which independently loads the retained user.
- A failed post-restore detail load relates only to the existing detail workflow
  and cannot recreate a snapshot, confirmation draft, warning, or restore call.

## Persistence and Tenancy

- MySQL and soft-delete semantics are unchanged.
- The recovery UUID remains owned by the existing globally unique retained user.
- The frontend cannot broaden school scope, discover a user, or authorize
  restoration; the backend re-evaluates every restore and detail request.
- Platform-user recovery and cross-tenant fallback remain unavailable.

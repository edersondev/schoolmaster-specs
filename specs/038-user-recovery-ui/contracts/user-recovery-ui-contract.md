# User Recovery UI Contract

## Delivery Boundary

Feature 038 changes frontend interpretation and interaction only. It introduces
no endpoint, wire field, response status, permission, database rule, or backend
behavior. Feature 037 remains the authoritative API contract.

| Operation | Method and path | Purpose in this workflow |
|---|---|---|
| `createUser` | `POST /api/v1/users` | Returns `201`, generic validation/conflict feedback, or the exact recoverable conflict |
| `restoreUser` | `POST /api/v1/users/{userId}/restore` | Performs the administrator-confirmed restore in the original school context |
| `getUser` | `GET /api/v1/users/{userId}` | Loads authoritative data after successful restore |

Existing update, activate, and deactivate operations remain separate actions on
the detail/edit journey. Recovery must not call them automatically.

## Canonical Recoverable Create Response

The frontend recognizes recovery only from this canonical nested error body:

```json
{
  "error": {
    "code": "recoverable_user_conflict",
    "message": "A retained user can be restored.",
    "details": {
      "user_id": "f34c45fe-7ee1-4bec-99b1-26cc2cad0456",
      "recommended_action": "restore"
    }
  }
}
```

All conditions below are required:

| Field | Required value |
|---|---|
| HTTP status | `409` |
| `error.code` | Exact `recoverable_user_conflict` |
| `error.details.user_id` | Valid UUID |
| `error.details.recommended_action` | Exact `restore` |

The `error.message` value and unexpected fields are ignored. A flat response
body is not an accepted alternative. Missing, malformed, differently cased, or
unsupported values produce existing generic conflict feedback and no recovery
action.

## Normalized Frontend Projection

The error mapper may project only the fields required by the workflow:

```js
{
  type: 'conflict',
  code: 'recoverable_user_conflict',
  status: 409,
  messageKey: 'administration.users.recovery.warning',
  recoveryAction: 'restore-user',
  recoveryUserId: '<validated-uuid>',
  operationId: 'createUser',
  requestId: '<safe-request-id-or-null>'
}
```

It must not copy the raw response, backend message, details object, submitted
email, school, tenant, user data, authentication data, or audit data into the
normalized object. Generic feedback must not contain `recoveryAction` or
`recoveryUserId`.

## Warning Component Contract

`UserRecoveryAlert.vue` is presentational.

- Displays localized frontend text: `An existing user can be restored.`
- Displays one action: `Restore existing user`.
- Receives only visibility and pending/disabled presentation state.
- Emits a `restore` event after deliberate activation.
- Uses a semantic `role="status"` or equivalent polite live region with
  `aria-live="polite"` and atomic announcement.
- Does not use `ElAlert` or contain a nested `role="alert"`, because that would
  introduce assertive semantics.
- Does not focus itself, autofocus the action, replace the active element, or
  use a custom/positive tab index when it appears. The action remains a normal
  button in DOM and keyboard order.
- Does not receive or render a UUID, email, raw error, backend message, school,
  lifecycle state, deletion detail, role, profile, or audit information.
- Disables or deduplicates activation while a restore request is pending.

## Restore Confirmation and Request

The action opens the existing `AdminLifecycleDialog.vue` through the existing
lifecycle composable. The dialog uses a fixed localized generic resource label
and must not disclose the recovery identifier.

The restore request remains:

```http
POST /api/v1/users/{validatedUserId}/restore
X-School-Id: {originalSchoolId}
Content-Type: application/json
```

```json
{
  "effective_at": "YYYY-MM-DD",
  "reason": "Administrator-provided reason"
}
```

`effective_at` is required and cannot be in the future. `reason` is required
and is limited to 500 characters. Only one current restore submission may be
active. The backend independently reauthorizes the operation.

## Context Validity Contract

Recovery remains executable only while all snapshot values are unchanged:

| Context value | Invalidation behavior |
|---|---|
| Submitted email | Any edit immediately clears recovery, even if changed back |
| Active school | Change or loss clears recovery and invalidates pending results |
| Actor/session | Sign-out, actor change, or auth generation change clears recovery |
| Permission context | Reset or loss clears recovery |
| Route | Leaving `userCreate` clears recovery |
| Request generation | A newer create result invalidates older recovery/results |
| Dialog/action | Cancellation or terminal failure clears target and pending result |

Create and restore promises use request generations or equivalent cancellation
guards. A stale resolution must not alter feedback, show success, make a
follow-up request, or navigate.

## Failure Contract

| Outcome | Dialog values | Recovery target/action | Feedback |
|---|---|---|---|
| Local validation | Preserve | Preserve | Existing field feedback; deliberate correction |
| Published HTTP `422` | Preserve | Preserve | Existing safe field/form feedback; deliberate correction |
| Network/no HTTP response (`0`) | Preserve | Preserve | Existing safe retryable feedback; deliberate retry |
| HTTP `408` | Preserve | Preserve | Existing safe retryable feedback; deliberate retry |
| HTTP `429` | Preserve | Preserve | Existing safe retryable feedback; deliberate retry |
| Any HTTP `5xx` | Preserve | Preserve | Existing safe retryable feedback; deliberate retry |
| Published HTTP `401` | Clear | Clear and invalidate | Existing safe session feedback |
| Published HTTP `403`, including tenant denial | Clear | Clear and invalidate | Existing safe forbidden/scope feedback |
| Published HTTP `404` | Clear | Clear and invalidate | Existing safe non-disclosing not-found feedback |
| Published HTTP `409` | Clear | Clear and invalidate | Existing safe conflict feedback |
| Any other HTTP status | Clear | Clear and invalidate | Existing safe normalized fallback |

The restore endpoint publishes only `200`, `401`, `403`, `404`, `409`, and
`422`. Network/no-response, `408`, `429`, and `5xx` entries specify defensive
frontend disposition without adding or guaranteeing endpoint responses. The
client assumes no payload shape for those transport/infrastructure outcomes.

No failure automatically retries restore or creates another identity.

## Success Contract

After a current-context `200` restore result:

1. Mark restoration successful once.
2. Reset the create form so its draft cannot trigger unsaved-change handling or
   become retained-user data.
3. Clear the recovery UUID, snapshot, warning, dialog, and pending state.
4. Navigate by named route to `userDetail` with the restored UUID in the path
   and explicit query `user_mode=school`.
5. Let `UserDetailPage.vue` call the existing detail service and render the
   authoritative retained user.

If that detail request fails, restoration remains complete. The route stays on
`userDetail`, and the existing detail page presents its safe retry or return
controls. Retry invokes `getUser` only. The recovery warning, UUID, confirmation
draft, and creation draft remain cleared; restore is never repeated and recovery
cannot reopen from the detail failure.

The failed creation draft is never passed in route state, query parameters, or
detail/edit props. Profile, role, status, activation, and invitation changes
remain separate explicit workflows.

## Privacy and Observability

The recovery UUID may exist only in the route-local composable and the required
restore/detail path after success. Before success it must not appear in rendered
DOM, route/query state, Pinia, local/session storage, analytics, telemetry,
logs, notification copy, or user-visible diagnostics. The submitted email and
lifecycle reason have the same no-logging/no-telemetry boundary.

## Verification Contract

- Mapper tests prove exact allowlisting and generic fallback for every malformed
  or unsupported variation.
- Service tests prove the existing restore path, method, body, and school header.
- Composable tests prove single-flight and stale create/restore invalidation.
- Component/page tests prove safe copy, no identifier rendering, explicit
  dialog opening, the exact status disposition matrix, draft discard, and
  school-mode navigation.
- Alert tests prove polite/atomic status semantics, no nested assertive alert,
  unchanged active element when mounted, normal keyboard order, and no UUID in
  accessible text or names.
- Page/workflow tests prove that a failed detail GET after one successful restore
  stays on the detail route; retry increments only the detail request count and
  restore remains called exactly once.
- Playwright proves the complete conflict-to-restore-to-detail workflow plus
  privacy, repeated-submit, context-change, exact failure classes, responsive,
  live-announcement, focus-preservation, and keyboard paths.
- Redocly lint confirms the consumed OpenAPI remains valid.

# Research: User Recovery UI

## Decision 1: Consume the published Feature 037 contract without changing it

**Decision**: Treat Feature 038 as a frontend-only consumer of the existing
`createUser`, `restoreUser`, and `getUser` operations. The canonical recoverable
response remains the nested error envelope published by Feature 037. A flat HTTP
body is a contract defect, not an alternate frontend contract.

**Rationale**: Backend implementation and tests already assert the exact
`409 recoverable_user_conflict` envelope and the existing restore authorization.
Adding another accepted wire shape would weaken contract governance and hide a
server regression.

**Alternatives considered**: Adding a new recovery endpoint, changing the error
envelope, or supporting both nested and flat wire bodies were rejected because
the existing operations are sufficient and already published.

## Decision 2: Classify recovery through an exact allowlist

**Decision**: Recognize recovery only when status is `409`, code is exactly
`recoverable_user_conflict`, `details.user_id` is a valid UUID, and
`details.recommended_action` is exactly `restore`. Project only the validated
UUID and supported action into normalized internal feedback. Ignore backend
message text and every additional response field. Any malformed or unsupported
variant remains a generic conflict with no recovery action.

**Rationale**: The response carries a disclosure-authorized target reference.
Strict classification prevents arbitrary conflict details, malformed IDs, or
future unsupported actions from becoming executable client state.

**Alternatives considered**: Status-only classification, code-only
classification, and passing the raw `details` object through the UI were
rejected because each can expose or execute unapproved data.

## Decision 3: Use frontend-owned localized recovery copy

**Decision**: Map the exact recovery result to localized frontend copy:
`An existing user can be restored.` and `Restore existing user`. Never render,
combine, log, or fall back to the backend `message` field.

**Rationale**: Fixed client copy is safe, testable, accessible to localization,
and consistent with the existing administration feedback architecture.

**Alternatives considered**: Rendering the backend message or combining it with
client guidance were rejected because server copy is not a presentation
contract and could disclose unexpected content.

## Decision 4: Keep recovery in a route-local composable

**Decision**: Add `useUserCreationRecovery.js` as the sole owner of the
validated recovery UUID, email and authorization-context snapshot, lifecycle
dialog orchestration, failure classification, and success navigation intent.
Expose readonly state and explicit actions. Add no Pinia store, URL parameter,
browser storage, analytics property, or persistent cache.

**Rationale**: Recovery is short-lived state for one create page. Keeping it
local minimizes stale target risk and follows the existing feature-composable
architecture.

**Alternatives considered**: Storing recovery in Pinia, route query, local
storage, or the page component directly was rejected because those approaches
either persist sensitive state or overload the route page with business rules.

## Decision 5: Use a focused alert and reuse lifecycle infrastructure

**Decision**: Add a presentational `UserRecoveryAlert.vue` that receives no user
identifier and emits only `restore`. Compose the existing `restoreUser` service,
`useAdminLifecycleAction`, and `AdminLifecycleDialog` with a fixed localized
resource label. Do not fetch the retained user before restoration. Present the
warning as a polite status/live region, leave the active element unchanged when
it appears, and keep the restore button in ordinary DOM and keyboard order.
Use semantic HTML with Tailwind and a normal `ElButton`; do not use `ElAlert`
because the installed Element Plus component supplies assertive `role="alert"`.

**Rationale**: The existing lifecycle stack already owns reason/date validation,
single-flight submission, safe feedback, and the published restore request. A
focused alert keeps props-down/events-up boundaries explicit without duplicating
the generic form or dialog.

**Alternatives considered**: A new restore modal, direct service calls from the
component, a pre-restore detail lookup, an assertive alert, or automatic focus
movement were rejected because they duplicate approved behavior, expand
disclosure, or disrupt the administrator's current form interaction.

## Decision 6: Invalidate both create and restore promises by context generation

**Decision**: Strengthen create-form and lifecycle composables with explicit
request invalidation. Reset, email change, school/session/permission context
change, route departure, cancellation, or a newer request advances a generation
and prevents late resolution, feedback, success messages, follow-up requests,
or navigation. Create and restore remain single-flight.

**Rationale**: Current create reset can accept a late promise result, and current
lifecycle close does not invalidate an in-flight request. Feature 038 requires
stale results to have no effect across school or session boundaries.

**Alternatives considered**: Clearing visible state only after a response and
relying solely on component unmount were rejected because promise callbacks can
still mutate state or start navigation after context changes.

## Decision 7: Navigate to authoritative school-mode detail after success

**Decision**: After restore succeeds under the unchanged context snapshot,
reset the create form to discard the draft and satisfy the unsaved-changes
guard, clear recovery, and navigate to `userDetail` with the restored UUID and
explicit `user_mode=school`. The detail page performs the authoritative
tenant-scoped load; profile and roles remain in its edit workflow and status
remains a lifecycle action.

**Rationale**: The restore outcome is not a full user record. An authoritative
detail load avoids treating failed creation values as retained-user data and
prevents master sessions from falling back to platform lookup mode.

Restoration is complete before routing. If the subsequent detail request fails,
remain on `userDetail` and use its existing safe retry or return controls. A
retry calls only `getUser`; it never repeats restore, reconstructs recovery,
restores the creation draft, or reopens the dialog.

**Alternatives considered**: Carrying draft values into edit, navigating
directly to edit, returning to the list, or putting the UUID in the create URL
were rejected by the clarified flow and privacy boundary.

## Decision 8: Separate retryable and terminal restore failures

**Decision**: Keep the dialog, entered reason/date, and current target for local
validation, published `422` validation, network/no-response failures, `408`,
`429`, and `5xx` outcomes. Permit only a deliberate retry. For published `401`,
`403` (including tenant denial), `404`, or `409` outcomes, close and invalidate
the dialog, clear the target reference, show only normalized safe feedback, and
require a new create submission before recovery can return. Clear recovery for
every other non-allowlisted HTTP status as the safe default.

The published restore contract declares `200`, `401`, `403`, `404`, `409`, and
`422`. Network failures and `408`, `429`, or `5xx` are generic client
transport/infrastructure resilience classes, not newly promised restore
responses. Their bodies and codes must not be assumed, and this policy does not
change OpenAPI.

**Rationale**: Correctable input and transient transport failures benefit from
preserved state. Authorization, scope, missing-target, and lifecycle conflicts
make the disclosed target stale or ineligible and must not remain executable.

**Alternatives considered**: Keeping recovery for every failure risks repeated
use of a stale target; clearing every failure needlessly discards correctable
input; treating transport resilience classes as new endpoint promises would
misstate the published contract.

## Decision 9: Use layered frontend verification and contract evidence

**Decision**: Use Vitest and Vue Test Utils for error classification, services,
create/lifecycle invalidation, the recovery composable, alert, and mounted create
page. Add a stateful mocked Playwright workflow for create conflict through
restore and authoritative detail navigation, privacy fallbacks, single-flight,
context changes, the exact failure disposition matrix, a failed detail request
with detail-only retry, responsive layouts, polite announcement without focus
movement, natural keyboard order, and dialog keyboard behavior. Verify the
unchanged published OpenAPI with Redocly and run the production build.

**Rationale**: Service/composable/component tests isolate rules; mounted and
browser tests prove composition and user flow. Mocked browser tests do not
replace existing backend authorization and contract tests from Feature 037.

**Alternatives considered**: Page tests alone, browser tests alone, live-backend
E2E as the default gate, and a TypeScript migration were rejected because they
either leave rule gaps, misstate evidence, or add unrelated scope.

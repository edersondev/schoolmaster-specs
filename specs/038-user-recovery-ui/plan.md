# Implementation Plan: User Recovery UI

**Branch**: `038-user-recovery-ui` | **Date**: 2026-08-23 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/038-user-recovery-ui/spec.md`

## Summary

Add frontend handling for the exact published `recoverable_user_conflict`
response during school-user creation. The Vue application will strictly
classify the response, display fixed localized guidance through a polite status
region without moving focus, keep the validated user reference in transient
route-local state, reuse the existing lifecycle confirmation and restore
service, and invalidate stale create and restore results.

The restore dialog preserves its draft for local/422 validation and defensive
network/408/429/5xx failures, while 401/403/404/409 and other non-allowlisted
HTTP results clear recovery. After success, the app discards the creation draft,
clears recovery, and navigates to authoritative school-mode user detail. A
failed detail request stays in the existing detail workflow and can retry only
`getUser`; restore is never repeated.

This is a specification and frontend delivery. The Feature 037 OpenAPI and
Laravel behavior remain unchanged prerequisites.

## Technical Context

**Language/Version**: JavaScript on Node.js `^22.18.0 || >=24.12.0`; Vue 3.5.38 with Composition API and `<script setup>`  
**Primary Dependencies**: Vue Router 5.1.0, Pinia 3.0.4, Axios 1.18.0, Element Plus 2.14.2, Tailwind CSS 4.3.1, Vue I18n 11.4.6, Vite 8.0.16  
**Storage**: Transient in-memory component/composable state only; no Pinia persistence, URL state, browser storage, MySQL, or backend schema change  
**Testing**: Vitest 4.1.9, Vue Test Utils 2.4.11, Playwright 1.61.0, Redocly OpenAPI lint, Vite production build  
**Target Platform**: SchoolMaster Vue 3 SPA in supported modern desktop and mobile browsers  
**Project Type**: Frontend web application backed by an existing Laravel REST API  
**Performance Goals**: One active create and one active restore at most; stale results have zero visible/navigation effect; no discovery request; recovery completes within the two deliberate actions defined by SC-003  
**Constraints**: Exact recovery allowlist; fixed localized copy; no raw message/identifier rendering; polite atomic status with unchanged focus and normal tab order; preserve local/422/network/408/429/5xx; clear 401/403/404/409/other HTTP; deliberate retries only; no repeated restore after success; no endpoint, package, store, or backend change  
**Scale/Scope**: One school-user create page, one new alert component, one new recovery coordinator, three existing composable/mapper seams, existing restore/detail workflows, and focused unit/component/workflow coverage

The frontend repository is explicitly JavaScript. This plan keeps its established
language rather than creating an isolated TypeScript migration.

## Constitution Check

*GATE: Design passed before Phase 0 research and was re-checked after Phase 1
design. Contract execution evidence remains a blocking T001 delivery gate.*

- **API-first governance — PASS**: Feature 038 consumes the published Feature
  037 create, restore, and detail operations. Restore publishes
  `200/401/403/404/409/422`; network/408/429/5xx handling is defensive client
  resilience with no payload assumption, not a new endpoint guarantee. OpenAPI
  is unchanged and must be verified with Redocly in T001 before frontend work.
- **Repository boundaries — PASS**: `schoolmaster-specs` leads and must approve
  the Feature 038 artifacts before `schoolmaster-frontend` begins code and tests.
  The frontend branch or pull request uses Feature 038 and links the approved
  specification change. `schoolmaster-backend` requires no delivery unless
  contract verification finds an existing defect.
- **Laravel boundaries — N/A/PASS**: No Laravel code changes. Existing Services,
  Form Requests, Policies, Resources, UUID boundaries, and Feature 037 tests
  remain authoritative.
- **Vue architecture — PASS**: Vue 3 Composition API, a route-local composable,
  focused presentational component, Vue Router, localized copy, and existing
  Axios services are used. Components make no direct HTTP calls, and no global
  store is added for short-lived recovery state.
- **Tenant and soft-delete safety — PASS**: The recovery target is bound to the
  exact school, actor, authorization generation, route, email, and request
  context. The frontend performs no lookup or cross-tenant inference. Existing
  email uniqueness and soft-delete semantics remain unchanged.
- **Compatibility and authorization — PASS**: Recovery is additive for one exact
  `409` shape. All malformed/other duplicates stay generic, and every restore
  and detail request remains independently authorized by the backend.
- **Verification — PASS**: Redocly, mapper/service/composable/component/page
  Vitest, Playwright, privacy/accessibility audits, and a production build cover
  the changed flow. No backend behavior changes, so no new PHPUnit implementation
  is required.

## Project Structure

### Documentation (this feature)

```text
schoolmaster-specs/specs/038-user-recovery-ui/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── user-recovery-ui-contract.md
└── tasks.md
```

### Source Code (target repositories)

```text
schoolmaster-frontend/
├── src/
│   ├── components/admin-system/users/
│   │   └── UserRecoveryAlert.vue                 # New polite status region
│   ├── components/ui/admin/
│   │   ├── AdminLifecycleDialog.vue              # Existing, reused
│   │   └── AdminDetailPage.vue                   # Existing retry/return surface
│   ├── composables/admin-system/
│   │   ├── useAdminCreateForm.js                 # Invalidate stale creates
│   │   ├── useAdministrationCreatePage.js        # Context/page reset integration
│   │   ├── useAdminLifecycleAction.js            # Invalidate stale restores
│   │   ├── useAdminDetail.js                     # Existing detail-only retry
│   │   └── useUserCreationRecovery.js            # New route-local coordinator
│   ├── contracts/admin-system/administration.js  # Internal recovery action
│   ├── locales/administration.js                 # Fixed localized copy
│   ├── pages/admin-system/users/
│   │   ├── CreateUserPage.vue                    # Thin composition surface
│   │   └── UserDetailPage.vue                    # Existing destination
│   ├── router/                                   # Existing named routes reused
│   └── services/admin-system/
│       ├── administration-error-mapper.js        # Exact safe projection
│       └── users.js                              # Existing create/restore/detail
├── tests/unit/
│   ├── admin-system/administration/
│   ├── admin-system/administration-lifecycle/
│   └── admin-system/user-recovery/               # New focused tests
└── e2e/
    └── user-recovery.spec.js                     # New complete workflow

schoolmaster-backend/                             # Verified prerequisite only
└── tests/                                        # Existing Feature 037 evidence
```

**Structure Decision**: Keep the create route as an orchestration surface. The
new recovery composable owns transient workflow rules; the new alert owns only
safe presentation and emits one action; existing services own HTTP; the
existing lifecycle dialog owns reason/date confirmation; the existing detail
page owns authoritative loading and retry/return. No TypeScript island, Pinia
store, duplicated modal, endpoint, or backend class is introduced.

## Recovery State and Component Map

| Unit | Single responsibility | Inputs | Outputs/calls | Prohibited behavior |
|---|---|---|---|---|
| `CreateUserPage.vue` | Compose create and recovery flows | Normalized create feedback and active route/session context | Opens recovery; resets draft; routes after current success | Raw HTTP classification, UUID rendering, direct Axios, detail prefetch |
| `UserRecoveryAlert.vue` | Present safe recovery guidance | Visibility and pending/disabled state only | Emits `restore` | UUID/email/backend message props, `ElAlert`, assertive alert, autofocus/custom tab order |
| `useUserCreationRecovery.js` | Own target/context, disposition matrix, dialog orchestration, success intent | Exact normalized feedback and current context | Calls lifecycle action; returns readonly state/actions | Persistence, discovery, presentation, broad type-only status decisions |
| `useAdminCreateForm.js` | Own create draft/validation/single-flight generation | Values and create service | Commits only current create result | Accepting late results after reset/context change |
| `useAdministrationCreatePage.js` | Integrate school/session/page lifecycle | Active administration context | Invalidates create/recovery on change | Preserving recovery across context or route change |
| `useAdminLifecycleAction.js` | Own reason/date validation and request generation | Lifecycle action configuration | Calls existing restore service | Accepting a late restore after close/reset/invalidation |
| `administration-error-mapper.js` | Normalize safe feedback and numeric status | Canonical nested envelope/transport error | Exact recovery projection or generic feedback | Raw message/details projection or malformed recovery action |
| `AdminLifecycleDialog.vue` | Existing accessible confirmation UI | Generic label and lifecycle state | Confirm/cancel | Recovery UUID or duplicate-account metadata |
| `UserDetailPage.vue` + `useAdminDetail.js` | Load authoritative restored user and expose retry/return | Restored path UUID and `user_mode=school` | `getUser` only on load/retry | Repeating restore or rebuilding create/recovery state |

```text
CreateUserPage
  ├─ useAdministrationCreatePage
  │    └─ useAdminCreateForm ── users.createUser
  ├─ UserRecoveryAlert (polite status, emits restore)
  ├─ useUserCreationRecovery
  │    └─ useAdminLifecycleAction ── users.restoreUser
  └─ AdminLifecycleDialog
         success ── reset/clear ── router → UserDetailPage (`user_mode=school`)
                                              └─ useAdminDetail ── users.getUser
                                                   failure → stay/retry GET or return
```

The alert follows props-down/events-up. The composable exposes readonly state
plus explicit `accept`, `open`, `submit`, `cancel`, `invalidate`, and success
intent actions as needed by existing conventions. Source state stays minimal;
derived visibility/pending state uses computed values, and watchers are reserved
for context invalidation side effects.

## Implementation Approach

### Phase 0 — Contract and source verification

1. Verify the canonical nested create-conflict envelope and published
   `createUser`, `restoreUser`, and `getUser` operations with Redocly and Feature
   037 evidence.
2. Preserve existing service endpoints and `X-School-Id`. Treat a flat error
   body as a backend defect rather than a second frontend shape.
3. Record that restore publishes only `200/401/403/404/409/422`.
   Network/no-response, `408`, `429`, and `5xx` handling is defensive client
   policy without response-body assumptions or OpenAPI changes.

### Phase 1 — Safe classification and stale-request hardening

1. Add the internal restore action and extend the error mapper to require status
   `409`, exact code, valid UUID, and exact `restore` recommendation. Project
   only the identifier/action; never project backend message or raw details.
2. Add explicit generation invalidation to create/lifecycle composables so
   reset, cancel, email/context/route changes, or newer requests make late
   resolutions inert.
3. In the recovery coordinator, preserve lifecycle state only for local
   validation, status `0`/network, `408`, `422`, `429`, and `>=500`. Clear the
   dialog/target for `401/403/404/409` and every other non-allowlisted HTTP
   status, while retaining only safe normalized terminal feedback.
4. Branch on numeric normalized status, not broad `unknown`/`unavailable` type,
   because `unknown` can represent both retryable transport failures and other
   HTTP results.

### Phase 2 — Recovery interaction and authoritative continuation

1. Add route-local `useUserCreationRecovery.js` with readonly state, strict
   context snapshot, explicit actions, exact failure disposition, and success
   navigation intent.
2. Add `UserRecoveryAlert.vue` using semantic HTML, Tailwind, and ordinary
   `ElButton`: polite atomic status region, no nested `role="alert"`, no focus
   mutation, no custom tab order, and no sensitive prop or accessible text.
3. Compose alert and lifecycle dialog in `CreateUserPage.vue`. Suppress the
   generic assertive conflict surface while exact recovery is active so the
   guidance is not duplicated or announced assertively.
4. Clear recovery for email, school, session, permission, route, cancellation,
   or newer-result changes. Repeated submissions remain single-flight.
5. On current restore success, reset the failed create draft, clear recovery,
   and route to named `userDetail` with `user_mode=school`. Do not prefetch or
   gate navigation on detail data.
6. If `getUser` fails, remain on the detail route and reuse existing safe
   retry/return behavior. Retry only the GET. Recovery cannot reopen and restore
   call count remains one.

### Phase 3 — Verification and delivery evidence

1. Extend mapper/service/create/lifecycle tests for exact classification,
   privacy, single-flight, and invalidation.
2. Add recovery composable tests for local/0/408/422/429/5xx preservation,
   401/403/404/409/other-HTTP clearing, safe feedback retention, and stale-result
   suppression.
3. Add alert tests for polite atomic semantics, no nested assertive alert,
   unchanged active element, natural keyboard order, and no UUID in DOM or
   accessible naming.
4. Add mounted page and Playwright journeys for conflict-to-restore-to-detail,
   generic/malformed privacy fallbacks, exact failure matrix, context changes,
   responsive layout, and keyboard behavior.
5. Simulate one successful restore followed by failed detail GET; assert route
   remains detail, normal retry/return is available, retry increments GET only,
   and restore remains called exactly once.
6. Run focused/full Vitest, Playwright, privacy/accessibility source audits,
   Redocly lint, and the production build.
7. Run moderated acceptance with a preselected cohort of at least 10 authorized
   administrators. Record whether at least 90% independently identify that the
   existing identity must be restored rather than recreated and select the
   restore action as the correct next step.

## Post-Design Constitution Check

Phase 1 design preserves every pre-research gate:

- Published API operations and response shapes remain unchanged; defensive
  transport handling does not invent a server contract.
- Specification and frontend repositories remain independently deliverable;
  backend is an already merged prerequisite.
- Vue business rules live in composables/normalization, UI components remain
  presentational, and API calls remain service-isolated.
- Tenant scope is explicit in context and `X-School-Id`; stale/cross-context
  references cannot remain executable.
- No persistent/global state contains UUID, email, lifecycle reason, or hidden
  account data; the polite live region exposes fixed copy only.
- Vitest, Playwright, Redocly, build, privacy, focus, keyboard, and moderated
  acceptance gates cover the clarified critical flow.

**Result**: PASS. No constitution exception is required.

## Complexity Tracking

No constitution violations or exceptional complexity are required.

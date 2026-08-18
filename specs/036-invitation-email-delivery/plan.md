# Implementation Plan: Invitation Email Delivery

**Branch**: `036-invitation-email-delivery` | **Date**: 2026-08-15 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/036-invitation-email-delivery/spec.md`

## Summary

Complete account onboarding through contract-first invitation email delivery.
Laravel will synchronously submit a transactional mailable containing the
newly issued plaintext token only in the setup URL, retain only its hash, and
record delivery-request metadata after mail transport acceptance. A transport
failure leaves a secret-safe, replaceable invitation and returns documented
`503 temporary_unavailable`. Vue keeps existing explicit create-then-invite
flow, removes the account-setup selector, and always maps create-user requests
to `invitation`; API omission remains `active` for compatibility.

## Technical Context

**Language/Version**: PHP 8.3 with Laravel 13.8; JavaScript with Vue 3.5 Composition API and `<script setup>`
**Primary Dependencies**: Laravel Mail/Mailable and Blade Markdown mail views; existing DTO, Policy, Resource, repository, and account lifecycle services; Vue Router 5.1, Pinia 3.0, Axios 1.18, Element Plus 2.14, Vue I18n 11.4, Tailwind CSS 4.3
**Storage**: Existing MySQL `account_invitations`, users, roles, and audit events; no migration or durable plaintext token payload
**Testing**: Redocly OpenAPI lint/bundle; PHPUnit 12 feature and mailable tests with `Mail::fake()` plus transport-failure doubles; Vitest 4.1 contract tests; Playwright 1.61 mocked end-to-end flow; frontend production build
**Target Platform**: Laravel JSON API on Linux, configured SMTP-compatible production mail transport, and modern browsers consuming `/api/v1`
**Project Type**: Multi-repository web application: shared specs/OpenAPI, Laravel API, Vue SPA
**Performance Goals**: One mail submission per accepted invitation request; invitation creation stays within configured mail-transport timeout; no background payload carries the plaintext token
**Constraints**: Contract before backend before frontend; no new package or endpoint; no token in API/log/audit/database/queue metadata; trusted frontend origin only; delivery acceptance is not inbox-delivery proof; existing limits, tenancy, authorization, and setup rules remain
**Scale/Scope**: One existing invitation endpoint, one new mailable/view, one trusted URL setting, one standard 503 response, one frontend invitation-only invariant, focused tests, and synchronized Feature 008/034 docs

## Constitution Check

*GATE: PASS before Phase 0; re-checked and PASS after Phase 1 design.*

- PASS: OpenAPI documents invitation email side effects and `503` before backend changes; no endpoint or API version is added.
- PASS: Specification, backend, and frontend impacts are separate and sequenced under Feature 036; Features 008 and 034 are synchronized.
- PASS: Existing thin controller, Form Request, DTO, Policy, Resource, and repository boundaries remain. `AccountInvitationService` owns orchestration and a focused delivery service owns mail submission. UUID boundaries remain.
- PASS: Vue remains JavaScript by repository convention, Composition API/`<script setup>`, existing Pinia session state, router, Tailwind/Element Plus, and Axios services. Default change stays in pure form-contract state; no component HTTP or new store is added.
- PASS: MySQL storage and tenant-first account lifecycle rules remain unchanged. No cross-tenant or soft-delete behavior changes.
- PASS: Compatibility, authentication, authorization, 201 success, 503 temporary failure, and secret-safe response behavior are explicit.
- PASS: Redocly, PHPUnit, Vitest, Playwright, mail-content assertions, and production build cover changed critical flow.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/036-invitation-email-delivery/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── invitation-email-delivery.md
└── tasks.md
```

### Source Code (target repositories)

```text
schoolmaster-specs/
├── api/
│   ├── components/responses/common/TemporaryUnavailable.yaml
│   └── paths/account-lifecycle/invitations.yaml
├── specs/001-schoolmaster-platform/contracts/openapi.yaml
├── specs/008-account-lifecycle-workflows/
├── specs/034-complete-account-lifecycle-ui/
└── specs/036-invitation-email-delivery/

schoolmaster-backend/
├── app/
│   ├── Exceptions/InvitationDeliveryException.php
│   ├── Mail/AccountInvitationMail.php
│   └── Services/AccountLifecycle/
│       ├── AccountInvitationDeliveryService.php
│       └── AccountInvitationService.php
├── bootstrap/app.php
├── config/app.php
├── resources/views/mail/account-invitation.blade.php
├── .env.example
└── tests/
    ├── Feature/AccountLifecycle/AccountInvitationCreationTest.php
    └── Unit/Mail/AccountInvitationMailTest.php

schoolmaster-frontend/
├── src/components/admin-system/users/UserForm.vue
├── src/pages/admin-system/users/CreateUserPage.vue
├── src/contracts/admin-system/users.js
├── tests/unit/admin-system/administration/
│   ├── contracts/access.contract.spec.js
│   └── services/users.spec.js
└── e2e/account-lifecycle.spec.js
```

**Structure Decision**: Extend existing account lifecycle boundaries. Backend
mail formatting is isolated in one mailable/view and delivery orchestration in
one service; `AccountInvitationService` retains lifecycle ownership. Frontend
change removes one field from the existing form and enforces the invariant in
the pure request mapper, so no component or composable split is needed.
Existing `CreateUserPage.vue` remains the composition surface.

## Component Map

| Surface | Responsibility | Inputs / outputs |
|---|---|---|
| `CreateUserPage.vue` | Compose identity/role form and invitation phase without setup-mode controls | Existing session/route/form in; create and navigation actions out |
| `UserForm.vue` | Render editable user identity and role fields only | Form model and errors in; `v-model` update out |
| `mapUserCreateRequest()` | Enforce frontend invitation-only creation | User identity/roles in; explicit invitation payload out |

## Implementation Approach

### Phase 0: Contract and source-of-truth alignment

- Add reusable `503 temporary_unavailable` response and attach it to
  `createAccountInvitation` in modular and aggregate OpenAPI.
- State that `201` means mail transport accepted submission, not inbox delivery;
  keep token and setup URL absent from response.
- Update Feature 008 provider-specific-delivery exclusions and Feature 034
  active-form-default wording without broadening password reset or admin resend.

### Phase 1: Backend mail delivery

- Add trusted `APP_FRONTEND_URL` configuration and validate a non-empty
  HTTP(S) origin before constructing
  `/auth/account-invitations/{token}/setup`.
- Add `AccountInvitationMail` and escaped Markdown mail view with product name,
  recipient greeting, expiry, setup action, and ignore-if-unexpected guidance.
- Extend invitation creation to retain plaintext token only in memory, commit
  hashed invitation state first, synchronously submit mail, then update
  `delivery_requested_at`, `delivery_channel`, safe metadata, and success audit.
- On configuration or transport failure, record secret-free failure audit and
  throw `InvitationDeliveryException`; global exception rendering returns
  standard `temporary_unavailable` 503. Pending undelivered invitation remains
  replaceable; retry supersedes it and any late/ambiguous earlier message.
- Keep synchronous sending because queue serialization would durably store the
  reusable plaintext token. No new queue/job/package is introduced.

### Phase 2: Frontend invitation-only creation

- Remove account-setup mode from form state and `UserForm.vue`.
- Make `mapUserCreateRequest()` always submit
  `account_setup_mode=invitation`, regardless of stale extra form fields.
- Preserve persisted create-then-invite phase, page reload recovery, permission
  gates, no auto-send, and no auto-sign-in.
- Update Vitest and Playwright expectations to prove default invitation mode
  and full create/invite/setup/login journey.

### Phase 3: Verification and rollout

- Validate modular and aggregate OpenAPI with Redocly.
- Run focused and full PHPUnit; inspect rendered mailable and assert one target,
  exact trusted URL, expiry, transport failure, retry, token secrecy, and login.
- Run focused/full Vitest, Playwright account lifecycle flow, and production
  build.
- Deployment must set trusted frontend origin and working mail transport before
  enabling invitation operations. Existing active API clients need no changes.

## Complexity Tracking

No constitution violations.

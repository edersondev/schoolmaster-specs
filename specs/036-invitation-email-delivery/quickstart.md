# Quickstart: Invitation Email Delivery Verification

## Preconditions

- Feature 036 contracts are applied before backend/frontend code.
- Backend has trusted `APP_FRONTEND_URL` and working mail transport settings.
- Eligible invited user and authorized same-scope administrator exist.

## Contract verification

- Lint aggregate `api/openapi.yaml` and bundled platform contract.
- Confirm invitation creation documents `201`, `401`, `403`, `409`, `422`, and
  `503`, with no token/setup URL in `AccountInvitation`.
- Confirm user-create API default remains `active`.

## Backend verification

1. Fake mail and create invitation.
2. Assert exactly one `AccountInvitationMail` to target current email.
3. Assert rendered mail contains trusted frontend origin, secret-free setup
   path, encoded issued token only in the fragment, product identity, and expiry.
4. Assert stored invitation contains hash only and response/log/audit/metadata
   contain neither token nor setup URL.
5. Assert `delivery_requested_at` is populated only after successful send.
6. Force transport failure; assert safe `503 temporary_unavailable`, candidate
   removal, any prior delivered invitation remains valid, the transport
   exception is chained, failed submissions do not consume quota, and the
   failure audit is secret-free.
7. Retry; assert old invitation superseded only after acceptance, replacement email sent, replacement
   setup completes, old link fails, and login succeeds only after setup.
8. Run focused then full PHPUnit and Pint.

## Frontend verification

1. Open fresh create-user form; verify no Account setup field or Active
   immediately option is rendered.
2. Submit identity and role details; verify request contains
   `account_setup_mode=invitation`.
3. Complete existing create-then-invite UI and validate safe success/failure
   feedback.
4. Open emailed link; verify the token is read from the fragment, immediately
   removed from browser history, and sent only in the JSON setup body. Set the
   password, then use normal login.
5. Run focused/full Vitest, Playwright account lifecycle test, and build.

## Release checks

- Production deploy has non-local trusted frontend origin and real mailer.
- Mail sender identity is configured and verified by selected transport.
- Support runbook treats 201 as accepted submission, not confirmed inbox
  delivery; provider bounce/complaint handling remains separate work.

## Implementation Evidence — 2026-08-15

- Aggregate and platform OpenAPI bundles regenerated; Redocly lint passed for
  `aggregate@v1` and `schoolmaster-platform@v1`.
- Backend Pint passed.
- Focused backend invitation suite passed: 9 tests, 48 assertions.
- Full backend PHPUnit suite passed after final secret-handling hardening.
- Focused frontend Vitest passed: 13 tests across 3 files.
- Full frontend Vitest suite passed.
- Production frontend build passed; existing Rolldown pure-annotation and large
  chunk warnings remain non-blocking and unrelated.
- Playwright account lifecycle suite passed 12 cases across Chromium, Firefox,
  and WebKit, including fresh Invitation default and emailed setup-link login.
- Security review confirmed mailable does not implement `ShouldQueue`; plaintext
  token is used only in request memory and email setup URL. Database stores hash
  only; response, delivery metadata, audit event, and failure envelope remain
  secret-free.

## Review Hardening Evidence — 2026-08-18

- Modular and aggregate OpenAPI validate; aggregate retains 9 unrelated
  pre-existing unused assessment-schema warnings.
- `createAccountInvitation` 201 specializes `delivery_requested_at` as required
  and non-null in both publications.
- Invitation setup uses a URL fragment, removes it from browser history, and
  submits `invitation_token` only in the JSON body at a secret-free API path.
- Backend focused invitation coverage passed: 21 tests, 122 assertions. Full
  backend PHPUnit passed: 523 tests, 2,558 assertions. Pint passed.
- Frontend focused coverage passed: 11 tests across 6 files. Full Vitest passed:
  718 tests across 347 files. Production build and 12 account-lifecycle
  Playwright cases passed.
- Failure regressions prove prior delivered invitations survive transport
  failure, failed submissions do not consume quota, and wrapped delivery
  failures preserve the transport exception.

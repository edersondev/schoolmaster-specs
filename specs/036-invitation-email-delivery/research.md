# Research: Invitation Email Delivery

## Decision 1: Send synchronously to keep setup token transient

**Decision**: Submit invitation mail synchronously through Laravel's configured
mailer. Do not queue a mailable or job containing the plaintext setup token.

**Rationale**: Laravel supports both direct `send` and queued mail. Queueing
serializes mailable/job state into durable queue storage, which conflicts with
the existing rule that reusable invitation secrets are never persisted. Direct
submission keeps the token in request memory and can be verified with
`Mail::fake()` / `assertSent`.

**Alternatives considered**: Queueing the plaintext URL was rejected as durable
secret storage. Encrypting a queued payload was rejected because it adds key and
retry lifecycle complexity. A one-time exchange-code redesign was rejected as
new endpoint and data-model scope.

## Decision 2: Commit invitation, then submit mail, then mark accepted

**Decision**: Persist only the token hash with delivery fields unset, commit,
submit mail, and set safe delivery-request metadata only after transport
acceptance. On failure, keep the pending undelivered invitation and return a
documented retryable error.

**Rationale**: Sending inside a database transaction can produce a working email
for rolled-back state. Sending after commit avoids that inconsistency. Keeping
failed state supports audit and safe replacement; retry supersedes the unknown
token. A timeout may be ambiguous, but any replacement invalidates an earlier
link.

**Alternatives considered**: Roll back after mail failure was rejected because
mail may have been accepted before a timeout. Marking delivery requested before
send was rejected as false confirmation. Returning 201 on failure was rejected
because administrators need accurate retry feedback.

## Decision 3: Use trusted frontend-origin configuration

**Decision**: Read `APP_FRONTEND_URL` through cached application configuration,
require an absolute HTTP(S) origin, strip trailing slash, and append the existing
public invitation setup route with an encoded token.

**Rationale**: API `APP_URL` points to backend and request host/header data is
attacker-influenced. Environment-backed configuration supports each deployment
without hardcoding production URLs. Laravel guidance keeps `env()` access in
configuration files so configuration caching remains safe.

**Alternatives considered**: Request-origin links were rejected for host-header
injection. Backend `APP_URL` was rejected because it targets the API. A token in
query parameters was rejected because the existing frontend path contract is
already implemented and tested.

## Decision 4: Preserve API default; enforce frontend invitation setup

**Decision**: Administrator forms expose no account-setup choice, and the
frontend request mapper always submits `invitation`. The public user-create
schema and backend DTO continue defaulting omitted `account_setup_mode` to
`active`.

**Rationale**: Active creation produces no usable human onboarding path in the
administrator UI because it issues no known credential or setup message.
Invitation-only frontend creation provides one complete path while preserving
backward compatibility for existing API clients.

**Alternatives considered**: Changing the OpenAPI/backend default was rejected
as a breaking semantic change. Keeping Active immediately in the frontend was
rejected because it creates an active account without giving the user a usable
credential.

## Decision 5: Reuse existing two-step administration flow

**Decision**: User persistence and invitation creation remain separate explicit
actions. Email sends from `createAccountInvitation`, never from `createUser`.

**Rationale**: Existing Feature 034 behavior makes persistence and delivery
outcomes distinguishable and retryable. It also prevents sending from draft
data or creating hidden lifecycle side effects.

**Alternatives considered**: Automatic send on user creation was rejected
because a mail outage would conflate persisted-user success with delivery
failure. A new combined endpoint was rejected as unnecessary contract growth.

## Decision 6: Use standard 503 failure envelope

**Decision**: Add reusable `temporary_unavailable` error response and map a
focused delivery exception to HTTP 503.

**Rationale**: Transport/configuration failure is temporary infrastructure
failure, not validation or lifecycle conflict. Existing API response structure
supports machine-readable code, safe message, empty details, and status 503.

**Alternatives considered**: `409` was rejected because user/invitation state
may be valid. Undocumented `500` was rejected as non-actionable and contrary to
contract-first governance.

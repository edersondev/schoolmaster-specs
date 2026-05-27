# Research: Backend Account Lifecycle Workflows

## Decision: Define the next backend slice as account lifecycle workflows

**Rationale**: `007-administration-lifecycle` covers resource maintenance but intentionally excludes invitations, password setup, password reset, account recovery, and lock recovery. User creation exists without operational onboarding or recovery. The next slice should close that account access gap before roster, teacher correction, guardian self-service, reporting expansion, or frontend delivery depends on incomplete security behavior.

**Alternatives considered**:

- Start classroom/course/section/roster modeling next: rejected because school users still cannot be onboarded or recovered through contract-governed workflows.
- Start teacher workflow corrections next: rejected because teacher account access and recovery must be stable before expanding teacher operations.
- Start frontend onboarding screens now: rejected because backend account lifecycle contracts are not yet published.

## Decision: Expand OpenAPI before backend implementation

**Rationale**: Current contracts publish authentication and user administration foundations but do not publish invitation, password setup, password reset, account recovery, account lock, or reactivation behavior. Contract-first governance requires operation IDs, request schemas, token lifetime semantics, non-enumerating responses, invalid-token responses, authorization rules, tenant-context errors, bearer-token revocation rules, and audit event expectations before backend exposure.

**Alternatives considered**:

- Implement backend routes first and backfill OpenAPI later: rejected because it violates API-first governance and risks frontend/backend drift.
- Reuse existing user update endpoints with action-like payload fields: rejected because account lifecycle actions need distinct token, authorization, audit, and security semantics.

## Decision: Separate platform account administration from school user administration

**Rationale**: Platform account administrators manage platform-scoped users only. School user administrators manage same-school users only under an active permitted school context. This preserves the existing platform/school authorization separation and prevents platform users from becoming implicit school-owned account lifecycle operators.

**Alternatives considered**:

- Allow platform account administrators to manage all school-scoped accounts: rejected because it creates an undocumented support override and weakens tenant boundaries.
- Allow teachers or principals to recover accounts by role name: rejected because actor roles must be permission-based and contract-governed, not inferred from workflow labels.
- Centralize all account lifecycle operations under platform administrators: rejected because school administration needs same-school user onboarding and recovery without platform intervention.

## Decision: Use hashed, single-use, scoped lifecycle tokens

**Rationale**: Invitation and reset tokens grant account access or credential replacement. They must never be stored or returned as reusable plaintext. Token records should keep only hash, status, scope, target user, actor, expiry, supersession, completion, delivery metadata, and audit context. Completion must validate token hash, expiry, scope, current user state, and supersession before changing account state.

**Alternatives considered**:

- Store plaintext tokens for resend or support lookup: rejected because token disclosure would compromise account access.
- Use reusable tokens until expiry: rejected because password setup and reset need one-time proof.
- Share one token model without scope metadata: rejected because platform and school account lifecycle need separate authorization and tenant checks.

## Decision: Set token expiry to 7 days for invitations and 30 minutes for password resets

**Rationale**: Invitations need enough time for school operations and user onboarding, while password resets directly permit credential replacement and should have a short validity window. The clarification session fixed these values as product rules for contract and test design.

**Alternatives considered**:

- 24-hour invitations: rejected because school onboarding may require administrator coordination and user availability.
- 60-minute password resets: rejected because 30 minutes is sufficient for self-service recovery while reducing exposure.
- Undocumented expiry controlled only by configuration: rejected because frontend, support, and tests need stable contract behavior.

## Decision: Supersede prior unused tokens on create or resend

**Rationale**: Creating or resending an invitation or reset token invalidates all prior unused tokens of the same type for that user and scope. This reduces confusion, ensures the newest delivery is authoritative, and avoids multiple simultaneously valid account access paths.

**Alternatives considered**:

- Allow multiple unused tokens until each expires: rejected because it creates ambiguous completion and support behavior.
- Reuse the same token until expiry: rejected because resending should rotate secrets and reduce exposure.
- Supersede invitations only: rejected because reset tokens are at least as sensitive as invitation tokens.

## Decision: Revoke bearer tokens after credential or account access-state changes

**Rationale**: Password reset completion, administrative lock, unlock/recovery, reactivation, and user deactivation revoke all active bearer tokens for the affected user. This creates predictable security behavior after credential replacement or access-state changes and aligns with inactive-user rejection in the backend foundation.

**Alternatives considered**:

- Revoke only the current token: rejected because other active sessions could continue after a credential or access-state change.
- Preserve tokens after recovery or reactivation: rejected because access-state transitions should force a clean authentication boundary.
- Rely only on 8-hour bearer token expiry: rejected because security-sensitive account changes need immediate effect.

## Decision: Use non-enumerating password reset requests with request throttling and reset-token failure suppression

**Rationale**: Password reset requests should not disclose whether an identifier exists, is inactive, locked, deleted, belongs to a tenant, or has exceeded a reset request limit. Password reset requests are limited to 3 per account or IP within 1 hour; over-limit requests return the same non-enumerating accepted envelope while creating no new token or email delivery request. Failed reset-token completions are limited to 5 per account or IP within 15 minutes, then new reset tokens are blocked for 15 minutes without locking the whole user account. This preserves recovery safety, limits email/token abuse, and avoids making reset behavior an account-level denial-of-service path.

**Alternatives considered**:

- Return account-specific reset errors at request time: rejected because it enables account enumeration.
- Allow unlimited reset requests while superseding tokens: rejected because it can abuse email delivery and token generation while remaining invisible to users.
- Use a 24-hour reset request cap: rejected because it is more likely to block legitimate recovery after user error or delivery delay.
- Lock the whole account after failed reset-token completions: rejected because attackers could abuse reset attempts to deny access.
- Audit only with no throttling: rejected because repeated token guessing needs an operational control.

## Decision: Apply a clear password policy for setup and reset

**Rationale**: Password setup and reset completion accept passwords from 12 to 128 characters, reject common passwords, and allow paste/password-manager input. This gives administrators and users a strong, testable rule without brittle composition requirements or UI-hostile paste blocking.

**Alternatives considered**:

- Require uppercase, lowercase, number, and symbol: rejected because composition rules often produce predictable passwords and worse password-manager ergonomics.
- Use 8-character minimum: rejected because account lifecycle covers administrative and school access and should set a higher baseline.
- Defer password policy to implementation only: rejected because request validation and acceptance tests need contract-visible behavior.

## Decision: Limit invitation sends/resends and revoke after repeated failed completion

**Rationale**: Invitation sends and resends are limited to 3 per user and scope per 24 hours. Failed invitation-token completions are limited to 5 per invitation, account, or IP within 15 minutes, after which the invitation is revoked. This bounds abuse, protects onboarding tokens, and lets an administrator intentionally send a fresh invitation after failure.

**Alternatives considered**:

- Allow unlimited resends while superseding tokens: rejected because it can spam users and hide operational abuse.
- Block invitation completion temporarily after failures: rejected because revocation plus explicit resend gives a cleaner administrator-controlled recovery path.
- Audit only without limits: rejected because token guessing and resend abuse need enforceable controls.

## Decision: Keep administrative locks durable until authorized clearance

**Rationale**: Administrative account locks remain active until an authorized unlock or recovery action clears them. Time-limited lockout is already covered by failed-login behavior. Administrative locks need durable, auditable semantics for support and security operations.

**Alternatives considered**:

- Require expiry dates on administrative locks: rejected because operational security holds may not have a known end time.
- Auto-clear administrative locks after 24 hours: rejected because it weakens explicit administrative action and auditability.
- Exclude administrative lock mutation entirely: rejected because account recovery and reactivation workflows need a clear lock state boundary.

## Decision: Represent email delivery request metadata, not messaging implementation

**Rationale**: This slice may request and audit invitation or reset email delivery metadata, but it must not implement messaging templates, SMS delivery, notification preferences, inboxes, campaigns, parent portal communication, provider-specific delivery behavior, or delivery-provider secrets. Account lifecycle can define the email delivery request boundary without owning a messaging product.

**Alternatives considered**:

- Include email provider implementation in this slice: rejected because messaging is explicitly outside the backend roadmap scope.
- Include SMS delivery metadata: rejected because SMS requires separate consent, formatting, and delivery assumptions not specified here.
- Block account lifecycle until messaging is specified: rejected because email delivery request metadata and token state can be contract-governed first.

## Decision: Treat reset-request throttling as a contract-visible security rule

**Rationale**: Reset request throttling affects OpenAPI response semantics, email delivery request creation, audit expectations, and backend tests. Keeping the externally visible response as the same accepted envelope preserves non-enumeration while making token creation and delivery suppression a clear backend requirement.

**Alternatives considered**:

- Leave every token and retry setting to implementation configuration: rejected because client behavior and tests require contract-visible semantics.
- Report an explicit rate-limit response for over-limit reset requests: rejected because it would expose account or identifier state.
- Defer reset request limits to OpenAPI drafting: rejected because backend abuse controls and tests would remain under-specified.

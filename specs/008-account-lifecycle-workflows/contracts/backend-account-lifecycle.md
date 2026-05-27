# Contract Boundary: Backend Account Lifecycle Workflows

## Purpose

This feature creates a public API surface for account lifecycle workflows across platform and school-scoped users. Backend implementation must wait until OpenAPI documents the exact operations, parameters, request schemas, response schemas, errors, and operation IDs.

## Authoritative Contracts

- Aggregate contract: `api/openapi.yaml`
- Platform feature contract: `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- Source-of-truth guide: `AGENTS.md`
- Backend implementation guidance: `docs/backend-guidelines.md`
- Multi-tenant guidance: `docs/multi-tenant.md`
- Security guidance: `docs/security.md`
- Tenant decision: `decisions/004-use-tenant-by-column.md`

## Proposed Operation Boundary

The exact operation IDs must be finalized in OpenAPI, but the backend slice should be limited to this surface:

| Resource | Proposed Operations | Boundary |
|----------|---------------------|----------|
| `AccountInvitation` | create invitation, resend or replace invitation where approved, complete invitation with password setup | Platform account administrators for platform users; school user administrators for same-school users only; 3 sends/resends per user and scope per 24 hours; revoke after 5 failed completions per invitation/account/IP within 15 minutes |
| `PasswordSetup` | complete initial password setup through valid invitation token | Token-proven operation; passwords are 12 to 128 characters, not common passwords, and compatible with paste/password-manager input; no authenticated session is created unless OpenAPI explicitly documents it |
| `PasswordResetRequest` | request password reset, complete password reset | Non-enumerating request behavior; 3 requests per account or IP per 1 hour with no token or email delivery request after the limit; 30-minute token expiry; token supersession; reset-token failure suppression |
| `AccountLock` | review, administratively lock, unlock where approved | Preserve failed-login lockout and same-scope actor boundaries; administrative locks remain active until authorized unlock or recovery |
| `AccountRecovery` | recover locked account or reactivate eligible account where approved | Must not bypass inactive school, soft-deleted user, role dependency, or tenant authorization rules |
| `BearerTokenRevocation` | revoke active bearer tokens as a side effect of approved transitions | Required after password reset, administrative lock, unlock/recovery, reactivation, and user deactivation |

No backend implementation may expose these or adjacent routes until OpenAPI documents them.

## Required Contract Expansion

OpenAPI must define, at minimum:

- operation IDs and versioned `/api/v1` paths for every approved account lifecycle action
- request schemas for invitation creation, invitation completion, password setup, reset request, reset completion, lock, unlock, recovery, and reactivation where approved
- response schemas for accepted reset request, token completion, recovery outcomes, invalid token outcomes, conflict outcomes, and audit-safe metadata where exposed
- token semantics: invitation token expiry of 7 days, password reset token expiry of 30 minutes, single-use completion, supersession of prior unused tokens of the same type for the same user and scope, and invalid-token response behavior
- password policy: 12 to 128 characters, common-password blocklist rejection, and compatibility with paste/password-manager input
- invitation send/resend limit: 3 sends or resends per user and scope within 24 hours
- invitation failed completion limit: 5 failed invitation-token completion attempts per invitation, account, or IP within 15 minutes revokes the invitation
- password reset request limit: 3 reset requests per account or IP within 1 hour; over-limit requests return the same non-enumerating accepted envelope while creating no reset token and no email delivery request
- reset-token failure suppression: 5 failed reset-token completions per account or IP within 15 minutes blocks new reset token creation for 15 minutes without locking the whole account
- bearer-token revocation semantics after password reset completion, administrative lock, unlock/recovery, reactivation, and user deactivation
- administrative lock semantics: administrative locks remain active until authorized unlock or recovery clears them
- non-enumerating password reset request behavior for missing, inactive, locked, deleted, unauthorized, and eligible accounts
- actor rules for platform account administrators and school user administrators
- school tenant-context errors for school-owned account lifecycle operations
- validation errors for invalid credentials, invalid lifecycle transitions, invalid token state, inactive references, incompatible role dependencies, duplicate active token state, unsupported fields, and cross-tenant references
- audit event requirements and fields safe for response or support review
- email delivery request metadata behavior without introducing provider-specific messaging, SMS delivery, notification, inbox, or provider-specific implementation
- not-found behavior that does not reveal cross-tenant or ineligible account existence

## Required Response Shapes

Backend implementation must follow only the response statuses, content types, and components declared on each approved OpenAPI operation. Expected response categories include:

- success envelope for invitation creation, invitation completion, password setup, reset completion, lock, unlock, recovery, and reactivation outcomes
- non-enumerating accepted envelope for password reset requests where required
- validation error envelope
- unauthorized response
- forbidden response for valid scope but insufficient permission
- tenant mismatch or inactive tenant response for school-scoped operations
- inactive-record response where the contract distinguishes inactive user or school state
- token-invalid response for expired, reused, superseded, malformed, missing, mismatched, or scope-incompatible tokens
- conflict response for blocked recovery, role dependency conflict, soft-deleted user, inactive school, unresolved invitation, unresolved password setup, or current-state mismatch
- not-found response that does not disclose cross-tenant records

OpenAPI must document `403` outcomes that let clients distinguish a tenant-context failure (`error.code = tenant_mismatch`) from a valid-scope permission failure (`error.code = forbidden`) without introducing undocumented status codes.

No backend-local product envelope, ad hoc error response, undocumented status code, undocumented field, undocumented filter, undocumented sort behavior, undocumented status value, undocumented lifecycle action, or secret token echo is approved in this slice.

## Tenant Behavior

- School-scoped account lifecycle operations use the documented `X-School-Id` tenant context behavior when the authenticated actor is not already bound to exactly one active school.
- V1 school-owned records use `school_id` as the concrete tenant column.
- Missing, mismatched, inactive, or unauthorized tenant context must fail before school-owned target user lookup, token lookup, authorization checks, recovery evaluation, lock evaluation, persistence, or response shaping.
- Platform account lifecycle operations are platform-scoped and must not expose school-owned account records.
- Platform account administrators do not receive implicit permission to perform school-owned invitation, reset, recovery, lock, reactivation, user, role, student, teacher, guardian, report, or content actions.

## Authorization Behavior

- All administrative account lifecycle operations require authenticated access and active actor status.
- Platform account administrators may manage platform user account lifecycle workflows only.
- School user administrators may manage same-school user account lifecycle workflows only under an active permitted school context.
- Password setup and reset completion are token-proven operations and must validate token scope, target account eligibility, and current account state before changing credentials or access state.
- Teachers, students, and guardians do not receive account lifecycle administration access through teacher workflow, student self-view, or guardian association permissions.
- Any future platform support override must be separately specified, documented in OpenAPI, and covered by regression tests.

## Validation Behavior

- Invitation creation accepts only documented target identity, role/scope inputs, and delivery metadata.
- Invitation creation rejects inactive schools, inactive or deleted users, cross-tenant roles, inactive roles, unsupported user states, duplicate active invitations where prohibited, unsupported actors, and undocumented fields.
- Invitation creation and resend reject attempts beyond 3 sends/resends per user and scope within 24 hours.
- Invitation completion rejects expired, used, superseded, revoked, malformed, missing, mismatched, or scope-incompatible tokens.
- Invitation completion revokes the invitation after 5 failed completion attempts per invitation, account, or IP within 15 minutes.
- Password setup and reset completion reject passwords shorter than 12 characters, longer than 128 characters, or found in the common-password blocklist while preserving paste/password-manager compatibility.
- Password reset request responses are non-enumerating unless OpenAPI explicitly documents authenticated visibility for a specific actor.
- Password reset requests beyond 3 per account or IP within 1 hour must return the same non-enumerating accepted envelope and must not create a reset token or email delivery request.
- Password reset completion rejects expired, used, superseded, revoked, malformed, missing, mismatched, or scope-incompatible reset tokens.
- Password reset completion must change credentials and revoke all active bearer tokens atomically.
- Failed reset-token completion attempts must enforce the 5-per-account-or-IP-within-15-minutes threshold and 15-minute new-token suppression.
- Account lock, unlock, recovery, and reactivation accept only documented transitions for the current account state.
- Administrative locks remain active until authorized unlock or recovery clears them.
- Reactivation validates active school where school-scoped, active role assignments, password setup state, lock state, and tenant or platform scope.
- Recovery must reject attempts that bypass inactive-school rejection, soft-deleted-user rules, role dependency conflicts, platform/school authorization separation, or unresolved invitation/password setup state.

## Blocked Until Contract Expansion

These behaviors are outside this implementation boundary until OpenAPI documents them:

- classroom, course, section, group, roster, or teacher assignment workflows
- teacher grade, attendance, questionnaire, learning-set, or content correction workflows
- guardian self-service or guardian student views
- student academic record correction workflows
- report retry, cancellation, deletion, restore, designer, custom reports, additional formats, or report output lifecycle changes
- platform support-user access to school-owned records
- frontend implementation
- provider-specific email delivery, SMS delivery, messaging, notification preferences, inboxes, campaigns, delivery templates, or parent portal communication
- permanent purge, anonymization, legal hold, or retention management
- billing or undocumented APIs
- additional filters, sorting options, response fields, status values, lifecycle actions, token modes, or authorization exceptions

## Verification Boundary

Backend implementation PRs for this slice must link to every operation ID implemented and include evidence for:

- OpenAPI validation of the aggregate and platform contracts
- PHPUnit feature coverage for every approved operation
- response-shape checks for success and standard error envelopes
- non-enumerating password reset request behavior
- tenant-isolation failures for missing, mismatched, inactive, and unauthorized school context
- authorization failures for platform account administrator versus school user administrator separation
- validation failures for invalid credentials, invalid token state, invalid lifecycle transitions, unsupported fields, inactive references, role dependency conflicts, and cross-tenant references
- token expiry checks for 7-day invitations and 30-minute reset tokens
- token reuse and token supersession checks
- password length, common-password rejection, and password-manager-compatible input checks
- invitation send/resend limit checks
- failed invitation completion revocation checks
- password reset request throttle checks that verify no account enumeration, token creation, or email delivery request after 3 requests per account or IP within 1 hour
- reset-token failure suppression checks
- administrative lock durability checks
- bearer-token revocation checks after password reset, administrative lock, unlock/recovery, reactivation, and user deactivation
- audit event coverage without plaintext credentials, reusable lifecycle tokens, credential hashes, delivery-provider secrets, or unauthorized cross-tenant details

# Feature Specification: Backend Account Lifecycle Workflows

**Feature Branch**: `008-account-lifecycle-workflows`  
**Created**: 2026-05-27  
**Status**: Draft  
**Input**: User description: "Define the next SchoolMaster backend implementation slice after implemented administration lifecycle management. The backend must define account lifecycle workflows for invitations, password setup, password reset, account reactivation, and account lock or recovery behavior. Preserve contract-first delivery, tenant-by-column isolation with School as tenant root and school_id for school-owned records, explicit platform versus school authorization, API-only /api/v1 behavior, OpenAPI-aligned response envelopes, validation, inactive-user handling, audit events, token lifetime and delivery assumptions, and allowed actor roles before implementation. Do not include classroom/course/section/roster workflows, teacher correction workflows, guardian self-service, report lifecycle expansion, frontend implementation, billing, messaging, or undocumented APIs in this slice."

## Clarifications

### Session 2026-05-27

- Q: What expiry windows should account lifecycle tokens use? → A: Invitation tokens expire after 7 days; password reset tokens expire after 30 minutes.
- Q: How should existing bearer tokens be handled after credential or account access-state changes? → A: Revoke all active bearer tokens after password reset, administrative lock, unlock/recovery, reactivation, and user deactivation.
- Q: How should new or resent invitation and reset tokens affect prior unused tokens? → A: Creating or resending an invitation or reset token invalidates all prior unused tokens of the same type for that user and scope.
- Q: Which actors may perform account lifecycle administration? → A: Platform account administrators manage platform users; school user administrators manage same-school users only.
- Q: How should failed reset-token completion attempts be limited? → A: Allow 5 failed reset-token completions per account or IP within 15 minutes, then block new reset tokens for 15 minutes.
- Q: What password policy applies to password setup and reset completion? → A: Passwords must be 12 to 128 characters, block common passwords, and allow paste/password managers.
- Q: How should invitation sends and resends be limited? → A: Allow 3 invitation sends or resends per user and scope per 24 hours.
- Q: How long do administrative account locks remain active? → A: Administrative locks remain until an authorized unlock or recovery action clears them.
- Q: What delivery metadata is in scope for account lifecycle tokens? → A: Email delivery request metadata only; no provider-specific messaging behavior.
- Q: How should failed invitation-token completion attempts be limited? → A: Revoke the invitation after 5 failed completion attempts per invitation, account, or IP within 15 minutes.
- Q: How should password reset request sends be throttled? → A: Limit password reset requests to 3 per account or IP per 1 hour; over-limit requests return the same accepted envelope but create no token or email delivery request.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Invite Users and Complete Initial Password Setup (Priority: P1)

A platform or school administrator invites an eligible user to activate access for the correct platform or school scope, and the invited user completes first password setup before authentication is allowed.

**Why this priority**: User creation exists, but operational onboarding remains blocked until invitations and first-password setup are governed by explicit token, actor, tenant, and audit rules.

**Independent Test**: Authenticate as an authorized administrator, invite a same-scope user, complete password setup with the issued invitation token, and verify the user can authenticate only after successful setup while invitation attempts outside the permitted scope are rejected.

**Acceptance Scenarios**:

1. **Given** a school user administrator has an active resolved school context and permission to manage same-school users, **When** they invite a same-school user with valid role and contact information, **Then** an invitation is created for that school scope and the response follows the approved envelope without exposing token secrets.
2. **Given** a platform account administrator has platform account-management permission, **When** they invite a platform-scoped user, **Then** the invitation is created without granting school-owned resource access.
3. **Given** an invited user receives a valid unexpired invitation, **When** they complete password setup with credentials that are 12 to 128 characters, not common passwords, and compatible with paste/password managers, **Then** the account becomes eligible for authentication in the documented scope and the invitation cannot be reused.
4. **Given** an invitation request references an inactive school, inactive role, cross-tenant role, existing active account, unsupported actor, or undocumented field, **When** the request is submitted, **Then** the request is rejected before creating invitation state.
5. **Given** an administrator has already sent or resent 3 invitations for the same user and scope within 24 hours, **When** they attempt another invitation send or resend, **Then** the request is rejected with the documented rate-limit response and no new token is created.
6. **Given** failed invitation-token completion attempts reach 5 failures for the same invitation, account, or IP within 15 minutes, **When** another completion attempt is submitted, **Then** the invitation is revoked and can no longer be completed.

---

### User Story 2 - Reset Passwords Securely (Priority: P2)

A user who cannot access their account requests a password reset, proves possession of the reset token, and sets a new password without weakening tenant or inactive-account rules.

**Why this priority**: Password recovery is a core operational workflow and a security-sensitive surface. It must be contract-governed before the backend exposes reset behavior.

**Independent Test**: Request a reset for an eligible active user, complete the reset with a valid unexpired single-use token, verify old credentials no longer work, and verify invalid, expired, reused, inactive-user, and cross-scope attempts are rejected with documented responses.

**Acceptance Scenarios**:

1. **Given** an eligible active user requests password reset, **When** the submitted account identifier is valid, **Then** the system accepts the request using a non-enumerating response and records audit context.
2. **Given** a valid unexpired reset token, **When** the user submits replacement credentials that are 12 to 128 characters, not common passwords, and compatible with paste/password managers, **Then** the password is changed, all active bearer tokens are revoked, and the reset token cannot be reused.
3. **Given** a reset request targets a missing, inactive, locked, deleted, or unauthorized account, **When** the request is submitted, **Then** the externally visible response does not reveal account existence or scope beyond the published contract.
4. **Given** a reset completion uses an expired, malformed, reused, mismatched, superseded, or previously replaced token, **When** the request is submitted, **Then** no credential state changes occur.
5. **Given** failed reset-token completion attempts reach 5 failures for the same account or IP within 15 minutes, **When** another password reset is requested during the suppression window, **Then** new reset tokens are blocked for 15 minutes and the event is audited without locking the whole user account.
6. **Given** password reset requests reach 3 requests for the same account or IP within 1 hour, **When** another reset request is submitted, **Then** the response remains the documented non-enumerating accepted envelope and no new token or email delivery request is created.

---

### User Story 3 - Recover Locked or Inactive Accounts (Priority: P3)

An authorized administrator reviews locked or inactive account state and performs documented recovery or reactivation actions without bypassing school lifecycle, user lifecycle, or authorization boundaries.

**Why this priority**: Account lockout and reactivation are less frequent than onboarding and password reset, but they affect support operations, auditability, and inactive-user behavior established by earlier backend slices.

**Independent Test**: Lock an account through documented failed-access behavior or administrative action, recover it with an authorized actor in the correct scope, and verify unauthorized, inactive-school, cross-tenant, and unsupported recovery attempts remain blocked.

**Acceptance Scenarios**:

1. **Given** an account is locked by documented security rules, **When** an authorized same-scope administrator performs a permitted recovery action, **Then** the account lock state is cleared or transitioned according to the published contract and an audit event is recorded.
2. **Given** an account is administratively locked, **When** no authorized unlock or recovery action has cleared it, **Then** the administrative lock remains active and continues to block authentication regardless of elapsed time.
3. **Given** a user is inactive but eligible for reactivation under administration lifecycle rules, **When** an authorized actor reactivates account access, **Then** the account can resume authentication only if the user, school, role assignments, and required password state are all valid.
4. **Given** an inactive school, deleted user, missing role dependency, unresolved invitation, or unresolved password setup blocks access, **When** recovery is attempted, **Then** the request is rejected with the documented conflict or inactive-record response.
5. **Given** a platform account administrator attempts to recover a school-scoped account without explicit permitted school context and school user administration authorization, **When** the request is submitted, **Then** access is rejected rather than treated as a cross-tenant override.

### Edge Cases

- How does the backend prevent account enumeration during invitation lookup, password reset request, and reset completion?
- What happens when an invitation or reset token is expired, reused, superseded by a newer token, malformed, missing, or tied to a different account or scope?
- How does the backend revoke all active bearer tokens after password reset, administrative lock, unlock/recovery, reactivation, and user deactivation while keeping first password setup independent from prior authenticated sessions?
- What happens when a user belongs to an inactive school, has no active role assignment, has been soft-deleted, or is inactive under administration lifecycle rules?
- How does the backend reject missing, inactive, mismatched, or unauthorized `X-School-Id` tenant context before school-owned invitation, recovery, or reactivation data access?
- What happens when a platform account administrator tries to manage school-scoped account lifecycle without explicit permitted school context and school user administration authorization?
- How does the backend prevent plaintext password, invitation token, reset token, or recovery secret exposure in responses, logs, audit events, and delivery metadata?
- How does the backend block new reset tokens for 15 minutes after 5 failed reset-token completions per account or IP within 15 minutes without locking the whole user account?
- How does the backend return a non-enumerating accepted envelope while suppressing token creation and email delivery after 3 password reset requests per account or IP within 1 hour?
- How does the backend reject password setup or reset completion passwords shorter than 12 characters, longer than 128 characters, or found in the common-password blocklist while allowing paste and password-manager input?
- How does the backend block invitation sends or resends beyond 3 attempts per user and scope within 24 hours without affecting other users or scopes?
- How does the backend revoke an invitation after 5 failed completion attempts per invitation, account, or IP within 15 minutes?
- How does the backend keep administrative locks active until an authorized unlock or recovery action clears them while preserving separate time-limited failed-login lockout behavior?
- How does the backend avoid implementing notification messaging, guardian self-service, roster workflows, teacher correction workflows, report lifecycle expansion, frontend behavior, billing, or undocumented API behavior in this slice?

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Implement an API-only account lifecycle slice for invitation creation and completion, password setup, password reset request and completion, account reactivation, lock state review where documented, and recovery actions. Backend work must include request validation, authorization policies, service-layer account transition rules, response resources, tenant context enforcement, audit events, security-sensitive token handling, inactive-user handling, and regression tests.
- **Frontend repository impact**: No frontend implementation is included in this feature. Future onboarding, password setup, password reset, and recovery screens must consume only the OpenAPI operations approved by this slice.
- **Specification or contract repository impact**: OpenAPI must be expanded before backend exposure because current completed slices do not publish invitation, password setup, password reset, account reactivation, or recovery operations.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines account lifecycle boundaries first, `schoolmaster-backend` implements only approved operations next, and `schoolmaster-frontend` consumes the behavior only after backend verification remains contract-compliant.

### API Contract Impact

- **OpenAPI update required**: Yes. The active feature contract and aggregate contract must define operation IDs, paths, request schemas, response schemas, token lifetime semantics, non-enumerating responses, inactive-account responses, conflict responses, audit-relevant state transitions, and allowed actor roles before backend implementation begins.
- **Versioned endpoints affected**: Proposed contract surface is limited to account lifecycle operations under `/api/v1/account-invitations`, `/api/v1/account-invitations/{invitationToken}/setup`, `/api/v1/auth/password-reset-requests`, `/api/v1/auth/password-resets`, `/api/v1/users/{userId}/account-reactivation`, and `/api/v1/users/{userId}/account-lock` unless OpenAPI approves a different path set.
- **JSON response impact**: Responses must use documented success, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, not-found, non-enumerating acceptance, and token-invalid envelopes. No backend-local envelope, ad hoc error code, undocumented status code, undocumented field, or secret token echo is approved.
- **Authentication/authorization impact**: Invitation and administrative recovery operations require authenticated authorized actors. Password setup and reset completion use documented single-use token proof. Platform account administrators may manage platform users. School user administrators may manage same-school users only and require active permitted school context. Platform administration does not imply school-owned account lifecycle access.
- **Compatibility impact**: This is additive against the current backend foundation and administration lifecycle behavior. Any change to login token expiry, failed-login lockout, user lifecycle, role assignment, school lifecycle, student self-view, or reporting access requires explicit contract revision and compatibility review.

### Data & Tenancy Impact

- **Tenant scoping impact**: School-scoped account lifecycle state for users with school roles remains governed by the resolved active `school_id` context. Platform-scoped account lifecycle state is managed only through platform-authorized operations.
- **Cross-tenant or platform access impact**: Platform users may manage platform-scoped accounts and documented school lifecycle where allowed. They do not receive implicit access to school-owned invitation, recovery, user role, student, teacher, guardian, report, or private content behavior.
- **Soft delete impact**: Soft-deleted users are not recoverable through account lifecycle operations unless administration lifecycle rules and OpenAPI explicitly allow restoration first. Account lifecycle tokens and audit history must remain retained according to documented security and support needs without exposing reusable secrets.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The backend implementation slice MUST be limited to documented invitation creation, invitation completion, initial password setup, password reset request, password reset completion, account lock review or mutation where allowed, account recovery, and account reactivation operations unless OpenAPI is updated first.
- **FR-002**: Every school-scoped account lifecycle operation MUST resolve an active permitted school context before validation that depends on school-owned users, roles, schools, invitation state, recovery state, authorization decisions, persistence, or response shaping.
- **FR-003**: Requests with missing, mismatched, inactive, or unauthorized school context MUST fail before any school-owned account, invitation, role, student, teacher, guardian, report, or recovery data is accessed.
- **FR-004**: Invitation creation MUST require a documented authorized actor, allowed account scope, eligible target user state, valid active role dependencies where applicable, and tenant-compatible contact identity before invitation state is created.
- **FR-004a**: Platform account administrators MUST be limited to platform-scoped user account lifecycle administration, and school user administrators MUST be limited to same-school user account lifecycle administration under an active permitted school context.
- **FR-005**: Invitation creation MUST reject inactive schools, inactive or deleted users, duplicate active invitations where prohibited, unsupported user states, cross-tenant role assignments, platform/school scope mismatches, undocumented fields, and unsupported delivery assumptions.
- **FR-006**: Invitation completion and first password setup MUST require a valid unexpired single-use invitation token and a password that is 12 to 128 characters, not found in the common-password blocklist, and compatible with paste/password-manager input before an invited account becomes eligible for authentication.
- **FR-007**: Invitation and password setup responses MUST NOT expose plaintext passwords, reusable invitation tokens, reset tokens, credential hashes, or delivery-provider secrets.
- **FR-008**: Password reset request behavior MUST use documented non-enumerating responses so callers cannot determine whether an account identifier exists, is inactive, is locked, or belongs to a particular tenant unless the contract explicitly permits that visibility for an authenticated actor.
- **FR-009**: Password reset completion MUST require a valid unexpired single-use reset token, eligible account state, and a replacement password that is 12 to 128 characters, not found in the common-password blocklist, and compatible with paste/password-manager input before changing credentials.
- **FR-010**: Password reset completion, administrative account lock, unlock or recovery, account reactivation, and user deactivation MUST revoke all active bearer tokens for the affected user; first password setup MUST NOT create or preserve any prior authenticated session unless OpenAPI explicitly documents that behavior.
- **FR-011**: Account lock behavior MUST preserve the failed-login lockout rules established by the backend foundation, and administrative locks MUST remain active until an authorized unlock or recovery action clears them.
- **FR-012**: Account reactivation MUST validate user lifecycle state, school lifecycle state, active role assignments, password setup state, lock state, and tenant or platform scope before allowing authentication to resume.
- **FR-013**: Account recovery MUST reject attempts that would bypass inactive-school rejection, soft-deleted-user rules, role dependency conflicts, school-scoped authorization, or platform versus school authorization separation.
- **FR-014**: Invitation tokens MUST expire after 7 days, password reset tokens MUST expire after 30 minutes, and creating or resending an invitation or reset token MUST invalidate all prior unused tokens of the same type for that user and scope.
- **FR-014a**: Password reset completion MUST allow at most 5 failed reset-token completion attempts per account or IP within 15 minutes, then block creation of new reset tokens for that account or IP for 15 minutes without locking the whole user account.
- **FR-014b**: Password reset requests MUST allow at most 3 requests per account or IP within 1 hour; further requests within that window MUST return the same non-enumerating accepted envelope while creating no new reset token and no email delivery request.
- **FR-014c**: Invitation sends and resends MUST allow at most 3 attempts per user and scope within 24 hours; further attempts within that window MUST be rejected without creating a new token.
- **FR-014d**: Invitation completion MUST allow at most 5 failed invitation-token completion attempts per invitation, account, or IP within 15 minutes, then revoke the invitation so it can no longer be completed.
- **FR-015**: Delivery behavior MUST be limited to auditable email delivery request metadata only; this slice MUST NOT implement provider-specific messaging behavior, SMS delivery, notification preferences, inboxes, campaigns, or parent portal communication features.
- **FR-016**: Backend audit events MUST record invitation creation, invitation resend or replacement where documented, invitation completion, password reset request acceptance, over-limit password reset request suppression, password reset completion, failed token completion, account lock, account unlock, recovery, reactivation, and blocked inactive-account attempts without storing plaintext credentials or reusable token values.
- **FR-017**: Backend validation MUST reject undocumented request fields, unsupported filters, unsupported sort values, invalid payload shapes, invalid credentials, invalid lifecycle transitions, invalid effective dates, duplicate active tokens where prohibited, inactive references, and cross-tenant references.
- **FR-018**: Backend responses MUST match the published OpenAPI success and error semantics declared for each approved operation. The backend MUST NOT emit undocumented status semantics unless OpenAPI first documents them on the relevant operation.
- **FR-019**: Backend authorization MUST keep platform account administration, school user administration, school administration lifecycle, teacher workflows, student self-view, guardian access, reporting, and support access permissions separate.
- **FR-020**: Backend implementation MUST NOT expose classroom, course, section, group, roster, teacher correction, guardian self-service, report lifecycle, report output, frontend, billing, messaging, notification-center, permanent purge, or undocumented API behavior in this slice.
- **FR-021**: Backend tests MUST cover successful flows, validation failures, authorization failures, tenant isolation failures, inactive school/user handling, password length and common-password rejection, password-manager-compatible input, invitation send/resend limits, failed invitation-token completion revocation, password reset request throttling without account enumeration, token expiry, token reuse, token supersession by newer or resent tokens, bearer-token revocation after credential or account access-state changes, non-enumerating reset responses, lock and recovery conflicts, audit event coverage, credential-secret non-exposure, and response shape for every operation in this slice.

### Key Entities *(include if feature involves data)*

- **AccountInvitation**: A 7-day, single-use onboarding record for a platform or school-scoped user, including target identity, intended scope, actor context, auditable email delivery request metadata, 3-per-user-and-scope-per-24-hours send/resend controls, 5-failures-per-invitation-account-or-IP completion controls, status, expiry, supersession state, revocation state, and completion state without exposing reusable token secrets.
- **PasswordSetup**: The initial credential setup transition tied to a valid invitation and user account state before the user can authenticate; it is represented by invitation completion state, user credential state, and audit events rather than a standalone API resource, and accepted passwords are 12 to 128 characters, not common passwords, and compatible with paste/password-manager input.
- **PasswordResetRequest**: A security-sensitive request to initiate credential recovery using non-enumerating response behavior, 3-requests-per-account-or-IP-per-1-hour request throttling, 30-minute token expiry, audit context, supersession state, 5-failures-per-account-or-IP reset-token completion controls, 15-minute new-token suppression, and auditable email delivery request metadata.
- **AccountLock**: A temporary failed-login or durable administrative account access restriction; administrative locks remain active until an authorized unlock or recovery action clears them.
- **AccountRecovery**: A documented administrative or token-proven transition that clears eligible lock or recovery states without bypassing user, school, role, or tenant constraints.
- **User**: Authenticated identity whose account lifecycle state determines invitation, password setup, password reset, lock, recovery, reactivation, and authentication eligibility.
- **PlatformAccountAdministrator**: Platform-scoped actor permitted to administer platform user account lifecycle workflows without receiving school-owned account lifecycle access.
- **SchoolUserAdministrator**: School-scoped actor permitted to administer same-school user account lifecycle workflows only within an active permitted school context.
- **Role**: Platform or school authorization grouping that must remain active and scope-compatible before an invited, recovered, or reactivated user can authenticate.
- **School**: Tenant root whose active or inactive state controls whether school-scoped account lifecycle workflows may proceed.
- **AuditEvent**: Security and administrative record of account lifecycle activity, including actor where available, target user, school where applicable, event type, outcome, timestamp, and tenant-safe metadata.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Backend reviewers can map 100% of implemented account lifecycle routes in this slice to approved OpenAPI operation IDs without finding undocumented product endpoints.
- **SC-002**: Tenant-isolation acceptance tests reject 100% of missing, mismatched, inactive, and unauthorized school-context attempts for school-scoped invitation, setup, reset, recovery, lock, and reactivation operations.
- **SC-003**: Invitation and password setup tests reject 100% of expired, reused, malformed, superseded by newer or resent tokens, over-limit invitation send/resend, revoked-after-failed-completion, cross-scope, inactive-user, inactive-school, too-short, too-long, and common-password completion attempts covered by this specification.
- **SC-004**: Password reset tests confirm 100% of reset request responses covered by this specification avoid account enumeration, requests beyond 3 per account or IP within 1 hour create no new token or email delivery request, invalid completion attempts leave credentials unchanged, too-short, too-long, and common replacement passwords are rejected, successful reset completion revokes all active bearer tokens, and new reset tokens are blocked for 15 minutes after 5 failed reset-token completions per account or IP within 15 minutes.
- **SC-005**: Authorization tests confirm 100% of platform account administrator actions and school user administrator actions remain separated by documented permissions and tenant scope.
- **SC-006**: Security review confirms 100% of account lifecycle responses, logs, and audit records avoid plaintext password, reusable invitation token, reset token, credential hash, and delivery-secret exposure.
- **SC-007**: Audit verification confirms 100% of invitation, password setup, password reset, administrative lock, unlock, recovery, reactivation, and blocked inactive-account events covered by this slice are recorded with tenant-safe context, and administrative locks remain active until authorized clearance.
- **SC-008**: Response-shape verification confirms every operation in this slice returns the documented success or error envelope for successful, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, token-invalid, non-enumerating acceptance, and not-found cases.
- **SC-009**: An authorized administrator can invite, recover, and reactivate an eligible same-scope user without exposing users, roles, tokens, or account state from another school or platform scope.

## Assumptions

- `007-administration-lifecycle` has implemented or approved user, role, school, active, inactive, soft-delete, restore, and dependency rules that account lifecycle workflows must respect.
- Existing backend authentication keeps bearer token expiry at 8 hours, rejects inactive users and inactive schools, rate-limits failed login attempts by email and IP, and audits login success, login failure, logout, and token rejection.
- Current OpenAPI contracts do not yet publish invitation, password setup, password reset, account lock recovery, or account reactivation operations; this feature must expand contracts before backend implementation.
- Invitation and reset email delivery may be requested and audited by this slice, but provider-specific messaging, SMS delivery, notification templates, inboxes, campaigns, and communication preferences remain outside scope.
- Platform-scoped and school-scoped accounts share account lifecycle concepts, but platform account administrator permissions, school user administrator permissions, tenant context, and visible account state remain separated.
- Classroom, course, section, roster, teacher correction, guardian self-service, report lifecycle expansion, frontend implementation, billing, messaging, notifications, and parent portal behavior remain outside this slice.

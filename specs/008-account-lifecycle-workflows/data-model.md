# Data Model: Backend Account Lifecycle Workflows

## Modeling Principles

- `School` remains the v1 tenant root.
- School-scoped account lifecycle records use `school_id` directly or inherit school ownership through the target user's same-school role assignment.
- Public identifiers crossing API boundaries use UUIDs.
- Platform account lifecycle and school account lifecycle remain separate authorization paths.
- Lifecycle tokens are single-use, scoped, hashed at rest, time-limited, and never returned by detail or list responses.
- New or resent lifecycle tokens supersede prior unused tokens of the same type for the same user and scope.
- Password reset request throttling must preserve non-enumerating responses while suppressing token creation and email delivery after 3 requests per account or IP within 1 hour.
- Password setup and reset completion accept only passwords from 12 to 128 characters that are not common passwords and remain compatible with paste/password-manager input.
- Account lifecycle delivery state stores auditable email delivery request metadata only; provider-specific messaging behavior and SMS delivery are out of scope.
- Account lifecycle transitions preserve audit history and record actor, target user, scope, outcome, and tenant-safe metadata.
- Cross-tenant access remains deny-by-default unless a future specification documents a support or platform override.

## Entities

### User

- **Purpose**: Authenticated identity whose account lifecycle state controls invitation, password setup, password reset, lock, recovery, reactivation, and authentication eligibility.
- **Core fields used in this slice**:
  - `id` (UUID)
  - identity and contact fields documented by OpenAPI
  - `status` (`invited`, `active`, `inactive`, `locked` where approved by contract)
  - password credential state (`unset`, `set`)
  - role assignment references
  - school relationship data where school-scoped
  - bearer token revocation marker or equivalent session invalidation metadata
  - soft-delete metadata from administration lifecycle where applicable
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to one or more schools through documented school-role assignments
  - has many role assignments
  - has many account invitations
  - has many password reset requests
  - has many account lock or recovery records
  - may be actor on account lifecycle audit events
- **Validation rules**:
  - platform user lifecycle administration requires platform account administrator permission
  - school user lifecycle administration requires active permitted school context and school user administrator permission
  - soft-deleted users cannot be recovered through account lifecycle until administration lifecycle restoration rules allow it
  - reactivation requires eligible user status, active school where school-scoped, active role assignments, valid password setup state, and no blocking lock state
  - password reset completion revokes all active bearer tokens
  - administrative lock, unlock/recovery, reactivation, and user deactivation revoke all active bearer tokens
  - administrative locks remain active until an authorized unlock or recovery action clears them

### PlatformAccountAdministrator

- **Purpose**: Platform-scoped actor permitted to administer platform user account lifecycle workflows.
- **Core fields used in this slice**:
  - actor user UUID
  - platform account administration permission
  - active user status
- **Validation rules**:
  - may invite, recover, lock, unlock, and reactivate platform users where OpenAPI approves the operation
  - must not manage school-scoped account lifecycle unless a future support-access specification grants an explicit exception
  - does not receive implicit access to school-owned users, roles, students, teachers, guardians, reports, or private content

### SchoolUserAdministrator

- **Purpose**: School-scoped actor permitted to administer account lifecycle workflows for same-school users.
- **Core fields used in this slice**:
  - actor user UUID
  - `school_id`
  - school user administration permission
  - active user and active school status
- **Validation rules**:
  - may invite, recover, lock, unlock, and reactivate users only inside the active resolved school context where OpenAPI approves the operation
  - cannot manage platform users or users in another school
  - missing, inactive, mismatched, or unauthorized school context fails before target account lookup

### AccountInvitation

- **Purpose**: A 7-day, single-use onboarding record for a platform or school-scoped user.
- **Core fields**:
  - `id` (UUID)
  - `target_user_id`
  - `school_id` for school-scoped invitations, nullable for platform-scoped invitations
  - `scope` (`platform`, `school`)
  - `token_hash`
  - `status` (`pending`, `completed`, `expired`, `superseded`, `revoked`)
  - `expires_at` (created timestamp plus 7 days)
  - `completed_at`
  - `superseded_at`
  - `revoked_at`
  - send/resend counter metadata by user, scope, and 24-hour window
  - failed completion counter metadata by invitation, account, and IP
  - `delivery_requested_at`
  - `delivery_channel` (`email`)
  - `email_delivery_metadata_summary`
  - `actor_user_id`
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to target `User`
  - belongs to actor `User`
  - optionally belongs to `School`
  - may have related audit events
- **Validation rules**:
  - creation requires an eligible target user, compatible scope, active role dependencies, and authorized actor
  - school-scoped invitation creation requires active permitted `school_id`
  - sends and resends are limited to 3 attempts per user and scope within 24 hours
  - creating or resending an invitation supersedes prior unused invitations of the same type for the same user and scope
  - completion requires matching token hash, pending status, unexpired token, compatible scope, eligible target user, and password setup inputs that satisfy the 12-to-128 character and common-password rules
  - 5 failed invitation-token completion attempts per invitation, account, or IP within 15 minutes revoke the invitation so it can no longer be completed
  - completion marks the token completed and makes the account eligible for authentication only where user, school, role, and password state are valid
  - plaintext token values must not be stored in account lifecycle state, audit events, logs, or API responses

### PasswordSetup

- **Purpose**: Initial credential setup transition tied to a valid invitation before the user can authenticate.
- **Persistence representation**:
  - represented by `AccountInvitation.completed_at`
  - represented by target `User` password credential state changing from `unset` to `set`
  - represented by an `AuditEvent` for password setup completion
  - no standalone password setup table or public API resource is required for this slice
- **Relationships**:
  - updates target `User`
  - completes one `AccountInvitation`
  - records related audit events
- **Validation rules**:
  - requires a valid account invitation token
  - cannot complete with an expired, used, superseded, revoked, or scope-mismatched invitation
  - must validate that passwords are 12 to 128 characters, not common passwords, and compatible with paste/password-manager input
  - must not create or preserve a prior authenticated session unless OpenAPI explicitly documents that behavior

### PasswordResetRequest

- **Purpose**: Security-sensitive credential recovery request using non-enumerating request behavior and a 30-minute single-use reset token.
- **Core fields**:
  - `id` (UUID)
  - `target_user_id` where resolvable internally
  - `school_id` for school-scoped account context where applicable
  - `account_identifier_hash` or equivalent non-public lookup metadata
  - `token_hash`
  - `status` (`pending`, `completed`, `expired`, `superseded`, `revoked`, `suppressed`)
  - `expires_at` (created timestamp plus 30 minutes)
  - `completed_at`
  - `superseded_at`
  - request counter metadata by account and IP for the 1-hour reset request window
  - failed completion counter metadata by account and IP
  - new-token suppression window metadata
  - `delivery_requested_at`
  - `delivery_channel` (`email`)
  - `email_delivery_metadata_summary`
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to target `User` when the identifier maps to an eligible user
  - optionally belongs to `School`
  - may have related audit events
- **Validation rules**:
  - request responses are non-enumerating for missing, inactive, locked, deleted, unauthorized, and eligible accounts unless OpenAPI explicitly documents authenticated visibility
  - requests beyond 3 per account or IP within 1 hour return the same non-enumerating accepted envelope while creating no new reset token and no email delivery request
  - creating or resending a reset token supersedes prior unused reset tokens for the same user and scope
  - completion requires matching token hash, pending status, unexpired token, compatible scope, eligible account state, and a replacement password that satisfies the 12-to-128 character and common-password rules
  - successful completion changes credentials, marks the token completed, and revokes all active bearer tokens for the affected user
  - 5 failed reset-token completions per account or IP within 15 minutes blocks creation of new reset tokens for that account or IP for 15 minutes without locking the whole user account
  - plaintext token values must not be stored in account lifecycle state, audit events, logs, or API responses

### AccountLock

- **Purpose**: Temporary failed-login or durable administrative restriction that blocks account authentication or account lifecycle actions where documented.
- **Core fields**:
  - `id` (UUID)
  - `user_id`
  - `school_id` where school-scoped
  - `lock_type` (`failed_login`, `administrative`, `recovery_hold` where approved by OpenAPI)
  - `status` (`active`, `cleared`)
  - `reason`
  - `locked_at`
  - `cleared_at`
  - `actor_user_id` where administrative
  - `created_at`
  - `updated_at`
- **Relationships**:
  - belongs to target `User`
  - optionally belongs to `School`
  - belongs to actor `User` where administrative
- **Validation rules**:
  - failed-login lockout rules from backend foundation remain preserved
  - administrative lock requires the same platform or school actor boundary as other account lifecycle actions
  - administrative lock revokes all active bearer tokens for the affected user
  - administrative locks remain active until an authorized unlock or recovery action clears them
  - lock state must be checked before password reset completion, reactivation, and authentication where OpenAPI requires it

### AccountRecovery

- **Purpose**: Documented administrative or token-proven transition that clears eligible lock or recovery states.
- **Core fields**:
  - `id` (UUID)
  - `user_id`
  - `school_id` where school-scoped
  - `recovery_type` (`unlock`, `reactivate`, `reset_required` where approved by OpenAPI)
  - `from_state`
  - `to_state`
  - `reason`
  - `actor_user_id`
  - `completed_at`
  - `created_at`
- **Relationships**:
  - belongs to target `User`
  - optionally belongs to `School`
  - belongs to actor `User`
  - may reference an `AccountLock`
- **Validation rules**:
  - recovery requires eligible current user state, active school where school-scoped, active role assignments, and same-scope actor authorization
  - recovery must not bypass inactive-school rejection, soft-deleted-user rules, role dependency conflicts, or platform/school authorization separation
  - unlock/recovery and reactivation revoke all active bearer tokens for the affected user

### AuditEvent

- **Purpose**: Security and administrative record of account lifecycle activity.
- **Core fields**:
  - `id` (UUID)
  - `school_id` for school-scoped events, nullable for platform-scoped account events
  - `actor_user_id` where authenticated
  - `target_user_id` where known
  - `event_type`
  - `outcome`
  - `ip_address`
  - `user_agent_summary`
  - `metadata_summary`
  - `created_at`
- **Relationships**:
  - belongs to actor `User` where authenticated
  - belongs to target `User` where known
  - optionally belongs to `School`
- **Validation rules**:
  - record invitation creation, resend or replacement, over-limit invitation send/resend attempts, invitation completion, failed invitation-token completion, invitation revocation, password reset request acceptance, over-limit password reset request suppression, password reset completion, failed reset-token completion, reset-token suppression, administrative lock, account unlock, recovery, reactivation, blocked inactive-account attempts, and bearer-token revocation outcomes
  - do not store plaintext passwords, reusable invitation tokens, reset tokens, credential hashes, delivery-provider secrets, private content, or unauthorized cross-tenant details

## Cross-Entity Rules

- School-scoped account lifecycle operations must resolve active permitted school context before target user lookup, token lookup, authorization, recovery checks, persistence, or response shaping.
- Platform account lifecycle operations must not expose school-owned account records or grant implicit school-scoped access.
- Target user scope, actor scope, role assignment scope, token scope, and optional `school_id` must match before invitation completion, password setup, reset completion, lock, recovery, or reactivation changes state.
- Soft-deleted users remain outside account lifecycle recovery until administration lifecycle restore behavior makes them eligible.
- Inactive schools block school-scoped invitation, setup, reset, recovery, lock, and reactivation behavior before module-specific logic runs.
- Bearer-token revocation must be atomic with successful password reset completion, administrative lock, unlock/recovery, reactivation, and user deactivation.
- Token completion and token supersession must be atomic with token state changes and audit events.
- Non-enumerating password reset request responses must not reveal whether a target identifier exists or why it is ineligible.
- Over-limit password reset requests must preserve the same accepted envelope while suppressing reset token creation and email delivery request creation.

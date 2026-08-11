# Contract Boundary: Complete Administrator Account Lifecycle UI

## Delivery Boundary

Contract/OpenAPI ships first, Laravel conformance second, Vue activation third.
Frontend must not enable lifecycle actions against older backend behavior.

## API Operations

| Operation ID | Method and path | Feature use |
|---|---|---|
| `listUsers` | `GET /api/v1/users` | Load users in platform-only or exact-school mode |
| `createUser` | `POST /api/v1/users` | Persist school user in invitation mode |
| `getUser` | `GET /api/v1/users/{userId}` | Load target in platform-only or exact-school mode |
| `createAccountInvitation` | `POST /api/v1/account-invitations` | Explicitly create invitation from persisted user |
| `completeAccountInvitation` | `POST /api/v1/account-invitations/{invitationToken}/setup` | Sole invited-to-active setup transition; guest regression scope |
| `getAccountLock` | `GET /api/v1/users/{userId}/account-lock` | Load authorized lock state |
| `lockAccount` | `POST /api/v1/users/{userId}/account-lock` | Lock with required reason |
| `unlockAccount` | `DELETE /api/v1/users/{userId}/account-lock` | Unlock with no body |
| `reactivateAccount` | `POST /api/v1/users/{userId}/account-reactivation` | Recover with `unlock` or reactivate |

`resendAccountInvitation` is not consumed because its path contains
`invitationToken`.

## User Creation Delta

`UserCreateRequest.account_setup_mode` is optional:

| Value | Result |
|---|---|
| omitted / `active` | Preserve existing active user creation |
| `invitation` | Persist `invited` user only; no invitation/token/delivery |

Response remains `201` success envelope containing `User`. `User.status` uses
`UserStatus`: `active`, `inactive`, or `invited`. Invalid mode is `422`.

## Invitation Contract

Invitation request uses persisted `scope`, `school_id`, `full_name`, `email`,
and `role_ids`; it omits `delivery_metadata` and `user_id`. Administrator UI
renders only `status`, `expires_at`, `delivery_channel`, and
`delivery_requested_at`; documented identifiers may remain service data but are
not rendered as delivery diagnostics. Failure does not roll back the persisted
invited user.

## Authorization and Tenant Contract

- Ordinary actor requires active `account_lifecycle.manage` matching platform or
  school target scope.
- Exact active platform System Administrator satisfies permission check only.
- Backend lifecycle authorization is enforced through `AccountLifecyclePolicy`;
  services invoke the policy before transition rules or protected state reads.
- School target requires active resolved same-school context and header.
- Platform target requires explicit platform lookup mode, platform authority,
  platform-only target query, and no school context/header. Lookup never retries
  automatically in school mode.
- Lookup mode is chosen before target fetch from validated route/list intent;
  otherwise active school selects school mode, otherwise platform authority
  selects platform mode, otherwise the frontend sends no target request.
- Existing `listUsers` and `getUser` preserve read authorization. With an exact
  school header they require school `users.view` and query only that school;
  without a school header they require platform `schools.view` or System
  Administrator master access and query only `school_id IS NULL` users. Matching
  `account_lifecycle.manage` or master access is still required before lifecycle
  sections mount. Unknown and opposite-mode identifiers share the non-disclosing
  result.
- General user permissions do not authorize account lifecycle.
- Unauthorized/missing/mismatched frontend views mount no lifecycle sections and
  send no lifecycle request.
- Backend authorizes tenant before scoped lookup; unknown/out-of-scope targets
  share non-disclosing result.
- Actor-equals-target lock review/lock/unlock/recover/reactivate is denied.

## Action Requests

- Lock: `{ reason }`, trimmed, 1–500 characters.
- Unlock: no body.
- Recover: `{ action: "unlock", reason? }`, reason at most 500.
- Reactivate: `{ action: "reactivate", reason? }`, reason at most 500.

Successful actions refresh both user and lock state. Invited users cannot be
activated through generic lifecycle actions.

## Frontend Component Contract

- Pages compose only; composables own effects and services own Axios.
- Panels receive readonly props and emit user intent.
- Raw scoped permission objects and exact active role drive visibility.
- Every request captures identity/permission/target/school generation.
- Context change aborts/invalidates result and follow-up refresh.
- Create-flow route intent contains only the persisted-user UUID; navigation or
  reload restores invitation state only after an authorized tenant-scoped
  `getUser` re-fetch confirms the same invited target.
- FR-026/SC-011 responsive and keyboard/dialog behavior is automated at 390,
  768, and 1440 pixels across Chromium, Firefox, and WebKit.
- No reusable token, permission payload, tenant-private error data, or submitted
  plaintext reason is included in post-submit feedback, diagnostics, logs, or
  persistence. A reason may exist only in the active input and documented
  outgoing action request.

## Compatibility and Exclusions

Default user creation is preserved. Frontend invitation mode deploys only after
contract/backend. No new endpoint, package, global store, automatic invitation,
admin resend, platform user creation expansion, or soft-delete restoration is
approved here.

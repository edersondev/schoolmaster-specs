# Contract Boundary: Duplicate Email Recovery

## Affected operations

| Operation | Contract change |
|---|---|
| `createUser` | Canonical email rules, exact generic 422 example, and new authorized recoverable 409 |
| `createAccountInvitation` | Same rules only when platform invitation creation would provision a missing platform user; preserve existing invitation conflict variants |
| `updateUser` | Canonicalize future email updates and preserve global retained ownership; existing response statuses remain |
| `restoreUser` | No shape/status change; remains explicit recommended recovery operation |

No endpoint or API version is added.

## Canonical identity email

For affected create/update inputs, server trims surrounding whitespace and
lowercases the complete email before validation, ownership comparison, and
storage. Retained lookup includes soft-deleted users and compares legacy stored
values by their trimmed, lowercase form. Existing rows are not bulk-updated.

## Direct create recoverable conflict

`POST /api/v1/users` adds:

```yaml
'409':
  $ref: ../../components/responses/users/RecoverableUserConflict.yaml
```

Canonical body:

```json
{
  "error": {
    "code": "recoverable_user_conflict",
    "message": "A retained user can be restored.",
    "details": {
      "user_id": "00000000-0000-4000-8000-000000000001",
      "recommended_action": "restore"
    }
  }
}
```

The response is permitted only when all conditions hold:

- matching user is soft-deleted
- matching user belongs to exact preselected request scope
- school mode uses exact active school context and actor has `users.lifecycle`
- platform mode actor has `schools.manage`
- requester already passed operation authentication, tenant, and creation
  authorization

These conditions authorize recovery disclosure only. The response does not
pre-approve restoration, and the explicit restore operation may still reject
current dependency, uniqueness, lifecycle, reason, or effective-date state.

`details` contains exactly `user_id` and `recommended_action`. Response contains
no email, name, school, scope, status, deletion timestamp, role, profile, audit
data, restore URL, or internal numeric identifier.

## Generic unavailable-email validation

All collisions that are inaccessible, unauthorized, hidden, not eligible for
recovery disclosure, or detected only at persistence time return:

```json
{
  "error": {
    "code": "validation_failed",
    "message": "Validation failed.",
    "details": {
      "fields": {
        "email": [
          "The email is unavailable."
        ]
      }
    }
  }
}
```

Status is `422`. Body shape and text are identical when matching identity is:

- active, inactive, or invited
- soft-deleted but outside exact request scope
- soft-deleted but requester lacks effective restore permission
- platform-owned during school mode or school-owned during platform mode
- detected only by final `users_email_unique` persistence failure

No matching identifier or lifecycle fact appears in the generic response.

## Account invitation conflicts

`POST /api/v1/account-invitations` already publishes status 409 for account
lifecycle conflicts. Its endpoint-specific 409 component must retain the
existing generic conflict example and add the recovery example above for the
platform-provisioning branch only.

```yaml
'409':
  $ref: ../../components/responses/account-lifecycle/AccountInvitationCreationConflict.yaml
```

School invitation creation requires an already persisted eligible invited user
and does not become a user-provisioning operation through this feature.
Invitation duplicate rejection persists no new user, role assignment,
invitation, token/setup credential, delivery request, or email submission.

## User update behavior

`PATCH /api/v1/users/{userId}` keeps existing statuses and envelopes. When email
is present, server canonicalizes it before validation and persistence. Matching
another retained identity returns the same generic 422 email field result; the
current target user is excluded from its own uniqueness decision. Existing
mixed-case values remain unchanged when email is omitted.

## Restore and follow-up flow

Authorized client receiving `recoverable_user_conflict` uses existing flow:

1. `POST /api/v1/users/{userId}/restore` with existing lifecycle action body.
2. If needed, use existing activation/deactivation lifecycle operation.
3. `PATCH /api/v1/users/{userId}` for permitted profile, role, or email changes.

Create request never performs any of these actions automatically. Existing
restore state, effective-date, reason, dependency, parent-state, authorization,
tenant, and history rules remain authoritative.

## Atomicity and concurrency

- Friendly retained-owner lookup does not replace database uniqueness.
- `users_email_unique` remains final authority.
- Any matching index failure rolls back full user/role/invitation transaction.
- Race loser receives generic 422, never recoverable 409.
- Duplicate feature audit is written only outside the rolled-back transaction.

## Audit boundary

Duplicate audit is backend-only and never added to response. Audit contains
actor, resolved school/platform scope, workflow, outcome, source IP, and
SHA-256 of canonical email. Target UUID is recorded only for an authorized
recoverable conflict. Plaintext email and hidden target information are absent.

## Compatibility

- `createUser` adds 409 for a scenario already rejected today; successful 201
  and generic 422 shapes remain.
- `createAccountInvitation` keeps 409 status but adds
  `recoverable_user_conflict` beside existing `conflict` codes. Clients that
  branch on error code must accept both.
- `updateUser` and `restoreUser` operation IDs/status sets remain unchanged.
- No frontend release, data migration, bulk rewrite, or API version bump is
  required.

# Data Model: Complete Administrator Account Lifecycle UI

No new user table or frontend persistence is introduced. One permission index
changes; remaining entities are existing records or transient UI state.

## User Account Setup

| Field | Type | Rules |
|---|---|---|
| `id` | UUID | Existing public user identifier |
| `schoolId` | UUID/null | School create flow requires resolved matching school |
| `fullName` | string | Persisted before invitation |
| `email` | email | Unique under existing user rule |
| `roleIds` | UUID[] | Active, same-scope roles; at least one |
| `accountSetupMode` | enum | `active` default or `invitation` |
| `status` | enum | `active`, `inactive`, `invited` |

Invitation mode creates `status=invited` without invitation/token/delivery.
Only successful invitation setup changes invited to active. Generic update,
activation, recovery, and reactivation reject invited users.

## Scoped Permission

| Field | Type | Rules |
|---|---|---|
| `id` | internal ID + UUID | Existing role relationship target |
| `code` | string | `account_lifecycle.manage` |
| `scope` | enum | `platform` or `school` |
| `status` | enum | Must be active |

Identity invariant: `(code, scope)` is unique. Platform and school records may
coexist; role scope and permission scope must match. System Administrator may
satisfy permission checks without a permission row, but not other gates.

## Lifecycle Authority Snapshot

| Field | Type | Rules |
|---|---|---|
| `actorId` | UUID | Must differ from target for lock/recovery flows |
| `systemAdministrator` | boolean | Derived from exact active platform role |
| `permissions` | scoped permission[] | Active raw session permissions only |
| `targetId` | UUID | Persisted target |
| `targetScope` | enum | Derived from target school ownership |
| `targetSchoolId` | UUID/null | Required only for school target |
| `activeSchoolId` | UUID/null | Must equal school target when required |
| `authorized` | boolean | Derived; never client authority for backend |

## Invitation Flow State

```text
editing
  -> creating
     -> create-error
     -> persisted-invited
        -> inviting
           -> invited
           -> invitation-error -> retry same persisted user
```

State contains persisted target, pending/error state, request context generation,
and administrator-visible invitation `status`, `expiresAt`, `deliveryChannel`,
and `deliveryRequestedAt`. It contains no lifecycle token, request
`delivery_metadata`, delivery secret, or other delivery diagnostic.

## Account Lock View and Action Draft

Existing lock fields remain: `id`, `userId`, nullable `schoolId`, `lockType`,
`status`, nullable `reason`, `lockedAt`, and `clearedAt`. Action draft contains
target, action, reason, pending/errors, eligibility, and context generation.

Action eligibility after lock load:

| Target/lock state | Actions |
|---|---|
| Active + no administrative lock | Lock |
| Active + administrative lock | Unlock, Recover |
| Inactive + no administrative lock + valid dependencies | Reactivate |
| Invited, soft-deleted, self-target, unauthorized, invalid tenant | None |

## Lifecycle UI State

```text
hidden -> loading-lock -> ready -> confirming -> submitting -> refreshing -> ready
   ^          |              |             |             |
   +----------+--------------+-------------+-------------+
       identity/permission/target/school change: abort, clear, re-evaluate
```

Conflict triggers safe feedback plus authoritative target/lock refresh. Stale
responses never commit and never start a follow-up request.

## Tenancy and Lookup Ordering

For school operations: authenticate actor → resolve active tenant → verify actor
scope → query target within school → deny self/action/state conflicts → mutate.
An unknown UUID and an existing UUID outside resolved school are indistinguishable.
For platform operations: verify platform authority → select platform lookup mode
→ query only platform-owned users with no school header → deny self/action/state
conflicts → mutate. Lookup mode is selected before the target request from a
validated route/list intent, otherwise active school context selects school mode,
otherwise platform authority selects platform mode, otherwise no request is sent.
The selected mode never falls back between platform and school.

## Migration Impact

- Drop `permissions_code_unique`.
- Add `permissions_code_scope_unique` on `(code, scope)`.
- Seed both scoped account lifecycle records using `(code, scope)` natural key.
- No users/status column migration; status is already an indexed string.
- Rollback must reconcile duplicate codes before restoring code-only uniqueness;
  otherwise migration is intentionally forward-only.
- Verification uses the configured MySQL test database, runs the seeder twice,
  proves duplicate composite rejection, and exercises the rollback guard.

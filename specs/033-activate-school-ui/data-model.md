# Data Model: School Context Selection UI

No database migration or new persisted backend entity is introduced. This
feature defines frontend view/session models over existing School,
authenticated-session, and lifecycle contracts.

## Entity: School Selection Choice

An active platform school presented as a possible context. Listing is not
authorization proof.

| Field | Type | Rules |
|---|---|---|
| `id` | UUID string | Required; maps from School `id` |
| `name` | string | Required display identity |
| `inepCode` | string | Required selector identity; maps from `inep_code` |
| `city` | string/null | Maps from `address.city`; render safe fallback if absent |
| `state` | string/null | Maps from `address.state`; render safe fallback if absent |
| `status` | `active`/`inactive` | Normalize numeric or string API status; only `active` is selectable |

`document`, CNPJ, email, and broader institutional fields are deliberately not
part of the selector model.

## Entity: School Selection Query

Transient server query owned by the selection composable.

| Field | Type | Default | Validation/behavior |
|---|---|---|---|
| `name` | string/null | null | Trimmed; backend contains match |
| `inepCode` | string/null | null | Normalized exact match; sent as `inep_code` |
| `status` | integer | `1` | Fixed active-only filter; not user-editable |
| `page` | integer | `1` | Minimum 1; reset to 1 when filters change |
| `perPage` | integer | `25` | Sent explicitly; maximum 100 |

Name and INEP filters combine with AND when both are provided.

## Entity: School Selection Result Page

Transient result and request state.

| Field | Type | Rules |
|---|---|---|
| `items` | School Selection Choice[] | Only normalized active choices are actionable |
| `page` | integer | From response meta |
| `perPage` | integer | From response meta |
| `total` | integer | From response meta |
| `pageCount` | integer | Derived as `ceil(total / perPage)` |
| `requestGeneration` | integer | Monotonic; older completions cannot commit |
| `status` | enum | `idle`, `loading`, `ready`, `empty`, `error` |
| `feedback` | object/null | Contract-safe localized error/empty state |

## Entity: Active School Context

One backend-confirmed school governing school-owned frontend work.

| Field | Type | Rules |
|---|---|---|
| `school` | School Selection Choice/null | Set only from confirmed `resolved_school` |
| `confirmationGeneration` | integer | Monotonic across restore/switch requests |
| `status` | enum | `unresolved`, `selecting`, `confirmed`, `blocked` |
| `source` | enum | `bootstrap`, `missing-context-page`, `shell-switch` |
| `invalidationReason` | enum/null | `bootstrap`, `tenant-mismatch`, `inactive-school`, `deactivated`, or `deleted` |

### Confirmation invariant

Context becomes `confirmed` only when:

1. `GET /api/v1/auth/me` succeeds with the selected UUID in `X-School-Id`;
2. `data.resolved_school` is present and non-null;
3. its `id` exactly matches the requested UUID;
4. normalized status is `active`; and
5. its request generation is still current.

School-owned requests remain blocked in every other state.

## Entity: Last-Confirmed School Preference

Non-authoritative browser session metadata.

| Field | Type | Rules |
|---|---|---|
| `schoolId` | UUID string | Written only after confirmation |
| `identityId` | UUID string/in-memory association | Must match current authenticated identity before restore |

The persisted school identifier is cleared with identity state on logout,
expiry, token rejection, lifecycle session cleanup, or identity replacement.
It is never accepted as authorization and always triggers fresh confirmation.
It is also cleared when an authoritative bootstrap, response, or lifecycle
result proves that specific school invalid; authenticated identity remains when
the token itself is still valid.

## Entity: Requested Route Intent

Safe navigation intent captured before entering the selector.

| Field | Type | Rules |
|---|---|---|
| `name` | registered route name | Never restore raw/unregistered paths |
| `params` | object | Must be empty for cross-school retention |
| `query` | object | Cleared by default; allow only declared context-neutral keys |
| `origin` | enum | `missing-context` or `shell-switch` |
| `schoolContextSwitch` | enum | `retain` only when route explicitly declares it |

Before restoration, the router rechecks registration, release, authentication,
permissions, context requirement, and cross-school compatibility. Unsafe intent
resolves to the school dashboard and is consumed once.

## Entity: Lifecycle Availability

Existing School state that controls selector eligibility.

| State | Selector behavior | Current-context behavior |
|---|---|---|
| Active, not deleted | May appear and be confirmed | Valid after fresh server confirmation |
| Inactive | Excluded | Clear when confirmed inactive |
| Soft-deleted/deleted | Excluded | Clear when deletion succeeds/is confirmed |
| Unknown/unavailable | Not committed | Keep school-owned content blocked |

Activation/restore does not create Active School Context. It changes
eligibility; a deliberate refresh/search and confirmation remain required.

## State Transitions

### Initial selection or restoration

```text
unresolved -> selecting -> confirmed
                      \-> blocked (recoverable failure)
                      \-> unresolved + identity cleared (401/token rejection)
```

### Switching School A to School B

```text
confirmed A
  -> route-leave/discard guard
     -> cancelled: confirmed A, data unchanged
     -> approved: clear A context + A-owned data
        -> selecting B
           -> confirmed B + safe route restore/reload
           -> blocked + selector retry/new choice
```

### Lifecycle invalidation

```text
confirmed A
  -> matching authoritative tenant_mismatch/inactive_school response
     OR bootstrap rejection for stored A
     OR successful local deactivate/delete of A
        -> increment confirmation generation
        -> clear A context + A-owned data + stored A preference
        -> authenticated with unresolved school (identity preserved)
        -> selector required only on school-owned route
```

Invalidation is idempotent. A response without `X-School-Id`, with a header or
stamped generation for an older context, with a generic 401/403 status, or
after context is already cleared causes no context transition. Bootstrap
fallback retries once without the stored header; it never loops.

### Search

```text
idle/ready/error -> loading(generation N)
loading N -> ready/empty/error only if N remains current
loading N -> ignored when a newer generation exists
```

## Tenancy and Persistence

- School remains the tenant root; all school-owned records continue using the
  existing `school_id` column and backend tenant scope.
- Selector discovery is platform-wide only for exact System Administrator and
  does not combine school-owned output.
- Platform-wide routes remain platform-wide after selection.
- Context validity is event-driven through bootstrap, matching authoritative
  API responses, and local lifecycle success; no polling state exists.
- No School, User, token, role, lifecycle, or audit database shape changes.
- No new DTO is needed; backend correction reuses existing `TenantContext`.
- No Repository is needed; no new or complex data access is introduced.

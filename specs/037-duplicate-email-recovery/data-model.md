# Data Model: Duplicate Email Recovery

Feature 037 adds one generated lookup key and index to `users`. It preserves
existing email values and `audit_events` storage while defining canonical write,
retained ownership, collision classification, and audit rules.

## User Identity

**Purpose**: Represent one authenticated platform identity whose email remains
globally owned through all lifecycle states.

**Relevant persisted fields**:

- `id`: internal numeric identifier; never exposed as the recovery reference.
- `uuid`: public identifier returned only for authorized recoverable conflict.
- `school_id`: nullable; null means platform identity, otherwise exact school
  ownership.
- `full_name` and `name`: profile/display fields unchanged by collision handling.
- `email`: `varchar(255)`, non-null, globally unique under the existing
  case-insensitive MySQL collation.
- `identity_email_key`: stored generated value `LOWER(TRIM(email))`, indexed by
  non-unique `users_identity_email_key_index` for sargable retained-owner lookup;
  never accepted from API input or returned by resources.
- `status`: `active`, `inactive`, or `invited`.
- `deleted_at`: nullable soft-delete marker; a non-null value does not release
  email ownership.
- `created_at` and `updated_at`: existing timestamps.

**Relationships**:

- optionally belongs to one `School`
- belongs to many `Role` records
- may own student, guardian, invitation, token, and historical relationships
- may be the affected resource of audit events

**Validation and ownership rules**:

- New and updated email input is trimmed and lowercased before validation,
  comparison, and storage.
- Comparison is global and includes soft-deleted users.
- Legacy stored email is compared through indexed `identity_email_key` but is
  not rewritten until an authorized email update.
- Direct creation and platform invitation provisioning may not create a second
  identity for an owned email.
- Database index `users_email_unique` remains final atomic authority for
  concurrent canonical writes.
- Lookup reads at most two canonical matches. More than one legacy match is an
  ambiguous ownership state and always produces generic unavailable-email
  validation with no recovery target.

**Lifecycle behavior**:

```text
active | inactive | invited
              |
              | soft delete
              v
same status + deleted_at set + email still owned
              |
              | explicit authorized restore
              v
same status + deleted_at cleared
              |
              | existing update/lifecycle operations
              v
permitted status, role, and profile changes
```

Creation conflict never changes this state automatically.

## Identity Email

**Purpose**: Canonical value used by every affected ownership decision and
materialized by MySQL for indexed retained-owner lookup.

**Values**:

- `submitted`: request or internal workflow value; never written to duplicate
  audit metadata.
- `canonical`: surrounding whitespace removed and complete value lowercased.
- `hash`: SHA-256 of canonical value for restricted audit correlation.

**Rules**:

- Canonical value is computed before email-format and length validation.
- Canonical value is the only value used for new/update persistence.
- Hash is pseudonymous audit metadata, not a public identifier and not an API
  field.
- MySQL derives `identity_email_key` for existing rows during the schema change;
  no stored `email` value is backfilled or rewritten.

## Duplicate Email Decision

**Purpose**: Transient result of global retained-owner lookup after request mode
and authorization are resolved.

**Decision values**:

| Decision | Conditions | External result |
|---|---|---|
| `available` | No retained owner matches canonical email | Creation may continue |
| `recoverable_user_conflict` | During school-scoped direct creation, exactly one matching owner is soft-deleted in the exact active school and actor has `users.view` plus `users.manage` | `409`, public target UUID, `recommended_action=restore`; later restore is not pre-approved |
| `email_unavailable` | Matching owner is active, inactive, invited, cross-tenant, platform-owned, ambiguous, or otherwise not eligible for recovery disclosure | Generic `422`, no target |
| `persistence_conflict` | `users_email_unique` wins after precheck during concurrent persistence | Same generic `422`, no target |

**Authorization rules**:

- School recovery disclosure requires exact active school context plus
  `users.view` and `users.manage` for that school.
- Platform invitation provisioning never discloses a recovery target; its email
  collisions are generic until platform-user restoration is contracted.
- Creation permission alone does not authorize recovery disclosure.
- Lookup/classification never changes the matching user's state.

## Recoverable User Conflict

**Purpose**: Public error payload for an authorized recovery-disclosure path.
It recommends explicit restoration but does not guarantee that current restore
constraints will pass.

**Fields**:

- `code`: constant `recoverable_user_conflict`
- `message`: constant `A retained user can be restored.`
- `user_id`: matching user's public UUID
- `recommended_action`: constant `restore`

No email, full name, school, tenant, scope, status, deletion timestamp, role,
profile, audit data, or restore URL is included.

## Generic Duplicate Validation

**Purpose**: Non-disclosing public error for every collision that is not eligible
for recovery disclosure.

**Fields**:

- `code`: constant `validation_failed`
- `message`: constant `Validation failed.`
- `fields.email`: one-element message array containing
  `The email is unavailable.`

Payload is identical for active, inactive, invited, hidden soft-deleted,
cross-tenant, platform-provisioning, ambiguous-legacy, unauthorized, and
persistence-race cases.

## Duplicate Attempt Audit

**Purpose**: Use existing audit storage to record every rejected duplicate user
creation without retaining plaintext email or unauthorized target identity.

**Existing persisted fields used**:

- `event_type`: `user_creation_duplicate_email`
- `actor_user_id`: authenticated administrator's internal identifier
- `school_id`: resolved school or null for platform mode
- `affected_resource_type`: `user` only for authorized recoverable conflict;
  otherwise null
- `affected_resource_id`: public target UUID only for authorized recoverable
  conflict; otherwise null
- `outcome`: `recoverable_user_conflict` or `validation_failed`
- `source_ip`: request source IP when available
- `tenant_safe_metadata`: strict allowlist described below
- `occurred_at`: rejection timestamp

**Allowlisted metadata**:

- `scope`: `school` or `platform`
- `workflow`: `direct_user_creation` or `account_invitation`
- `email_hash`: SHA-256 of canonical email
- `reason_code`: `recoverable_user_conflict`, `email_unavailable`, or
  `persistence_conflict`

**Rules**:

- Exactly one feature audit is written per rejected creation attempt.
- Precheck rejection is audited immediately before its domain failure, outside
  any doomed write transaction.
- Persistence conflict is audited only after full transaction rollback.
- Metadata never includes plaintext/canonical email, name, tenant identifier,
  target state, role, profile, invitation, credential, or delivery data.
- Audit storage and access use existing retention and authorization rules.

## Atomic Creation Boundaries

### Direct user creation

```text
authorize tenant/create -> canonicalize -> classify retained owner
  -> recoverable/generic audit + reject
  OR
transaction(user + role assignments)
  -> commit success
  OR users_email_unique rollback -> generic audit + 422
```

### Platform invitation provisioning

```text
authorize platform invitation -> canonicalize -> classify retained owner
  -> generic audit + reject for every collision
  OR
transaction(platform user + roles + invitation state)
  -> existing email-delivery lifecycle continues
  OR users_email_unique rollback -> generic audit + 422
```

School invitation creation does not provision a missing user and retains its
existing target eligibility behavior.

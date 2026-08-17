# Data Model: Invitation Email Delivery

No migration is required. Feature changes transition timing and interpretation
of existing invitation delivery fields.

## AccountInvitation

**Purpose**: Store secret-safe invitation lifecycle state.

**Relevant fields**:

- `uuid`: public invitation identifier; never used as setup proof.
- `target_user_id`: invited recipient.
- `school_id`: nullable school tenant boundary.
- `scope`: `platform` or `school`.
- `token_hash`: one-way hash; plaintext token is never persisted.
- `status`: `pending`, `completed`, `expired`, `superseded`, or `revoked`.
- `expires_at`: seven days after issuance.
- `delivery_channel`: `email` only after accepted mail submission; otherwise null.
- `delivery_requested_at`: acceptance timestamp; otherwise null.
- `email_delivery_metadata_summary`: safe purpose/recipient-domain summary only;
  no token, URL, password, transport credential, or full sensitive payload.

**Relationships**:

- belongs to one target `User`
- optionally belongs to one `School`
- belongs to one administrator actor

**State transitions**:

```text
create hash -> pending / delivery unset
pending -> pending / delivery accepted
pending delivery failure -> pending / delivery unset
retry -> old pending becomes superseded; new pending invitation created
valid setup -> completed; target user invited -> active
expiry/failure threshold -> expired or revoked
```

## InvitationEmail

**Purpose**: Transient transactional message submitted to configured mail
transport.

**Transient values**:

- target email and display name
- absolute setup URL containing plaintext token
- expiry timestamp
- product name

InvitationEmail is not a database entity. Its plaintext URL must not enter queue
storage, database columns, logs, audits, or API payloads.

## InvitedUser

Existing `User` with `status=invited`. Current email at invitation creation is
the sole recipient. Only successful setup changes status to `active`.

## Delivery Failure

No new persisted entity. Secret-free audit records identify invitation delivery
failure, actor, target, scope, timestamp, and source IP using existing audit
rules. External error response is `503 temporary_unavailable` with no provider
details.

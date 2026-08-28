# Data Model: Secure User Password Delivery

## Credential Delivery Request

| Field | Rules |
|---|---|
| `user_id` | Eligible active target UUID |
| `school_id` | Matching school UUID, or null for platform scope |
| `scope` | `school` or `platform` |
| `delivery_channel` | Always `email` |
| `delivery_requested_at` | Set only after accepted mail handoff |
| `status` | `requested` only in safe external result |

No plaintext token, link, or password is stored in or exposed by this record.

## Credential Link

Server-side token hash tied to one user. It is single-use, expiring, and moves
from active to consumed, superseded, revoked, or expired. A newer accepted
delivery supersedes older active links; mail failure leaves no newly usable link.
Existing reset-token failure suppression blocks new delivery token creation for
15 minutes after five failed reset-token completions per account or IP within
15 minutes.

## Eligibility and completion

Target must be active, unlocked, not invited, not soft deleted, and in the
authorized scope. School tenant validation precedes target lookup. Existing
password-reset completion applies password validation and atomically revokes
active sessions; it never unlocks an account.

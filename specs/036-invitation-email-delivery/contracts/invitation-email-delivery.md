# Contract Boundary: Invitation Email Delivery

## Affected operations

| Operation | Change |
|---|---|
| `createAccountInvitation` | `201` now guarantees configured mail transport accepted one setup email submission; adds documented `503 temporary_unavailable` |
| `completeAccountInvitation` | Token moves from the path into `invitation_token` in the JSON body at `POST /api/v1/account-invitations/setup` |
| `createUser` | No API default change; frontend explicitly submits `invitation` for fresh forms |

## `createAccountInvitation` success

- Existing `201` success envelope and `AccountInvitation` resource remain.
- `delivery_channel=email` and non-null `delivery_requested_at` mean transport
  submission acceptance, not inbox delivery.
- Response never contains invitation token, setup URL, mail body, provider
  response, or transport credential.

## `createAccountInvitation` temporary failure

```yaml
'503':
  description: Invitation email submission is temporarily unavailable
  content:
    application/json:
      schema:
        $ref: ../../components/schemas/common/ErrorEnvelope.yaml
```

Canonical body:

```json
{
  "error": {
    "code": "temporary_unavailable",
    "message": "Invitation email could not be submitted. Try again.",
    "details": {}
  }
}
```

Failure exposes no provider name, exception, recipient, token, setup URL, school,
role, or hidden target detail. Authorized retry uses existing invitation
creation and supersession rules.

## Setup link contract

```text
{trustedFrontendOrigin}/auth/account-invitations/setup#token={urlEncodedInvitationToken}
```

- Absolute HTTP(S) URL only.
- The fragment is copied to memory and removed from browser history during SPA
  initialization; frontend infrastructure never receives it in the HTTP target.
- Completion submits `{ invitation_token, password }` to the secret-free
  `/api/v1/account-invitations/setup` API path.
- No school, scope, actor, user, role, email, or password query data.
- Opening link does not consume it; successful password submission does.
- Existing seven-day expiry, single-use, supersession, revocation, and failed
  completion rules remain.

## Compatibility

- `completeAccountInvitation` preserves its operation ID, success/error
  behavior, and password policy, but replaces the pre-merge secret-bearing path
  with `/api/v1/account-invitations/setup` and requires
  `invitation_token` in the JSON body. Backend and frontend deploy together.
- Existing API clients omitting `account_setup_mode` retain active creation.
- Frontend exposes no account-setup selector and always sends
  `account_setup_mode=invitation`, even if stale form input contains another
  value.
- Password reset email, admin token-path resend redesign, provider event
  tracking, SMS, and bulk delivery remain out of scope.

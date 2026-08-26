# Contract: User Password Delivery

## New operation

`POST /api/v1/users/{userId}/password-delivery`

**Operation ID**: `requestUserPasswordDelivery`

Authenticated scoped `account_lifecycle.manage` authority is required; school
targets require existing `X-School-Id`. No request body. Server derives purpose
and email channel.

### 201 response

```json
{"data":{"status":"requested","delivery_channel":"email","delivery_requested_at":"2026-08-26T12:00:00Z"}}
```

No token, URL, password, email address, provider result, or private target
detail may appear.

| Status | Rule |
|---|---|
| 401 | Missing or invalid authentication |
| 403 | Missing, inactive, mismatched, or unauthorized `X-School-Id` tenant context returns `tenant_mismatch`; valid tenant context with insufficient authority returns `forbidden` |
| 404 | Existing safe lookup behavior |
| 409 | Inactive, invited, locked, deleted, or state-conflict target; no link |
| 422 | Request validation failure; no private detail |
| 429 | More than 3 deliveries per user/scope/24h; no link |
| 503 | Mail unavailable/rejected; no usable new link; retry allowed |

## Completion and unchanged behavior

Reuse `POST /api/v1/auth/password-resets` (`completePasswordReset`) for
single-use token completion and session revocation. Existing reset-token
failure suppression also applies before administrator delivery issues a new
token: five failed reset-token completions per account or IP within 15 minutes
block new reset-token creation for 15 minutes. `createUser` remains active by
default with no automatic mail; invitation and public reset request behavior
remain unchanged.

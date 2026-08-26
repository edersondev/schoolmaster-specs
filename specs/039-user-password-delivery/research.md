# Research: Secure User Password Delivery

## Decisions

### Explicit user-scoped delivery

Add authenticated `requestUserPasswordDelivery`; do not change active user
creation. This preserves active-creation compatibility and avoids automatic
credential mail.

### Reuse reset completion

The email link uses existing `completePasswordReset`. It is the one guest
completion path for first-password setup or reset, selected internally from
credential state. A separate setup endpoint and administrator-selected mode are
rejected because they duplicate secret handling or expose needless state.

### Accepted mail handoff is success

Success requires email transport acceptance. Rejection/unavailability returns a
safe temporary failure and creates no usable new link; a deliberate retry is
allowed. This prevents an inaccessible credential link.

### Eligibility and privacy

Require authorized scope, active matching tenant context, active/unlocked target,
and no more than 3 requests/user/scope/24h. Response contains only
`status=requested`, `delivery_channel=email`, and `delivery_requested_at`.
Never expose token, URL, password, email, or provider diagnostics.

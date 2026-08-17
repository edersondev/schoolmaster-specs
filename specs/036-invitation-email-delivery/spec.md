# Feature Specification: Invitation Email Delivery

**Feature Branch**: `036-invitation-email-delivery`
**Created**: 2026-08-15
**Status**: Draft
**Input**: User description: "Complete the production account setup flow so an administrator creates an invitation, the invited user receives a secure setup link by email, chooses a password, and can then log in. Make invitation the default account setup choice for new users."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Receive and complete account invitation (Priority: P1)

An invited user receives an account setup email containing a secure link, uses
that link to choose an initial password, and then signs in with the activated
account.

**Why this priority**: Invitation records and password setup already exist, but
users cannot reach setup because no invitation email is sent. Completing this
journey removes the production onboarding blocker.

**Independent Test**: Create an invitation for an eligible persisted user,
receive the email at that user's address, open its setup link, choose a valid
password, and verify normal sign-in succeeds while the link cannot be reused.

**Acceptance Scenarios**:

1. **Given** an eligible invited user and authorized same-scope administrator, **When** the administrator creates an invitation, **Then** one invitation email is submitted to the configured email service for the invited user's validated email address.
2. **Given** a newly created invitation, **When** the user opens the email, **Then** it identifies SchoolMaster, explains that account setup is required, states the invitation expiry, and provides one secure setup link for that invitation.
3. **Given** a valid unexpired setup link, **When** the user submits a policy-compliant password, **Then** setup activates the invited account, consumes the invitation, and directs the user to sign in without automatically creating an authenticated session.
4. **Given** an expired, used, superseded, revoked, malformed, or scope-incompatible link, **When** the user attempts setup, **Then** no credential or account state changes and existing safe invalid-link feedback is shown.

---

### User Story 2 - Default new users to invitation setup (Priority: P1)

An administrator opening the new-user form sees invitation setup selected by
default, while retaining an explicit active-account option for the exceptional
legacy workflow.

**Why this priority**: Email delivery provides little operational value if new
users continue to default to immediately active accounts with system-generated
credentials.

**Independent Test**: Open a fresh new-user form, verify invitation setup is
selected, create the user, and verify the account remains invited until the
administrator explicitly creates the invitation and the user completes setup.

**Acceptance Scenarios**:

1. **Given** an administrator opens a fresh new-user form, **When** no prior form state exists, **Then** invitation setup is selected by default.
2. **Given** invitation setup remains selected, **When** valid user creation succeeds, **Then** one invited user is persisted without an active password, invitation, token, or delivery request, and the explicit create-invitation step is shown.
3. **Given** the administrator explicitly selects active setup, **When** valid user creation succeeds, **Then** existing active-user creation behavior remains available and no invitation is created.
4. **Given** an API client omits the account setup choice, **When** it creates a user, **Then** the existing active-account default remains unchanged for backward compatibility.

---

### User Story 3 - Recover safely from delivery failure (Priority: P2)

An administrator receives accurate feedback when the email service cannot
accept an invitation message and can retry without exposing or reusing the
invitation secret.

**Why this priority**: Production email can fail. False delivery confirmation
would leave users blocked while administrators believe setup instructions were
sent.

**Independent Test**: Make email submission fail, create an invitation, verify
delivery is not reported as accepted and no secret is exposed, restore email
service, retry invitation creation, and verify the replacement link works while
the earlier invitation cannot be completed.

**Acceptance Scenarios**:

1. **Given** the email service rejects or cannot accept the invitation message, **When** invitation creation is attempted, **Then** the system does not report delivery as accepted and returns documented retryable feedback to the administrator.
2. **Given** a delivery attempt failed after invitation state was created, **When** an authorized administrator retries creation for the same user and scope, **Then** a new invitation and email replace any prior pending invitation without exposing either token.
3. **Given** email submission succeeds, **When** the invitation response is returned, **Then** safe delivery-request metadata is present and no reusable invitation token or setup URL appears in the API response, application log, audit event, or delivery metadata.

### Edge Cases

- Email configuration is missing, invalid, or unavailable when invitation creation occurs.
- User email changes between invited-user creation and invitation creation; delivery uses the validated current target address.
- Administrator submits invitation creation twice; send limits and token supersession still apply, and every accepted message references only its own invitation.
- Invitation email arrives after a newer invitation supersedes it; the older link fails safely.
- Setup link is copied into another browser or opened by an email security scanner; viewing the page does not consume the token.
- Public application origin is missing or unsafe; no relative, internal, or attacker-controlled setup URL is sent.
- User-provided name or school text contains markup; email content renders it as text and does not allow injected links or markup.
- Email service accepts a message but later cannot deliver it; provider-specific bounce, complaint, and delivery-event tracking remain outside this feature.
- Invitation expires before the user completes setup; existing expiry behavior remains authoritative.
- Account is inactive, deleted, cross-tenant, or otherwise ineligible when delivery or setup is attempted; existing lifecycle and tenant rules remain enforced.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Deliver invitation setup emails through the configured mail capability, build the public setup URL from trusted configuration, keep token handling transient and secret-safe, distinguish accepted delivery requests from delivery failures, and add focused mail, failure, security, and account-setup tests.
- **Frontend repository impact**: Change fresh create-user form state so invitation is the selected setup mode, preserve explicit active setup, and update unit and end-to-end coverage for the default and full emailed-link journey.
- **Specification or contract repository impact**: Update Feature 008 and Feature 034 rules that currently limit invitation behavior to delivery metadata or describe active setup as the operational default; document email handoff and retryable failure behavior in OpenAPI without exposing a token or setup URL.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines behavior first; `schoolmaster-backend` implements secure email submission next; `schoolmaster-frontend` changes the administrator default and validates the existing setup route after backend behavior is verified.

### API Contract Impact

- **OpenAPI update required**: Yes. Existing invitation creation and response documentation must state real email submission semantics, secret-safe response behavior, and the documented temporary delivery-failure outcome. User creation retains its current API default.
- **Versioned endpoints affected**: `POST /api/v1/account-invitations`, `POST /api/v1/account-invitations/{invitationToken}/setup`, and `POST /api/v1/users` documentation; no new endpoint is added.
- **JSON response impact**: Successful invitation responses keep the existing envelope and safe invitation fields. No token or setup URL is added. Email-submission failure uses the approved standard temporary-failure envelope and does not claim accepted delivery metadata.
- **Authentication/authorization impact**: No new permission or public access path. Invitation creation retains existing same-scope account-lifecycle authority and tenant-first lookup. Password setup remains token-proven and unauthenticated.
- **Compatibility impact**: Email submission adds a real side effect to invitation creation. Fresh frontend forms change default choice to invitation. API clients omitting `account_setup_mode` retain active creation, avoiding an implicit breaking change.

### Data & Tenancy Impact

- **Tenant scoping impact**: Existing platform-versus-school invitation scope, school context, role validation, target eligibility, and tenant-first lookup remain unchanged. Setup links contain only the opaque invitation token and public setup path, not tenant identifiers.
- **Cross-tenant or platform access impact**: Email delivery grants no cross-tenant access and does not weaken platform/school administrator separation.
- **Soft delete impact**: Deleted users remain ineligible for invitation delivery and setup. No restore behavior is added.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST submit exactly one account setup email for each successfully created invitation.
- **FR-002**: System MUST address the setup email only to the invitation target's validated current email address.
- **FR-003**: System MUST include a single absolute setup link built from a trusted public application origin and the newly issued invitation secret.
- **FR-004**: Setup email MUST identify the product, explain the required action, and communicate the invitation's expiration without including passwords or unrelated tenant-private data.
- **FR-005**: System MUST keep the plaintext invitation secret transient and MUST NOT persist it in application data, delivery metadata, queued payloads, logs, audit events, or API responses.
- **FR-006**: System MUST preserve existing invitation expiry, single-use, supersession, send-limit, failed-completion, tenant, authorization, and target-eligibility rules.
- **FR-007**: System MUST record delivery-request time only when the configured email service accepts the message submission.
- **FR-008**: System MUST return documented retryable feedback when email submission fails and MUST NOT report the invitation as delivered or expose its setup link.
- **FR-009**: System MUST allow an authorized administrator to create a replacement invitation after delivery failure, subject to existing send limits, and MUST invalidate prior pending invitations.
- **FR-010**: Successful password setup MUST remain the only operation that activates an invited user and MUST leave the user signed out until normal login.
- **FR-011**: Fresh administrator create-user forms MUST select invitation setup by default.
- **FR-012**: Administrators MUST retain an explicit active setup choice for supported exceptional use.
- **FR-013**: User creation in invitation mode MUST remain separate from invitation creation so persistence success and delivery failure remain distinguishable and retryable.
- **FR-014**: API user creation MUST retain `active` as the default when `account_setup_mode` is omitted; frontend requests MUST explicitly submit the selected invitation default.
- **FR-015**: System MUST define changed invitation delivery and temporary-failure behavior in OpenAPI before backend implementation begins.
- **FR-016**: System MUST preserve existing success envelopes and use the standard documented failure envelope without adding secret-bearing response fields.
- **FR-017**: System MUST preserve tenant isolation and MUST NOT encode school or platform authorization context into the setup URL.
- **FR-018**: Delivery implementation MUST use existing configurable email infrastructure without adding a provider-specific product dependency.
- **FR-019**: Tests MUST prove successful message submission, recipient and link correctness, failure handling, token non-persistence, frontend invitation default, setup completion, token reuse rejection, and successful subsequent login.

### Key Entities

- **Account Invitation**: Single-use, expiring proof that an eligible invited user may set an initial password; stores only a hash of its secret plus lifecycle and safe delivery-request metadata.
- **Invitation Email**: Transactional message submitted to the invited user's current email address; contains the only user-facing copy of the plaintext setup link.
- **Invited User**: Persisted inactive-for-login account awaiting successful invitation setup.
- **Delivery Request**: Safe record that the configured email service accepted message submission; it is not proof of inbox delivery.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In acceptance testing, 100% of successfully created invitations submit one email to the intended target and expose no token through API responses, stored metadata, logs, or audit records.
- **SC-002**: An invited user can move from received email to completed password setup and successful login in under 5 minutes, excluding email-provider transit time.
- **SC-003**: 100% of fresh administrator create-user forms select invitation setup until the administrator explicitly chooses active setup.
- **SC-004**: In failure testing, 100% of rejected email submissions produce retryable administrator feedback and never claim accepted delivery.
- **SC-005**: Expired, consumed, superseded, revoked, malformed, and reused invitation links produce zero unauthorized account activations.
- **SC-006**: Existing active-setup API clients that omit `account_setup_mode` continue producing active accounts without request changes.

## Assumptions

- Administrator-triggered invitation creation remains an explicit step after an invited user is persisted.
- Invitation email is transactional account email, independent of notification preferences.
- Configured email service acceptance means delivery was requested; inbox delivery, bounces, complaints, analytics, and provider webhooks are outside this feature.
- One existing public frontend invitation-setup route remains the canonical link target.
- English email content is sufficient for this slice; broader email localization and branding are separate work.
- Synchronous submission is acceptable for this security-sensitive token because durable background payloads must not contain the reusable plaintext invitation secret.
- Password reset email delivery, invitation resend redesign, SMS, bulk invitations, and automatic invitation creation during user persistence are outside scope.

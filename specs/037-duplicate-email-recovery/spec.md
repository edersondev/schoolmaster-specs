# Feature Specification: Duplicate Email Recovery

**Feature Branch**: `037-duplicate-email-recovery`  
**Created**: 2026-08-18  
**Status**: Draft  
**Input**: User description: "Improve duplicate-email handling when creating users. Email remains globally unique, including soft-deleted accounts, and duplicate identities must not be created. When an email belongs to a deleted user within administrator’s permitted scope, recommended flow is to restore existing user, then update status, roles, and profile as needed. For all other duplicate cases, return clear generic validation error without exposing deleted-account or cross-tenant details. Cover case-insensitive email matches and concurrent creation attempts."

## Clarifications

### Session 2026-08-18

- Q: What should a same-school recoverable deleted-user collision return? → A: Return `409 recoverable_user_conflict` with the matching user UUID and a `restore` recommendation.
- Q: How must identity emails be normalized? → A: Trim surrounding whitespace and lowercase the entire email before validation, comparison, and storage.
- Q: What should generic unavailable-email collisions return? → A: Return `422 validation_failed` with the field message `email: "The email is unavailable."`.
- Q: How should existing stored email values be normalized? → A: Normalize only new users and future email updates; do not bulk-update existing records.
- Q: Which duplicate-email creation outcomes require dedicated audit records? → A: Audit every duplicate outcome with actor, resolved scope, outcome, and normalized email hash; include the target UUID only for authorized recoverable conflicts.
- Q: Does `409 recoverable_user_conflict` guarantee that restoration will succeed? → A: No. It confirms only that recovery guidance may be disclosed because the retained user is soft-deleted in the exact active school and the requester has effective restore authorization; the explicit restore action re-evaluates all current lifecycle constraints and may still fail.
- Q: Which workflow may disclose the recoverable conflict? → A: Only school-scoped direct user creation when the requester has the established `users.view` and `users.manage` permissions; platform invitation provisioning remains generic because platform-user restoration is not contracted by this feature.
- Q: How is legacy-aware normalized ownership lookup kept indexable? → A: Add an indexed generated `identity_email_key` derived from `LOWER(TRIM(email))`; do not rewrite existing email values.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Restore Retained Identity (Priority: P1)

An authorized administrator who attempts to create a user with an email held by
a soft-deleted user in the administrator's permitted scope is directed to
restore the retained identity instead of creating a duplicate. After restore,
the administrator may update the user's status, roles, and profile through the
existing approved administration workflows.

**Why this priority**: Email identifies one platform identity. Restoring the
retained record preserves history, role and profile relationships, and audit
continuity while preventing two identities from representing the same person.

**Independent Test**: Soft-delete a user, submit the same email through an
authorized user-creation workflow, verify no user is created and safe recovery
guidance is returned, then restore the original user and update its permitted
fields.

**Acceptance Scenarios**:

1. **Given** a soft-deleted user owns the submitted email in the administrator's exact active school and the administrator has `users.view` plus `users.manage`, **When** the administrator attempts to create a user with that email, **Then** creation is rejected with `409 recoverable_user_conflict`, no duplicate identity is persisted, and the response contains only the matching user UUID and a `restore` recommendation.
2. **Given** the administrator received an eligible same-school recovery result, **When** they explicitly restore the retained user, **Then** the original identity and history are preserved and the user becomes available according to existing restore rules.
3. **Given** the retained user has outdated status, roles, or profile details after restoration, **When** the administrator submits permitted updates, **Then** those fields are updated through existing administration workflows without creating another identity.
4. **Given** the administrator received an eligible recovery result but a dependency, uniqueness condition, or lifecycle state change prevents restoration, **When** restoration is attempted, **Then** the existing restore workflow rejects the action without creating a replacement identity.

---

### User Story 2 - Reject Duplicates Without Disclosure (Priority: P2)

An administrator receives a clear, generic email validation result when the
submitted address belongs to an active, inactive, invited, inaccessible,
cross-tenant, or otherwise non-recovery-disclosure-eligible identity.

**Why this priority**: Administrators need a useful correction signal, but an
email collision must not reveal another user's lifecycle state, tenant, scope,
identifier, roles, profile, or existence beyond what the requester is already
permitted to know.

**Independent Test**: Attempt user creation against duplicate emails held by
several identities that are not eligible for recovery disclosure and verify
every result is generic, no identity details are disclosed, and no user is
created.

**Acceptance Scenarios**:

1. **Given** an active, inactive, or invited user already owns the submitted email, **When** another user is created with that email, **Then** creation fails with `422 validation_failed`, the `email` field says `The email is unavailable.`, and no duplicate is persisted.
2. **Given** a soft-deleted user with the submitted email exists outside the requester's permitted scope, **When** creation is attempted, **Then** the same `422` email field result is returned and no record identifier, tenant, scope, or lifecycle state is disclosed.
3. **Given** the requester lacks permission to restore a matching soft-deleted user, **When** creation is attempted, **Then** the system provides no recovery reference and returns only the same `422` email field result.

---

### User Story 3 - Preserve Uniqueness Under Equivalent and Concurrent Input (Priority: P3)

Administrators receive consistent results when email casing differs or when
multiple requests attempt to claim the same email at nearly the same time.

**Why this priority**: Friendly pre-checks alone cannot guarantee identity
uniqueness. Equivalent email forms and concurrent requests must not bypass the
same platform rule or surface internal failures.

**Independent Test**: Submit case variants and coordinated concurrent creation
attempts for one email, then verify at most one identity owns the address and
every rejected request receives a documented client-correctable result.

**Acceptance Scenarios**:

1. **Given** a user owns `joao@teste.com.br`, **When** creation is attempted with a casing variant of that address, **Then** the system treats both values as the same identity email and rejects the duplicate.
2. **Given** no user currently owns an email, **When** two permitted requests concurrently attempt to create users with equivalent versions of that email, **Then** at most one identity is created and every rejected request receives a documented conflict or validation result rather than an internal error.
3. **Given** a soft-deleted user owns an email, **When** concurrent creation requests submit equivalent versions of that email, **Then** all creation attempts are rejected and the retained identity remains the sole owner.

### Edge Cases

- A submitted email differs only by letter casing or surrounding whitespace; both forms resolve to the same trimmed, lowercase identity email.
- An existing mixed-case email remains stored unchanged until an authorized email update, but all ownership comparisons still treat its normalized form as the same identity email.
- The matching user is active, inactive, invited, or soft-deleted.
- The matching soft-deleted user belongs to the same school, another school, or platform scope.
- The requester may create users but may not restore the matching identity.
- The matching user is in scope, but its parent school is inactive or its restore dependencies are invalid.
- The matching user is restored, deleted again, or otherwise changes state between the creation rejection and the restore attempt.
- Another request claims the email between an availability check and persistence.
- Direct user creation and invitation-driven user provisioning share the same
  identity ownership, privacy, and atomicity rules; only school-scoped direct
  creation can disclose the contracted recovery guidance.
- A restoration would collide with another retained or active identity because of inconsistent legacy data.
- Recovery guidance must not automatically restore, reactivate, reassign roles, or change profile data.
- Repeated generic duplicate attempts remain non-disclosing to the requester but produce distinct privacy-preserving audit records for authorized review.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Align every workflow that can create a user with platform-wide, case-insensitive email uniqueness across active and soft-deleted records. Add an indexed generated canonical-email lookup key without rewriting email values, translate duplicate outcomes into documented responses, provide recovery guidance only for authorized same-school soft-deleted identities, record privacy-preserving duplicate-attempt audits, align school restore authorization with the established permission contract, and add regression and concurrency coverage.
- **Frontend repository impact**: No frontend delivery is required in this feature. Existing or future administration clients may consume the documented recoverable conflict and continue through existing restore and update workflows; they must not infer recovery eligibility from a generic duplicate result.
- **Specification or contract repository impact**: Clarify global identity ownership, same-school recovery eligibility, non-disclosing duplicate behavior, and concurrent outcomes. Update the aggregate OpenAPI descriptions and response contracts for affected creation operations before backend behavior changes.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines behavior and contract first; `schoolmaster-backend` implements and verifies it second. Optional frontend guidance follows only after the backend contract is published and verified.

### API Contract Impact

- **OpenAPI update required**: Yes. Document the recoverable same-school soft-deleted-user conflict separately from the generic unavailable-email validation result and define the minimum safe recovery information.
- **Versioned endpoints affected**: `POST /api/v1/users` and `POST /api/v1/account-invitations` when invitation creation would provision a missing platform user. Existing school-scoped `POST /api/v1/users/{userId}/restore` and `PATCH /api/v1/users/{userId}` remain the recommended follow-up operations.
- **JSON response impact**: Creation operations must preserve existing success envelopes. Duplicate active, inactive, invited, inaccessible, cross-tenant, platform-provisioning, and unauthorized recovery cases return `422 validation_failed` with `email: "The email is unavailable."`. An eligible same-school soft-deleted collision during direct user creation returns `409 recoverable_user_conflict` containing only the matching user UUID and a `restore` recommendation.
- **Authentication/authorization impact**: Existing creation permissions remain required. School recovery guidance is available only with exact active school context and the established `users.view` plus `users.manage` permissions used by the restore workflow. Authorization and tenant resolution occur before any recoverability disclosure. Platform invitation provisioning never discloses a recovery target in this feature.
- **Compatibility impact**: Additive response documentation for an existing rejected creation scenario. Successful creation, email ownership, restore, and update semantics remain unchanged; clients that already handle documented validation or conflict envelopes remain compatible.

### Data & Tenancy Impact

- **Tenant scoping impact**: Email ownership remains platform-wide while recovery guidance remains bound to the exact active school scope. A school-scoped request cannot discover or recover a user from another school or platform scope.
- **Cross-tenant or platform access impact**: Same-school recovery guidance is never a cross-tenant lookup path. Opposite-scope, cross-tenant, and platform invitation collisions remain indistinguishable from other unavailable-email results.
- **Schema impact**: Add an indexed generated `identity_email_key` derived from `LOWER(TRIM(email))` for sargable retained-owner lookup. Existing `users.email` values are not backfilled or rewritten, and `users_email_unique` remains the final concurrency authority.
- **Soft delete impact**: Soft deletion retains email ownership, identity history, relationships, and restore eligibility. Deletion does not release the email for a new identity, and no permanent deletion or anonymization behavior is added. Existing stored email values are not bulk-rewritten by this feature.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST treat a user's email as one globally unique platform identity key across active, inactive, invited, and soft-deleted users.
- **FR-002**: Every workflow capable of creating a user MUST trim surrounding whitespace and lowercase the entire email before validation, comparison, and storage.
- **FR-003**: Soft deletion MUST NOT release a user's email for assignment to a new identity.
- **FR-004**: Every user-creation workflow MUST check retained identities, including soft-deleted users, before reporting that an email is available.
- **FR-005**: A duplicate-email rejection MUST persist no new user, role assignment, invitation, setup credential, or partial related state.
- **FR-006**: During school-scoped direct user creation, when the matching identity is soft-deleted, belongs to the requester's exact preselected active school, and the requester has both `users.view` and `users.manage`, the system MUST reject creation with `409 recoverable_user_conflict`. This result authorizes recovery disclosure only and MUST NOT guarantee that the later restore action will pass its current lifecycle constraints. Platform invitation provisioning MUST NOT emit this result in this feature.
- **FR-007**: The `recoverable_user_conflict` response MUST contain only the matching user UUID and a `restore` recommendation and MUST NOT include additional profile, role, school, tenant, audit, or lifecycle details.
- **FR-008**: The system MUST NOT automatically restore, reactivate, update, or reassign roles to a matching soft-deleted user as part of a creation request.
- **FR-009**: Restoration MUST remain an explicit authorized lifecycle action and MUST continue to enforce existing reason, effective-date, tenant, parent-state, dependency, uniqueness, and history rules.
- **FR-010**: After restoration, status, roles, and profile changes MUST occur only through existing authorized update or lifecycle workflows.
- **FR-011**: Duplicate emails held by active, inactive, invited, inaccessible, opposite-scope, cross-tenant, or otherwise not eligible for recovery disclosure under FR-006 MUST return `422 validation_failed` with the field message `email: "The email is unavailable."`.
- **FR-012**: A generic duplicate result MUST NOT disclose whether a matching identity is active, inactive, invited, deleted, platform-owned, school-owned, cross-tenant, or otherwise present.
- **FR-013**: Scope and authorization decisions MUST complete before the system discloses whether a matching soft-deleted identity is recoverable by the requester.
- **FR-014**: Concurrent requests for equivalent emails MUST result in at most one newly created identity and MUST NOT surface an internal failure to rejected requesters.
- **FR-015**: If a duplicate is detected only at final persistence time, the system MUST return `422 validation_failed` with `email: "The email is unavailable."` and MUST NOT expose an internal failure.
- **FR-016**: Direct user creation and invitation-driven user provisioning MUST use the same global identity ownership, normalization, privacy, atomicity, and generic duplicate rules; platform invitation provisioning MUST return the generic result because platform-user restoration is not contracted by this feature.
- **FR-017**: The OpenAPI contract MUST document all changed response semantics before backend implementation begins.
- **FR-018**: Tests MUST cover exact duplicates, case variants, surrounding whitespace under documented normalization, every user lifecycle state, same-school recovery eligibility, unauthorized and cross-tenant non-disclosure, generic invitation-driven provisioning, and concurrent creation attempts.
- **FR-019**: This feature MUST NOT add permanent deletion, purge, anonymization, automatic restoration, cross-tenant recovery, a new authentication method, or direct per-user permission assignment.
- **FR-020**: The system MUST normalize emails for new users and future email updates, MUST compare existing emails using the same normalized ownership rule, and MUST NOT bulk-update existing stored email values in this feature.
- **FR-021**: Every duplicate-email creation outcome MUST record an audit event containing the actor, resolved scope, outcome, and normalized email hash without storing the plaintext submitted email; the matching target UUID MAY be recorded only when the requester was authorized to receive `recoverable_user_conflict`.
- **FR-022**: Retained-owner lookup MUST use an indexed canonical representation derived from the trimmed, lowercase stored email and MUST NOT require a full `users` table expression scan.

### Key Entities

- **User Identity**: The retained platform identity represented by one globally unique email, lifecycle state, optional school ownership, roles, profile relationships, and historical references.
- **Identity Email**: The trimmed, lowercase value that remains owned by the same user through active, inactive, invited, and soft-deleted states.
- **Recoverable Creation Conflict**: A `409 recoverable_user_conflict` outcome available only during school-scoped direct creation to a requester with `users.view` and `users.manage` for an eligible same-school soft-deleted user; it contains the matching user UUID and a `restore` recommendation for explicit restoration, but it does not pre-approve or guarantee the later restore action.
- **Generic Duplicate Validation**: The non-disclosing `422 validation_failed` outcome with `email: "The email is unavailable."`, used whenever recovery disclosure is not permitted under FR-006.
- **Restore Action**: The existing explicit lifecycle transition that returns a retained user to operational availability after authorization, state, dependency, uniqueness, and history checks pass.
- **Duplicate Attempt Audit**: A privacy-preserving record of actor, resolved scope, duplicate outcome, and normalized email hash; it includes a target UUID only for an authorized recoverable conflict.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In 100% of tested duplicate-email attempts across active, inactive, invited, and soft-deleted states, no second identity is created.
- **SC-002**: In 100% of tested recovery-disclosure-eligible soft-deleted collisions, an authorized administrator receives safe recovery guidance; when current restore constraints pass, the administrator can complete restore followed by permitted updates through existing workflows.
- **SC-003**: In 100% of tested cross-tenant, opposite-scope, inaccessible, and unauthorized recovery cases, responses reveal no user identifier, tenant, scope, lifecycle state, roles, or profile details.
- **SC-004**: In repeated concurrent tests for one equivalent email, at most one new identity is created and 100% of rejected requests receive documented client-correctable outcomes rather than internal errors.
- **SC-005**: Exact-match, case-variant, and surrounding-whitespace scenarios produce the same ownership decision in 100% of acceptance tests, and every newly created or updated email is stored in trimmed, lowercase form.
- **SC-006**: Reviewers can map 100% of changed creation and recovery outcomes to published contract responses before backend delivery is accepted.
- **SC-007**: In 100% of duplicate-email acceptance tests, one privacy-preserving audit event is recorded with no plaintext submitted email, and a target UUID appears only for authorized recoverable conflicts.

## Assumptions

- Email is the platform-wide identity key and is not reusable after soft deletion.
- Soft-deleted users and their historical relationships remain retained. Authorized recovery guidance does not bypass any existing lifecycle rule that may block restoration.
- Existing user restore and update operations remain the only approved way to recover and revise a deleted identity.
- A recoverable conflict may identify a same-school deleted user only after current tenant and authorization checks prove the requester has `users.view` and `users.manage` in that active school.
- Generic duplicate responses intentionally prioritize privacy over explaining why an email is unavailable.
- The existing administration lifecycle audit and reason requirements continue to apply to restore actions.
- Duplicate-attempt audit records use existing audit retention and access controls.
- Existing email values are not bulk-normalized; the generated lookup key derives their canonical comparison value without rewriting them, and they become trimmed and lowercase only when an authorized workflow changes them.
- No new deleted-user browser, permanent purge workflow, or frontend delivery is included; clients use the safe reference from the recoverable conflict with existing restore behavior.
- Specification and OpenAPI changes lead backend implementation; frontend consumption is optional follow-up work.

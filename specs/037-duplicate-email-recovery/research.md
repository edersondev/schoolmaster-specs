# Research: Duplicate Email Recovery

## Decision 1: Preserve the existing global database uniqueness rule

**Decision**: Keep the existing case-insensitive unique index on `users.email`
as the final authority across active and soft-deleted users. Do not add a
migration, partial uniqueness rule, generated column, or deleted-email rewrite.

**Rationale**: Product identity remains global and soft deletion intentionally
retains ownership. The current MySQL index already prevents two canonical new
writes from winning a race and includes soft-deleted rows.

**Alternatives considered**: Releasing email on soft delete was rejected because
it breaks identity/history continuity and makes restore ambiguous. A generated
active-only key and mutating deleted email were rejected because they implement
email reuse, contrary to the specification. A new normalized column was
rejected because no bulk rewrite is approved.

## Decision 2: Normalize at request and service boundaries without backfill

**Decision**: Trim surrounding whitespace and lowercase the complete email in
all affected Form Requests, then normalize defensively in the shared identity
service before comparison or persistence. Apply this to new users, platform
invitation provisioning, and future user email updates only.

**Rationale**: Laravel Form Requests support pre-validation normalization, while
the service boundary protects internal calls that do not originate from HTTP.
Avoiding a model-wide mutator keeps legacy fixtures and retained rows unchanged
until an authorized email update.

**Alternatives considered**: Request-only normalization was rejected because
internal service calls could bypass it. A model mutator was rejected because it
obscures when legacy values change. Bulk normalization was rejected by the
clarified scope.

## Decision 3: Use one legacy-aware ownership resolver

**Decision**: Add `IdentityEmailService` for global retained-owner lookup using
the canonical input against `LOWER(TRIM(email))`, including soft-deleted rows.
Use it from direct user creation, platform invitation provisioning, and user
email update validation. Retire the account-lifecycle-only email lookup so
ownership rules have one implementation.

**Rationale**: Existing records are not backfilled, so raw equality alone cannot
honor surrounding-whitespace normalization for legacy rows. One focused query is
simple enough that a new repository abstraction is not justified.

**Alternatives considered**: Separate queries in each service were rejected as
drift-prone. Raw equality was rejected because it misses legacy whitespace.
Adding a repository for one lookup was rejected as unnecessary indirection.

## Decision 4: Authorize recovery disclosure with effective restore rules

**Decision**: Resolve tenant/platform mode and creation authorization before
global owner classification. Extend the administration policy with a focused
recovery-disclosure decision that matches actual restore permissions:
`users.lifecycle` in the exact active school or `schools.manage` for a platform
user. Only a soft-deleted exact-scope target passing that decision may expose its
UUID.

This decision authorizes disclosure only. It does not pre-approve the explicit
restore action, which re-evaluates all lifecycle constraints against current
state and may still fail.

**Rationale**: The existing administration policy allows a lifecycle fallback
that differs from the individual restore service. Reusing that broad fallback
could tell a create-only administrator about a deleted identity they cannot
restore. A dedicated policy decision makes disclosure deny-by-default without
changing existing restore behavior.

**Alternatives considered**: Treating `users.manage` or
`account_lifecycle.manage` as restore authority was rejected because those
permissions do not authorize the follow-up operation. Performing global lookup
before tenant authorization was rejected as a cross-tenant disclosure risk.

## Decision 5: Publish specialized errors without new endpoints

**Decision**: Add a specialized direct-create 409 response with code
`recoverable_user_conflict`, message `A retained user can be restored.`, and
details containing only `user_id` plus `recommended_action=restore`. Generic
duplicates use the existing validation envelope with
`details.fields.email=["The email is unavailable."]`. Invitation creation uses
an endpoint-specific 409 response that preserves existing `conflict` cases and
adds the recoverable variant.

**Rationale**: A distinct recoverable code gives authorized clients a safe next
action, while the identical 422 body prevents lifecycle or tenant enumeration.
Invitation creation already has unrelated 409 cases, so replacing its response
with a recovery-only component would misdocument current behavior.

**Alternatives considered**: Generic 422 for recoverable users was rejected
because clients would still need a deleted-user browser that is out of scope.
Changing the common conflict response was rejected because it is shared by many
unrelated operations. A new recovery endpoint was rejected because existing
restore/update operations are sufficient.

## Decision 6: Let MySQL arbitrate races and translate one exact index failure

**Decision**: Keep the friendly precheck, but catch Laravel 13's
`UniqueConstraintViolationException` only when its index is
`users_email_unique`. Catch outside the full database transaction. After
rollback, write one generic duplicate audit and throw the canonical 422 without
re-querying for recoverability.

**Rationale**: Two requests can pass a precheck. MySQL is the atomic arbiter, and
Laravel 13 exposes the violated index. Catching after rollback guarantees no
partial user, role, or invitation state and ensures the rejection audit is not
rolled back. Persistence-time failures must remain generic by specification.

**Alternatives considered**: Catching all query/SQLSTATE errors was rejected
because UUID, token, or unrelated constraint defects could be mislabeled as
email validation. Auditing inside the failed transaction was rejected because
the audit would roll back. Re-querying after the race was rejected because it
could change a generic race into a state-disclosing response.

## Decision 7: Record duplicate attempts through an allowlisted audit service

**Decision**: Add a focused duplicate-email audit service. Record event type
`user_creation_duplicate_email`; outcome `recoverable_user_conflict` or
`validation_failed`; actor, resolved school/platform scope, workflow, reason,
source IP, and SHA-256 of the canonical email. Set affected resource type/user
UUID only for an authorized recoverable conflict.

**Rationale**: The existing audit table already supports every required value,
so no migration is needed. An explicit allowlist prevents plaintext email or
hidden target data from entering metadata. SHA-256 matches an existing
account-lifecycle metadata convention; audit access controls and retention
treat the value as pseudonymous operational data.

**Alternatives considered**: Relying on the global audit sanitizer was rejected
because it is shallow and intentionally does not remove email today. Plaintext
email was rejected by the specification. A new audit table was rejected as
duplicate storage. A dedicated HMAC key was deferred because it adds deployment
configuration outside this focused behavior; it can be introduced through a
future audit-hardening feature.

## Decision 8: Verify both deterministic translation and real concurrency

**Decision**: Test normalized/state/privacy decisions with regular MySQL feature
tests, test index-specific exception translation deterministically, and add a
barrier-controlled two-connection MySQL integration test for the creation race.
Fixtures for the concurrency case must be committed and visible to both
connections rather than hidden inside the test transaction.

**Rationale**: A sequential duplicate test does not prove the gap between
precheck and insert. Separate deterministic and integration coverage keeps
failure diagnosis clear while satisfying the at-most-one identity outcome.

**Alternatives considered**: `artisan test --parallel` was rejected because its
workers use separate test databases. A mocked precheck alone was rejected
because it does not exercise MySQL uniqueness. An uncoordinated timing test was
rejected as flaky.

## Decision 9: Keep frontend and existing restore behavior unchanged

**Decision**: Deliver specification/OpenAPI first and backend second. Do not add
frontend handling, automatic restoration, a deleted-user browser, or changes to
restore/update payloads.

**Rationale**: Existing clients may already handle 409/422 envelopes, and the
feature explicitly excludes frontend delivery. The returned UUID is sufficient
for an authorized client to invoke the existing explicit restore workflow.

**Alternatives considered**: Auto-restore was rejected because it bypasses
reason, effective-date, dependency, and audit rules. Coordinated frontend work
was rejected as unnecessary scope. A new restore contract was rejected because
the current versioned operation already supports the required transition.

# Research: System Administrator Master Access

## Decision: Treat System Administrator as a global permission-check override

**Rationale**: The role is intended to be the product master user. Duplicating
every feature-specific permission onto the role creates drift and can block
operational recovery when new permissions are added. A single documented
override keeps behavior consistent while preserving normal permission checks
for every other role.

**Alternatives considered**: Assigning every permission to the role was
rejected because future permissions could be missed. Granting only platform
routes was rejected because the requested role must access school-scoped and
self-service routes too.

## Decision: Keep tenant and subject context gates outside the override

**Rationale**: Permission checks answer whether the actor may perform a class
of action. Tenant and subject context answer which school or person-specific
record is being targeted. Keeping those gates separate lets System
Administrator access everything intentionally without loading unscoped school
or self-service data.

**Alternatives considered**: Allowing unscoped school-owned reads was rejected
because it breaks tenant isolation. Loading self-service pages as the signed-in
System Administrator was rejected because self-service pages are identity-owned
and need an explicit selected subject.

## Decision: Allow System Administrator to select any active school

**Rationale**: The master role must support onboarding, administration,
support, and emergency operations across the platform. Restricting the role to
assigned schools would recreate the permission-management burden this feature
removes. Limiting selection to active schools preserves school lifecycle
controls.

**Alternatives considered**: Explicit school assignment was rejected because it
does not satisfy "master user" behavior. Support-access-only selection was
rejected because this feature covers general administration, not only support
diagnostics.

## Decision: Enforce approval workflows and business safety gates

**Rationale**: Approval flows, support opt-ins, explicit confirmations,
closed-period checks, file safety gates, and similar controls protect business
state, data safety, or tenant trust. Master access should satisfy permission
checks, but bypassing these controls would make critical workflows less safe
and harder to review.

**Alternatives considered**: Bypassing every approval was rejected because it
weakens operational accountability. Bypassing only platform approvals was
rejected because it creates inconsistent semantics across workflows.

## Decision: Mark audit evidence for writes and lifecycle actions

**Rationale**: Writes and lifecycle actions change protected state and need
reviewable evidence when performed through master access. Read-only navigation
and list/detail viewing can rely on existing access evidence unless a specific
operation already requires an audit record.

**Alternatives considered**: Auditing only existing operation logs was rejected
because reviewers could not reliably identify master-access writes. Auditing
every read was rejected because it substantially expands audit volume without a
clear requirement for this feature.

## Decision: Preserve non-System Administrator behavior

**Rationale**: This feature changes one role's permission evaluation. Existing
school administrator, teacher, student, guardian, platform support, and other
limited roles must continue to receive current denial behavior when required
permissions are missing.

**Alternatives considered**: Broadening platform roles generally was rejected
because the user explicitly named System Administrator as the master role.

## Decision: Preserve response envelopes and distinguish denial causes

**Rationale**: The feature changes authorization decisions, not resource
schemas. Existing success envelopes should remain stable. Forbidden responses
caused only by missing feature-specific permissions must not apply to System
Administrator, but tenant-context, subject-context, account-state,
school-state, safety-gate, and release-state denials must remain distinct.

**Alternatives considered**: Adding new success fields was rejected because
clients do not need resource-shape changes. Collapsing all blocked states into
one denial was rejected because users and tests need to distinguish missing
context from authorization failure.

## Decision: Verify through contract, backend, and frontend tests

**Rationale**: The rule affects the shared API contract, backend
authorization, audit evidence, frontend route guards, navigation visibility,
and tenant/subject gating. Redocly/OpenAPI lint verifies documented
authorization notes, PHPUnit verifies backend behavior, and Vitest verifies
frontend access behavior.

**Alternatives considered**: Manual-only verification was rejected because this
is a platform-wide authorization change. Backend-only verification was rejected
because frontend route guards could still hide or deny System Administrator.

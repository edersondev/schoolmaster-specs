# Research: System Administrator Frontend Access

## Decision: Use one shared master-access predicate

**Decision**: Derive master access from the existing resolved session role
collection through a single shared authorization predicate, then consume that
predicate in route guards and navigation/action visibility helpers.

**Rationale**: A global role override must not be duplicated across route
modules and pages. A shared decision prevents individual screens from drifting
from the backend's exact active platform-role rule.

**Alternatives considered**:

- Per-route or per-page role checks: rejected because they create incomplete
  coverage and inconsistent navigation behavior.
- Treat all role strings as master access: rejected because the backend
  contract requires the exact active platform role named `System Administrator`.

## Decision: Evaluate after session resolution

**Decision**: Route permission evaluation waits for the existing session
bootstrap state before deciding System Administrator access.

**Rationale**: Direct navigation during role loading must not briefly deny or
redirect a valid System Administrator session.

**Alternatives considered**:

- Evaluate before session restoration: rejected because it produces transient
  false denials and unstable navigation.
- Assume master access before session resolution: rejected because it exposes
  protected UI before authentication is confirmed.

## Decision: Retain context and safety guards around permission override

**Decision**: Apply master access only at feature-permission decisions. Keep
school-context, subject-context, authentication, release, approval, and safety
guards as separate required checks.

**Rationale**: This mirrors the completed backend rule and prevents client-side
visibility from suggesting a bypass that the backend correctly denies.

**Alternatives considered**:

- Skip all guards for System Administrator: rejected because it weakens tenant
  and identity boundaries.
- Hide all actions until a backend response: rejected because it discards the
  approved role-aware frontend experience.

## Decision: Clear tenant-owned UI state before context reload

**Decision**: On an active-school change, invalidate or clear school-owned
state before starting requests for the new school.

**Rationale**: Loading indicators are safer than displaying records from the
previous tenant while the new context resolves.

**Alternatives considered**:

- Keep prior data until replacement succeeds: rejected because it can present
  stale cross-tenant information.
- Reload only the current page: rejected because shared stores and navigation
  can retain tenant-owned state outside that page.

## Decision: Preserve identity-owned self-service boundaries

**Decision**: Do not apply the frontend override to student self-service or
guardian self-service routes outside their existing actor-owned profile or
guardian-link context.

**Rationale**: Feature 031 explicitly excludes System Administrator
impersonation and selected-subject transport. The frontend must not advertise
access the backend will deny.

**Alternatives considered**:

- Allow entry with arbitrary selected subject: rejected because it requires a
  new backend contract and expands privacy-sensitive scope.
- Let the route load then show backend denial: rejected because navigation
  visibility should respect known identity boundaries.

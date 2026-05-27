# Research: Backend Administration Lifecycle Management

## Decision: Define the next backend slice as administration lifecycle management

**Rationale**: The roadmap marks student profile and enrollment as already specified in `006`, making administration lifecycle the next unspecific backend dependency. Existing school administration behavior covers mostly list/create foundations. Operational maintenance now needs contract-governed detail, update, activation, deactivation, soft delete, restore, and selected bulk lifecycle actions.

**Alternatives considered**:

- Start account lifecycle workflows next: rejected because invitations, password setup, reset, recovery, and lock recovery are a separate security-sensitive roadmap item.
- Start classroom/course/section/roster modeling next: rejected because administration maintenance gaps still affect users, roles, academic structures, guardians, and schools that roster work depends on.
- Start frontend administration screens now: rejected because backend lifecycle contracts and conflict rules are not yet published.

## Decision: Expand OpenAPI before backend implementation

**Rationale**: Current contracts publish foundation list/create behavior and selected school management behavior, but not a complete lifecycle surface for the affected resources. Contract-first governance requires operation IDs, request schemas, response envelopes, lifecycle status semantics, dependency conflict responses, soft-delete behavior, tenant-context errors, and bulk all-or-nothing semantics before backend exposure.

**Alternatives considered**:

- Implement backend lifecycle routes first and backfill OpenAPI later: rejected because it violates the repository workflow and risks frontend/backend drift.
- Reuse create/list endpoints with action-like request fields: rejected because lifecycle transitions need explicit authorization, conflict, history, and response semantics.

## Decision: Separate platform-scoped school lifecycle from school-scoped resource lifecycle

**Rationale**: `School` is the v1 tenant root and is managed at platform scope. Users, roles, academic years, academic periods, guardians, and their associations are school-owned records that require an active permitted school context. This separation preserves the rule that platform administration is not an implicit bypass for school-scoped module actions.

**Alternatives considered**:

- Allow platform administrators to manage school-owned records through platform operations: rejected because it would create an implicit cross-tenant override without a documented support-access model.
- Require school context for school lifecycle actions: rejected because schools are tenant roots and cannot be owned by themselves.

## Decision: Treat delete as recoverable soft delete only

**Rationale**: Administrative records are linked to authorization, academic records, guardian associations, student enrollment, reports, and audit history. Recoverable soft deletion preserves history, supports restore decisions, and aligns with the constitution's default for recoverable business records.

**Alternatives considered**:

- Permanent deletion: rejected because retention, anonymization, legal hold, and purge rules are not specified.
- Deactivation only with no delete operation: rejected because product scope explicitly calls for delete/restore where allowed.

## Decision: Record lifecycle history for administrative transitions

**Rationale**: Activation, deactivation, soft deletion, restoration, and dependency-blocked transitions need explainability for support review, audit, and future reporting. Lifecycle history should capture actor, previous state, new state, reason, effective date, and operation context without exposing sensitive cross-tenant data.

**Alternatives considered**:

- Store only the current status and deletion timestamp: rejected because it cannot explain who changed state or why.
- Reuse authentication audit events only: rejected because administrative lifecycle decisions need resource-specific context and dependency evidence.

## Decision: Make bulk lifecycle actions bounded and all-or-nothing

**Rationale**: Bulk lifecycle requests multiply tenant, authorization, validation, and dependency risks. Restricting each request to one resource type, one action, one permitted scope, a documented maximum count, and all-or-nothing semantics keeps outcomes predictable and testable.

**Alternatives considered**:

- Partial success responses: rejected because they complicate review, rollback, and frontend reconciliation before single-resource lifecycle rules are proven.
- Mixed-resource bulk operations: rejected because dependency and authorization checks differ by resource.
- Unbounded bulk operations: rejected because they create operational and performance risk without a documented business need.

## Decision: Dependency conflicts block invalid lifecycle transitions

**Rationale**: Lifecycle changes can affect active users, role assignments, academic calendars, attendance, grades, learning sets, enrollment history, reports, guardian associations, and tenant availability. Invalid transitions must return documented conflict responses rather than silently changing state or breaking dependent workflows.

**Alternatives considered**:

- Cascade deactivate or delete dependencies automatically: rejected because cascading business effects are not specified and could hide data loss or access changes.
- Allow lifecycle changes and let downstream workflows fail later: rejected because it creates inconsistent state and unclear support outcomes.

## Decision: Defer adjacent workflows and permanent purge

**Rationale**: Account lifecycle, roster models, teacher corrections, guardian self-service, report lifecycle expansion, frontend behavior, billing, messaging, notifications, anonymization, permanent deletion, and purge each need separate actors, permissions, contracts, and retention rules. Including them here would blur the product boundary.

**Alternatives considered**:

- Include nearby workflows for convenience: rejected because route existence and payload shape are public contract behavior.
- Add placeholder backend routes for future use: rejected because placeholders are still undocumented API behavior.

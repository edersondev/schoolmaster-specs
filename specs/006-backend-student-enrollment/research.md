# Research: Backend Student Profile and Enrollment Management

## Decision: Define the next backend slice as student profile and enrollment management

**Rationale**: Completed slices validate and consume existing `StudentProfile` records for guardians, teacher assignments, student self-view, grades, attendance, and reports, but no backend feature currently governs creation or lifecycle management of those profiles. Making this the next slice removes an implicit dependency before classroom, roster, guardian self-service, correction, or advanced reporting work.

**Alternatives considered**:

- Start classroom/course/roster modeling next: rejected because roster membership depends on a stable student profile lifecycle.
- Start administration lifecycle updates first: rejected because student profile lifecycle is the immediate dependency exposed by the completed reporting and teacher workflow slices.
- Add frontend student administration screens now: rejected because the backend contract and lifecycle behavior are not yet published.

## Decision: Expand OpenAPI before backend implementation

**Rationale**: Current contracts publish authentication, school administration, teacher workflows, student self-view, and reports, but not student profile create/detail/status/transfer operations. Contract-first governance requires these operations, request schemas, response envelopes, authorization behavior, tenant errors, conflict behavior, and lifecycle status semantics to be documented before backend exposure.

**Alternatives considered**:

- Implement local backend routes first and backfill OpenAPI later: rejected because it violates the repository's mandatory workflow and risks frontend/backend drift.
- Reuse guardian endpoints for student profile lifecycle behavior: rejected because guardian management and student enrollment have different authorization, history, and tenant constraints.

## Decision: Limit initial operation boundary to list, create, detail, status update, and transfer recording

**Rationale**: The feature must establish core lifecycle management without opening broader administration behavior. These operations cover the known dependency gap while preserving explicit future expansion for merge, purge, restore, bulk import, roster assignment, and correction workflows.

**Alternatives considered**:

- Include full CRUD and bulk import: rejected because deletion, restore, merge, anonymization, and bulk import need separate retention and audit rules.
- Include classroom or roster assignment with enrollment: rejected because v1 explicitly lacks class/course/section/group models.
- Include guardian self-service: rejected because guardian-facing access requires a separate actor and authorization model.

## Decision: Use same-school atomic guardian association validation

**Rationale**: Guardian creation already treats invalid student references atomically. Student profile creation should mirror that pattern in reverse: every supplied guardian must be active, same-school, and authorized before the profile and associations are created. This avoids partial records and cross-tenant relationship leakage.

**Alternatives considered**:

- Create the student profile and skip invalid guardians: rejected because it hides data-quality and authorization failures.
- Allow cross-school guardian links for shared family cases: rejected because no cross-school guardian model is documented for v1.

## Decision: Preserve enrollment history as append-only lifecycle evidence

**Rationale**: Inactive and transferred student states affect student self-view, teacher assignment eligibility, reports, and administrative history. An append-only enrollment history gives audit and support teams a durable explanation of lifecycle changes without mutating academic records.

**Alternatives considered**:

- Store only the current status on `StudentProfile`: rejected because it cannot explain past status changes or transfer timing.
- Allow history edits through this slice: rejected because corrections and administrative audit mutations require explicit future rules.

## Decision: Treat transfer as source-school lifecycle recording, not academic data movement

**Rationale**: Tenant isolation requires historical grades, attendance, learning sets, private content, report references, and guardian links to remain owned by the source school. A destination profile can be created or linked only through explicitly documented, permission-checked behavior, and source-school history must not become visible to the destination school.

**Alternatives considered**:

- Move the existing `StudentProfile` to another `school_id`: rejected because it would rewrite tenant ownership and break historical references.
- Copy source academic records to the destination school: rejected because no cross-school transcript or migration rule is specified.
- Allow platform administrators to transfer across schools without school context: rejected because platform access is not an implicit school-scoped permission bypass.

## Decision: Defer unsupported lifecycle and adjacent workflows

**Rationale**: The slice should remain narrowly focused. Permanent deletion, restore, merge, anonymization, classroom/roster workflows, teacher assignment workflows, academic-record correction, report changes, guardian self-service, billing, messaging, notifications, and frontend implementation each require separate product and contract decisions.

**Alternatives considered**:

- Include nearby workflows for convenience: rejected because it would blur product boundaries and create undocumented API behavior.
- Add placeholders in backend code for future routes: rejected because route existence itself is public behavior and must be contract-approved.

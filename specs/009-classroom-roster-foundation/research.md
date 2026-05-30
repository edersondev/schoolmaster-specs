# Research: Backend Classroom Roster Foundation

## Decision: Define the next backend slice as classroom roster foundation

**Rationale**: `008-account-lifecycle-workflows` closes operational account access gaps. The backend roadmap identifies classroom, course, section, group, roster membership, teacher assignment, and academic-period scoping as the next dependency before teacher workflow lifecycle corrections, guardian self-service, report lifecycle expansion, platform support access, or advanced assessment types. Existing teacher workflows assign learning sets directly to selected student profiles; group-based teaching needs a formal roster foundation before later workflows can depend on roster context.

**Alternatives considered**:

- Start teacher workflow lifecycle and corrections next: rejected because correction authority and visibility will depend on class/roster ownership and teacher assignment rules.
- Start guardian self-service next: rejected because guardian visibility should not be expanded while roster and class membership context is undefined.
- Start report lifecycle expansion next: rejected because group-based reporting needs roster membership history and academic-period scoping first.
- Start frontend roster screens next: rejected because backend contracts and tenant-safe behavior are not published.

## Decision: Expand OpenAPI before backend implementation

**Rationale**: Current contracts do not publish `ClassSection/Roster`, roster membership, or teacher assignment management operations. Contract-first governance requires operation IDs, request schemas, response schemas, lifecycle states, duplicate conflict responses, all-or-nothing batch behavior, tenant-context errors, school-administrator authorization rules, pagination/filter/sort rules, and legacy direct-assignment compatibility expectations before backend exposure.

**Alternatives considered**:

- Implement backend routes first and backfill OpenAPI later: rejected because it violates API-first governance and risks frontend/backend drift.
- Reuse existing learning-set assignment operations with roster-shaped payloads: rejected because roster membership and teacher assignment are distinct school administration concepts with their own history, conflicts, and lifecycle rules.
- Publish only internal models without public contract changes: rejected because the feature exists to create a contract-governed backend surface.

## Decision: Use one primary ClassSection/Roster resource in v1 with structured metadata fields

**Rationale**: The clarification session selected one primary `ClassSection/Roster` model with structured course, classroom, section, and group metadata fields. This creates a smaller, testable contract while preserving useful labels for schools. It avoids prematurely introducing separate top-level or internal course, classroom, section, and group resources with unresolved hierarchy, dependency, migration, and lifecycle behavior.

**Alternatives considered**:

- Separate Course, Classroom, Section, and Group resources: rejected because it expands the contract and lifecycle state space before product rules require those resources independently.
- Separate internal Course, Classroom, Section, and Group tables hidden behind `ClassSection/Roster`: rejected because hidden lifecycle and migration complexity would still exist without a public contract.
- Course plus Section plus Roster resources: rejected because it still requires additional hierarchy and dependency rules not needed for the v1 foundation.
- Flexible JSON metadata only: rejected because structured request fields are easier to validate, document in OpenAPI, and test for unsupported shapes.
- Free-text labels only: rejected because it would under-specify the contract and weaken validation.

## Decision: Limit each structured metadata block to optional code and name

**Rationale**: Course, classroom, section, and group metadata need enough structure for stable labels and validation, but not independent lifecycle behavior. Allowing only optional `code` and `name` in each metadata block keeps OpenAPI schemas small, blocks hidden resource modeling, and prevents unreviewed descriptive or relational fields from entering the API.

**Alternatives considered**:

- Name-only metadata blocks: rejected because many schools use short section, room, or course codes for operational lookup.
- Code, name, and description: rejected because descriptions are not required for the roster foundation and add payload surface without clear validation value.
- Defer metadata fields entirely to OpenAPI: rejected because backend data modeling and validation tasks need a concrete field boundary before implementation.

## Decision: Require code uniqueness per school and academic period

**Rationale**: `ClassSection/Roster.code` is required and unique per school and academic period, while names may repeat. This gives administrators and contracts a stable human-facing identifier for conflict checks and lookup without preventing the same code from being reused in later academic periods.

**Alternatives considered**:

- Unique name per school and academic period: rejected because display names often repeat across grade levels, shifts, or programs.
- Unique code across all academic periods: rejected because schools commonly reuse class section codes each year.
- Unique code/name pair: rejected because it permits ambiguous duplicate codes in the same period.

## Decision: Keep roster management school-administrator-only

**Rationale**: Teacher assignment management and roster membership management define who can operate against student groups. Limiting writes to authorized school administrators gives the clearest authorization boundary and avoids inventing lead teacher, department manager, or platform support override roles in this slice.

**Alternatives considered**:

- Allow lead teachers to manage assignments: rejected because lead ownership rules are not defined in the current platform model.
- Allow department managers to manage assignments: rejected because department authority is not specified.
- Allow platform administrators to manage school rosters by default: rejected because it creates an implicit cross-tenant support override.

## Decision: Allow teachers to read only their own active assignments

**Rationale**: Future teacher workflows need a stable way to validate that a teacher is assigned to a teaching structure, but teachers should not gain roster administration or broad roster visibility from this foundation. Limiting teacher visibility to the teacher's own active assignments supports access checks without exposing other teachers' assignments or student roster memberships.

**Alternatives considered**:

- Give teachers no roster or assignment read access: rejected because later teacher workflows would lack a contract-governed assignment basis.
- Allow teachers to list assigned rosters and memberships: rejected because student roster membership visibility requires separate teacher workflow scope and privacy review.
- Allow teachers to manage assignments they own: rejected because management writes are school-administrator-only in this slice.

## Decision: Use narrow lifecycle states

**Rationale**: `ClassSection/Roster` uses `active` and `inactive`, `RosterMembership` uses `active` and `ended`, and `TeacherAssignment` uses `active` and `inactive`. These states support current administration, history, and access checks without adding archive, draft, restore, or purge behavior that would need additional product rules.

**Alternatives considered**:

- Use active/inactive/archived for all records: rejected because archive semantics are not needed and could conflict with history retention.
- Use draft/active/inactive/archived for class sections: rejected because draft publishing behavior is not required for the backend foundation.
- Use ended for teacher assignments: rejected because teacher assignment status should align with active/inactive access semantics while membership needs an ended history state.

## Decision: Start ClassSection/Roster records active and reject reactivation

**Rationale**: New `ClassSection/Roster` records start as `active`; `inactive` is reached only through the documented inactivation workflow, and inactive records cannot be reactivated in v1. This keeps lifecycle behavior one-way and avoids hidden restore semantics for old memberships, teacher assignments, and teacher access. If a school needs to resume a structure, administrators create a new roster with its own explicit history.

**Alternatives considered**:

- Allow inactive creation: rejected because inactive records would be unusable in v1 when reactivation is not supported.
- Allow school administrators to reactivate inactive rosters: rejected because reactivation requires unresolved rules for old memberships, teacher assignments, audit history, and duplicate conflicts.
- Restore prior memberships and teacher assignments on reactivation: rejected because that creates implicit access changes outside the foundation scope.

## Decision: Reject roster inactivation while active dependencies exist

**Rationale**: Inactivating a `ClassSection/Roster` with active memberships or active teacher assignments would create ambiguous current access and history. Requiring administrators to end memberships and deactivate teacher assignments first keeps lifecycle intent explicit, avoids hidden cascading changes, and gives each lifecycle transition its own audit event and reason.

**Alternatives considered**:

- Automatically end memberships and deactivate assignments: rejected because cascading writes can surprise administrators and obscure why each related record changed.
- Allow roster inactivation while dependencies remain active but unusable: rejected because it creates active records that cannot be used and complicates authorization checks.
- Soft-delete the roster instead: rejected because permanent or restore-style lifecycle behavior is out of scope.

## Decision: Limit effective dates to today-or-past dates inside the academic period

**Rationale**: Membership and assignment changes often need administrative cleanup for dates that already occurred, but future scheduling introduces pending-state behavior that this foundation does not define. Limiting effective dates to today or past dates within the selected academic period supports history correction without adding future activation semantics. Today-or-past validation uses the resolved school's configured timezone, falling back to the application default timezone only when the school has no configured timezone, so tenant-local school operations do not depend on server timezone drift.

**Alternatives considered**:

- Allow future effective dates inside the academic period: rejected because future scheduling needs pending state, activation jobs, and additional conflict behavior.
- Always use the system date: rejected because schools need to record historical membership and assignment changes accurately.
- Allow dates outside the academic period: rejected because it breaks the academic-period scoping boundary.
- Use UTC for all date comparisons: rejected because the domain is school-local calendar administration, not instant-based scheduling.
- Defer timezone behavior to OpenAPI: rejected because backend tests need deterministic date-boundary behavior.

## Decision: Require reasons for lifecycle-ending actions

**Rationale**: Roster inactivation, roster membership ending, and teacher assignment deactivation remove records from current use while preserving history. Requiring reasons improves auditability and support review for state changes that affect access and membership. Creates and ordinary updates stay lightweight.

**Alternatives considered**:

- Require reasons for every create, update, and lifecycle operation: rejected because routine creates and metadata updates would become unnecessarily heavy.
- Make reasons optional everywhere: rejected because lifecycle-ending actions need auditable business context.
- Require reasons only for roster inactivation: rejected because membership ending and teacher assignment deactivation also affect current access.

## Decision: Make batch roster membership changes all-or-nothing

**Rationale**: Roster membership is naturally batch-oriented, but partial success creates complicated support, retry, and response semantics. All-or-nothing behavior keeps tenant safety and validation predictable: if any requested member is invalid, inactive, transferred, deleted, cross-tenant, duplicate, overlapping, lacks active same-school enrollment covering the membership effective start date, or if the batch exceeds 100 requested changes, no membership in that request changes. The 100-change ceiling is large enough for normal classroom administration while preventing the endpoint from becoming an undocumented bulk import path.

**Alternatives considered**:

- Per-member success/failure results: rejected because partial rosters make operational reconciliation harder and require a more complex response contract.
- One membership change per request only: rejected because school administration commonly needs to add or end multiple memberships together.
- Best-effort partial writes with audit only: rejected because it weakens atomicity and makes tests less deterministic.
- No explicit batch size limit: rejected because it leaves transaction size, response size, and retry behavior ambiguous.
- Higher batch limits such as 250: rejected because large imports need separate import behavior and operational controls.

## Decision: Require explicit effective start dates for membership and teacher assignment creation

**Rationale**: Effective start dates determine eligibility, history, audit records, and future workflow access. Requiring explicit start dates avoids silent defaults to today or the academic-period start date and keeps OpenAPI request schemas, backend validation, and test expectations deterministic.

**Alternatives considered**:

- Default omitted start dates to today: rejected because it can hide administrative backdating intent and produce inaccurate history.
- Default omitted start dates to the academic-period start date: rejected because not every membership or assignment starts at the period boundary.
- Let OpenAPI decide per operation later: rejected because inconsistent defaults across membership and assignment creation would create rework and unclear tests.

## Decision: Anchor student membership eligibility to enrollment on the effective start date

**Rationale**: Roster membership creation requires active same-school enrollment covering the membership effective start date. This gives validation a concrete date anchor, supports legitimate mid-period student joins, and avoids requiring whole-period enrollment before a student can be rostered.

**Alternatives considered**:

- Require enrollment covering the entire academic period: rejected because it blocks normal mid-period enrollment and transfer scenarios.
- Check only current active same-school status: rejected because it ignores enrollment history and can create memberships before or after valid enrollment coverage.
- Defer to generic enrollment service behavior without a roster-specific date rule: rejected because roster tests and OpenAPI validation need a deterministic eligibility boundary.

## Decision: Anchor teacher assignment eligibility to role on the effective start date

**Rationale**: Teacher assignment creation requires the target teacher to have an active same-school teacher-compatible role on the assignment effective start date. This mirrors the student eligibility date anchor and supports legitimate mid-period staffing changes without requiring role coverage across the entire academic period.

**Alternatives considered**:

- Require teacher-compatible role coverage for the entire academic period: rejected because staffing can change mid-period and the assignment start date is the actual access boundary.
- Check only current active same-school role status: rejected because it can create assignment history that does not match the effective start date.
- Defer to generic role service behavior without an assignment-specific date rule: rejected because teacher assignment tests need a deterministic date-based eligibility rule.

## Decision: Reject lifecycle-ending dates before effective start dates

**Rationale**: Membership end and teacher assignment deactivation effective dates must be on or after their effective start dates. This closes a concrete validation gap, preserves chronological history, and avoids impossible membership or assignment intervals without adding future scheduling semantics.

**Alternatives considered**:

- Allow end/deactivation before start when a reason is supplied: rejected because an administrative reason should not override impossible date ordering.
- Do not compare ending dates to start dates in v1: rejected because it would leave history and audit records internally inconsistent.
- Automatically adjust invalid ending dates to the start date: rejected because silent date correction would hide administrator input errors.

## Decision: Resolve concurrent conflicting writes transactionally

**Rationale**: Duplicate prevention, dependency checks, all-or-nothing membership batches, and lifecycle transitions can race under concurrent requests. Transactional conflict handling ensures one conflicting request succeeds and the other returns the documented conflict response without partial state, preserving tenant safety and deterministic audit history.

**Alternatives considered**:

- Serialize and retry conflicting writes before returning conflict: rejected because retry semantics would need additional timing and idempotency rules.
- Last-write-wins updates: rejected because it can overwrite lifecycle intent and weaken auditability.
- Leave concurrency behavior unspecified: rejected because it would make duplicate and dependency tests non-deterministic.

## Decision: Expose teacher assignment detail retrieval in v1

**Rationale**: The spec allows teachers to list and retrieve only their own active teacher assignments, and school administrators need contract-governed same-school assignment detail retrieval. Adding `GET /api/v1/teacher-assignments/{teacherAssignmentId}` prevents list-only workarounds and gives authorization tests a clear route for own-active-assignment and other-teacher denial behavior.

**Alternatives considered**:

- Keep teacher assignment access list-only: rejected because clients would need to infer detail behavior from list payloads.
- Allow detail retrieval for school administrators only: rejected because it conflicts with the clarified teacher read requirement.
- Delay detail retrieval to a later feature: rejected because the current slice already establishes the teacher assignment read boundary.

## Decision: Require tenant-safe audit event field minimums

**Rationale**: Roster foundation actions affect access, membership, and historical state. Audit events must therefore include actor user ID, school ID, target type and target ID, action, outcome, lifecycle reason when present, and tenant-safe summary metadata. This is enough for traceability and support review without storing full request payloads or private student, teacher, credential, or unauthorized cross-tenant data.

**Alternatives considered**:

- Minimal actor, target, action, and outcome only: rejected because lifecycle reasons and school context are essential for tenant-safe support review.
- Full request payload capture: rejected because it risks storing private or cross-tenant data unnecessarily.
- Defer audit fields to backend implementation: rejected because audit verification would be untestable from the specification.

## Decision: Keep list query behavior narrow and paginated

**Rationale**: Roster foundation list endpoints need bounded, predictable behavior for admin workflows and future UI consumption. V1 lists support pagination with default page size 25 and maximum page size 100, plus `academicPeriodId` and `status` filters only. Include expansion, additional filters, and sort behavior are not approved unless OpenAPI later documents them.

**Alternatives considered**:

- Pagination only with no filters: rejected because academic-period and status filtering are core to roster administration.
- Broader filters such as code, name, teacher ID, or student profile ID: rejected because those imply search and relationship traversal behavior not needed for the foundation.
- Include expansion in v1: rejected because it can leak adjacent student, teacher, or membership data without a separate visibility review.
- Defer query behavior to OpenAPI: rejected because list tests and response-shape checks need concrete limits.

## Decision: Reject overlapping active memberships and duplicate active teacher assignments

**Rationale**: Overlapping active roster memberships for the same student, roster, and academic period are rejected without changing existing membership history. Duplicate active teacher assignments for the same teacher, `ClassSection/Roster`, and academic period are also rejected. These rules create deterministic conflict behavior and avoid ambiguous teacher access or roster membership state.

**Alternatives considered**:

- End previous records automatically: rejected because automatic state changes can surprise administrators and obscure historical intent.
- Allow duplicates if metadata differs: rejected because it would make access checks and reporting ambiguous.
- Allow overlaps and rely on effective dates: rejected because the v1 status model should stay simple and testable.

## Decision: Preserve existing direct learning-set assignments as read-only legacy records

**Rationale**: Existing teacher workflow assignments must remain readable for history and current student self-view/reporting behavior. New roster-aware workflow writes must use roster memberships once those workflows are approved, but migration, backfill, or removal of legacy reads requires a future specification and contract update.

**Alternatives considered**:

- Keep direct assignments valid indefinitely for new writes: rejected because it undermines the move toward roster-aware workflows.
- Require immediate migration before roster workflows are enabled: rejected because it creates rollout risk and data migration scope beyond this foundation.
- Hide legacy direct assignments: rejected because existing learning-set, grade, attendance, student self-view, and report behavior must not regress.

## Decision: Keep platform support access out of scope

**Rationale**: The platform already separates platform administration from school-owned workflow access. Roster data includes student membership and teacher assignment context, so any platform support view would need explicit redaction, minimization, auditing, and authorization exceptions in a separate feature.

**Alternatives considered**:

- Allow platform administrators to manage rosters for support: rejected because it creates an implicit cross-tenant override.
- Allow read-only platform roster visibility: rejected because support access is a separate roadmap item with distinct audit and privacy obligations.
- Add emergency access now: rejected because no emergency support workflow is specified.

# Research: SchoolMaster Platform Foundation

## Decision 1: Separate repositories with contract-first sequencing

- **Decision**: Keep `schoolmaster-specs`, `schoolmaster-backend`, and
  `schoolmaster-frontend` as independent repositories, with OpenAPI and
  business rules authored in the specification repository before implementation
  begins in the other two repositories.
- **Rationale**: This matches the project constitution, gives a clear review
  boundary for business rules, and prevents the frontend or backend from
  inventing interface behavior outside the agreed contract.
- **Alternatives considered**:
  - Monorepo with contracts embedded in backend: rejected because it weakens
    contract visibility for frontend-first review and product-level governance.
  - Backend-defined contracts after implementation: rejected because it violates
    API-first delivery and delays cross-team alignment.

## Decision 2: Tenant model anchored on school ownership

- **Decision**: Treat each school as the primary tenant boundary and require
  every school-scoped record to carry explicit tenant ownership, whether
  directly or through an owned parent relationship.
- **Rationale**: The product is a multi-tenant school SaaS, and school-level
  isolation is the core security boundary for users, content, academic
  structure, reports, grades, and attendance.
- **Alternatives considered**:
  - Mixed tenant strategies by module: rejected because it increases leakage
    risk and makes authorization and reporting inconsistent.
  - Global shared records without tenant attribution: rejected because it
    conflicts with the isolation requirement.

## Decision 3: Role model split between platform and school scope

- **Decision**: Use a dual-scope authorization model where system
  administrators operate at platform scope, while school administrators,
  teachers, and students operate within one school tenant context.
- **Rationale**: The specified actor list already separates platform-wide and
  school-scoped responsibilities, and the distinction is necessary to keep
  school data isolated while still allowing tenant provisioning.
- **Alternatives considered**:
  - Fully global roles for all users: rejected because it would complicate
    tenant isolation and expose unnecessary privileges.
  - Separate products for platform administration and school operations:
    rejected for v1 because the spec requires both scopes in one SaaS platform.

## Decision 4: Academic structure as the organizing timeline for operations

- **Decision**: Academic years and academic periods form the canonical time
  structure for grades, attendance, learning sets, and reports.
- **Rationale**: The product scope repeatedly references academic years and
  academic periods, and a shared time structure is required to keep school
  operations coherent.
- **Alternatives considered**:
  - Free-form scheduling with no canonical academic timeline: rejected because
    it would weaken reporting, validation, and business rules.
  - Attendance and grades independent from academic periods: rejected because
    it would create inconsistent reporting and lifecycle rules.

## Decision 5: Content and assessments are teacher-owned school assets

- **Decision**: Teacher folders, uploaded content, questionnaires, and learning
  sets are school-owned records managed by teachers under school permissions,
  not personal assets detached from tenant governance.
- **Rationale**: Content must remain available for school operations, reporting,
  and continuity even when staff status changes, and it must never escape the
  tenant boundary.
- **Alternatives considered**:
  - Personal teacher asset ownership only: rejected because it complicates
    handover, auditability, and school-level reporting.
  - Global shared content library: rejected for v1 because it introduces
    ambiguous ownership and curation rules.

## Decision 6: Status-driven lifecycle with reversible deactivation

- **Decision**: Core administrative and operational records use explicit active
  or inactive status, with reversible deactivation preferred over destructive
  removal where the record participates in academic history or reporting.
- **Rationale**: The specification explicitly requires status fields, and
  school data often needs historical integrity for audits and reporting.
- **Alternatives considered**:
  - Hard delete as the default lifecycle: rejected because it endangers audit
    history and operational recovery.
  - Soft delete only with no explicit status: rejected because the product
    requirement calls for active/inactive business status, not just storage
    lifecycle state.

## Decision 7: Initial API surface grouped by product modules

- **Decision**: Publish initial `/api/v1` REST endpoints by module group:
  authentication, schools, users, roles, academic years, academic periods,
  guardians, teacher content, questionnaires, learning sets, grades,
  attendance, and reports.
- **Rationale**: This mirrors the scope in the feature specification and gives
  backend and frontend teams a predictable contract surface to plan against.
- **Alternatives considered**:
  - Single catch-all administrative endpoint family: rejected because it makes
    ownership, authorization, and client integration less clear.
  - Overly granular contract split before domain design: rejected because it
    would add noise without improving v1 planning.

## Decision 8: Verification strategy tied to critical flows

- **Decision**: Treat onboarding, academic setup, teacher instructional
  management, attendance, grades, and student/report visibility as critical
  flows that require OpenAPI verification, backend PHPUnit coverage, and
  frontend Vitest coverage before merge.
- **Rationale**: These flows represent the main user value for launch and cross
  multiple repositories, so partial testing would not be a safe release gate.
- **Alternatives considered**:
  - Backend-only verification: rejected because frontend and contract drift are
    explicit project risks.
  - Manual acceptance only: rejected because it does not satisfy the
    constitution's release gates.

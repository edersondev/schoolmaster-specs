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

## Decision 9: P2 upload policy for teacher content

- **Decision**: Limit v1 instructional uploads to PDF, images, text, and office
  documents up to 25 MB. Reject executables and archives before persistence,
  store accepted files privately by tenant, and expose files only after malware
  scan completion and authorization checks.
- **Rationale**: This policy supports common classroom material workflows while
  reducing storage, security, and validation risk for launch.
- **Alternatives considered**:
  - Broad media support including audio and video: rejected for v1 because it
    increases storage, streaming, scanning, and UX complexity.
  - Metadata-only content records: rejected because teacher upload is part of
    the approved P2 workflow.

## Decision 10: P2 questionnaire scope

- **Decision**: Support multiple-choice, true-or-false, and short-text
  question types in v1 questionnaires.
- **Rationale**: These types cover common launch assessments while keeping
  authoring, validation, response capture, and frontend rendering tractable.
- **Alternatives considered**:
  - Multiple-choice only: rejected because it is too narrow for initial teacher
    assessment workflows.
  - Long-text and file-response questions: rejected for v1 because they require
    additional review, storage, grading, and moderation behavior not yet
    specified.

## Decision 11: P2 learning set assignment model

- **Decision**: Assign learning sets directly to selected active
  `StudentProfile` records in the same school and academic period.
- **Rationale**: The current model has `StudentProfile` but no class, course,
  or group entity. Direct assignment keeps the contract implementable without
  inventing an unapproved grouping model.
- **Alternatives considered**:
  - Assignment to all students in an academic period: rejected because it does
    not support selective teacher distribution.
  - Class or group assignment: rejected for v1 until a class or group model is
    specified.

## Decision 12: P2 grade value model

- **Decision**: Represent v1 grade values as numeric decimal values from 0 to
  100, with an optional display label.
- **Rationale**: Numeric values support validation, aggregation, sorting, and
  launch reports while allowing schools to show a friendly label when needed.
- **Alternatives considered**:
  - Letter grades only: rejected because it requires school-defined catalogs
    before reporting can be reliable.
  - Flexible string grades: rejected because they prevent consistent
    aggregation and validation in v1.

## Decision 13: P2 attendance status catalog

- **Decision**: Use the fixed v1 attendance status catalog `present`, `absent`,
  `late`, `excused`, `remote`, and `suspended`.
- **Rationale**: The catalog covers common in-person, remote, and exceptional
  attendance states while keeping validation contract-backed.
- **Alternatives considered**:
  - `present` and `absent` only: rejected because it omits common school
    attendance states.
  - School-defined attendance statuses: rejected for v1 because it introduces
    catalog management and reporting variability not yet specified.

## Decision 14: P3 student content access

- **Decision**: Allow students to view assigned teacher content metadata and
  download the file only when the content belongs to their school, is included
  in a learning set assigned to their active student profile, has a clean
  malware scan status, and passes authorization.
- **Rationale**: This preserves the teacher workflow while making student
  access explicit, tenant-bound, and dependent on the upload security gate.
- **Alternatives considered**:
  - Metadata-only access: rejected because approved P3 clarification allows
    file download.
  - Direct file URL exposure: rejected because it would bypass tenant and
    assignment authorization.

## Decision 15: P3 student learning timeline order

- **Decision**: Require `academic_period_id` when listing a student's learning
  timeline. Order assigned learning sets by publish date and entries inside a
  learning set by teacher-defined sequence.
- **Rationale**: Academic period filtering keeps the student view aligned with
  the school timeline, while publish-date and entry sequence provide stable,
  product-approved ordering without inventing classes or groups.
- **Alternatives considered**:
  - Unfiltered student timeline: rejected because it can mix academic periods.
  - Sorting only by assignment date: rejected because the approved clarification
    uses learning set publish date.

## Decision 16: P3 report output formats

- **Decision**: Generate PDF and CSV outputs for all launch-scope report types:
  attendance, grades, academic structure, and school activity.
- **Rationale**: PDF supports review and sharing, while CSV supports operational
  analysis without report-type-specific format negotiation.
- **Alternatives considered**:
  - PDF only: rejected because it limits downstream analysis.
  - Per-report format configuration: rejected for v1 because the clarification
    standardizes both formats for all launch reports.

## Decision 17: P3 report generation lifecycle

- **Decision**: Treat every report request as asynchronous. The API returns a
  `ReportRun` with status, and generated outputs are downloadable only after
  the run reaches `generated`.
- **Rationale**: A consistent asynchronous model avoids separate synchronous
  and background code paths and keeps larger tenant reports from blocking
  request handling.
- **Alternatives considered**:
  - Synchronous generation for smaller reports: rejected because the approved
    clarification requires asynchronous generation for all report types.
  - Client-side report generation: rejected because authorization, tenant
    scoping, and canonical output generation belong on the backend.

## Decision 18: P3 report filters

- **Decision**: Require `academic_period_id` for every report request and allow
  `student_profile_id`, `user_id`, `status`, `start_date`, and `end_date` as
  optional filters where relevant to the selected report type.
- **Rationale**: Academic period is the shared reporting boundary, and the
  optional filters cover the approved launch filter set without adding
  report-specific business rules not yet specified.
- **Alternatives considered**:
  - No required report filter: rejected because it weakens academic-period
    alignment and can produce overly broad tenant reports.
  - Report-type-specific required filters: rejected until the product spec
    defines those constraints.

## Decision 19: P3 report output retention

- **Decision**: Retain generated report output files for 90 days after
  generation, then expire the files while retaining `ReportRun` metadata.
- **Rationale**: This gives school administrators a practical download window
  while limiting long-lived private report files. Keeping metadata preserves
  audit and history without promising permanent output-file availability.
- **Alternatives considered**:
  - 30-day retention: rejected because it is less forgiving for school
    operational review cycles.
  - Retention until academic period close: rejected because period duration can
    vary and makes storage behavior less predictable.
  - No retained files with regeneration on every download: rejected because it
    increases repeat processing and changes download behavior.

## Decision 20: P3 expired report regeneration

- **Decision**: Expired report output files are not regenerated automatically
  during download. Users request a new `ReportRun` with the same filters to
  generate fresh output files.
- **Rationale**: Explicit regeneration keeps authorization, validation, and
  tenant-bound report filters on the normal report request path instead of
  hiding background work behind a download action.
- **Alternatives considered**:
  - Automatic regeneration on download: rejected because it makes a read action
    trigger asynchronous write behavior and can surprise clients.
  - No regeneration path: rejected because administrators still need a way to
    recreate outputs from the approved filters after expiry.

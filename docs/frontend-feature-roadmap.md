# Frontend Feature Roadmap

This roadmap lists frontend feature areas that should be specified before
implementation in `schoolmaster-frontend`. Each item needs approved product
workflow, contract review, and backend readiness before frontend code consumes
new behavior.

## Current Frontend Baseline

Documented frontend baseline sources:

- [frontend-architecture.md](frontend-architecture.md): shared Vue SPA
  structure, service boundary, state boundary, and UI composition rules.
- [frontend-admin-system-architecture.md](frontend-admin-system-architecture.md):
  System Administrator panel blueprint, reusable CRUD foundation, layout, theme
  rules, stores, router baseline, shared contracts, and example module structure.
- [frontend-guidelines.md](frontend-guidelines.md): implementation guardrails
  for Vue, Pinia, Vue Router, Element Plus, Tailwind, and reusable admin
  patterns.

The first frontend architecture slice is complete in
`specs/015-frontend-architecture-baseline/`, and the System Administrator shell
runtime slice is complete in `specs/016-admin-shell-dashboard/` and
`schoolmaster-frontend`.

## Recommended Sequence

### 1. Frontend Architecture Baseline

**Status**: Complete. Baseline specification, implementation plan, contracts,
tasks, and supporting documentation are complete in
`specs/015-frontend-architecture-baseline/`, `docs/frontend-architecture.md`,
`docs/frontend-guidelines.md`, `docs/frontend-admin-system-architecture.md`,
and `docs/naming-conventions.md`.

**Purpose**: Define the durable frontend baseline for Vue 3, JavaScript,
Element Plus, Tailwind, routing, stores, services, composables, contracts, clean
frontend boundaries, and reusable CRUD-oriented patterns.

**Why next**: The frontend needs one stable architectural source of truth
before layout, auth, and module specs start consuming shared patterns.

**Contract and backend gate**: None beyond preserving the API-first rule that
frontend behavior may consume only approved OpenAPI-backed endpoints and
contract semantics.

### 2. System Administrator Shell and Dashboard Foundation

**Status**: Complete. Feature specification, implementation plan, completed
tasks, and frontend implementation are complete in
`specs/016-admin-shell-dashboard/` and `schoolmaster-frontend`.

**Purpose**: Define the reusable `AdminSystemLayout.vue`, sidebar navigation,
top header, permission-aware menu visibility, dashboard summary cards, recent
activity surface, quick actions, responsive shell behavior, and the reusable
admin frame that later modules will live inside.

**Why next**: The admin shell is the composition surface for later
administration, reporting, and support modules. Reusable navigation and CRUD
framing should exist before deeper module work begins.

**Contract and backend gate**: This slice documents placeholder-only dashboard
summary, recent activity, and notification regions. Live data remains blocked
until later specs and OpenAPI contracts approve it.

### 3. Authentication and Session Foundation UI

**Status**: Complete. Feature specification, quality checklist, implementation
plan, research, data model, quickstart, frontend contract, tasks, and runtime
implementation are complete in `specs/017-auth-session-ui/` and
`schoolmaster-frontend`. School selection remains intentionally blocked pending
an approved user-authorized source contract.

**Purpose**: Define login, forgot-password, authenticated session bootstrap,
session expiration handling, current-user hydration, permission loading, and
layout selection behavior for the SPA.

**Why next**: Every protected frontend surface depends on a stable
authentication, session, and permission baseline before protected CRUD pages
can behave predictably inside the admin shell.

**Contract and backend gate**: Consume only approved authentication,
current-user, permission, tenant-context, password-recovery entry, and
user-authorized school-selection contracts. School selection remains blocked
unless an approved OpenAPI operation returns only schools authorized for the
current user. Specify session-expired, unauthorized, forbidden, inactive-user,
inactive-school, and tenant-mismatch UI behavior before implementation.

### 4. Administration Foundation UI

**Status**: Complete. Feature specification, quality checklist, implementation
plan, research, data model, frontend contract, quickstart, tasks, and
verification record are available in
`specs/018-administration-foundation-ui/`. Runtime artifacts are implemented
in `schoolmaster-frontend/src/components/ui/admin/`,
`schoolmaster-frontend/src/components/admin-system/`,
`schoolmaster-frontend/src/composables/admin-system/`,
`schoolmaster-frontend/src/services/admin-system/`, and
`schoolmaster-frontend/src/router/modules/`. Automated unit, build,
responsive, keyboard, latency, denial, and 30-attempt browser-surrogate checks
pass. Moderated UAT with five representative human administrators is complete.

**Purpose**: Define frontend consumption for schools, users, roles,
permissions, academic years, academic periods, and guardians list/create
foundation, including reusable list pages, filter bars, data tables, forms,
and pagination.

**Why next**: These are the first CRUD-heavy operational surfaces and they
establish reusable admin page patterns for later modules.

**Contract and backend gate**: Consume only approved and implemented list/create
operations. Define empty states, pagination behavior, validation display,
authorization denials, and not-found handling for each resource before
implementation.

### 5. Administration Lifecycle UI

**Status**: Complete. Feature specification, implementation plan, research,
data model, frontend contract, quickstart, tasks, runtime implementation,
review fixes, and verification record are complete in
`specs/020-administration-lifecycle-ui/` and `schoolmaster-frontend`. Detail,
update, activate/deactivate, soft-delete/restore, and approved bulk lifecycle
workflows are implemented for supported administration resources, with schools
and permissions retaining their scoped lifecycle constraints.

**Purpose**: Define detail, update, activate/deactivate, delete/restore, and
allowed bulk workflows for administrative resources where contracts approve
them.

**Why next**: Real administration needs lifecycle maintenance, not only
list/create surfaces. This also expands reusable confirm-dialog, detail-page,
status-tag, and conflict-handling patterns.

**Contract and backend gate**: Approve operation IDs, editable-field rules,
status transitions, soft-delete behavior, conflict envelopes, audit-sensitive
actions, and role visibility before the frontend exposes maintenance actions.

### 6. Account Lifecycle Workflows UI

**Status**: Complete. Feature specification, quality checklist,
implementation plan, research, data model, frontend contract, quickstart, and
tasks are complete in `specs/021-account-lifecycle-ui/`.

**Purpose**: Define invitation, password setup, password reset, reactivation,
account lock, and recovery screens and flows for the SPA.

**Why next**: User onboarding and recovery are core operational workflows that
affect both login behavior and admin user management.

**Contract and backend gate**: Approve token-flow behavior, inactive or locked
account states, validation errors, safe denial messages, and actor permissions
before frontend implementation.

### 7. Student Enrollment and Classroom Roster UI

**Status**: Complete. Feature specification, implementation plan, tasks, and
supporting documentation are complete in
`specs/022-student-enrollment-roster-ui/`.

**Purpose**: Define student profile, enrollment, class-section/roster,
membership, teacher assignment, academic-period scoping, and related admin
screens.

**Why next**: Teacher workflows, guardian visibility, and student summaries all
depend on explicit student and roster structure in the UI.

**Contract and backend gate**: Consume only approved student-enrollment and
roster operations. Define transfer behavior, lifecycle status display,
assignment constraints, and conflict handling before implementation.

### 8. Teacher Workflow Workspace

**Status**: Complete. Feature specification, quality checklist,
implementation plan, research, data model, frontend contract, quickstart,
tasks, runtime implementation, and verification record are complete in
`specs/023-teacher-workflow-workspace/` and `schoolmaster-frontend`.
Teacher-facing workspace surfaces and limited same-school admin-observed
surfaces are implemented for approved content, questionnaires, grades,
attendance, correction history, downloads, and JSON imports. Learning-set
create and scoped learning-set, grade, and attendance lists remain
intentionally blocked where OpenAPI contract support is still undocumented.

**Purpose**: Define teacher-facing or admin-observed screens for content,
questionnaires, learning sets, grades, attendance, correction history,
downloads, and imports where the product approves them.

**Why next**: Teacher workflows are a large operational area and need the
reusable list/detail/form/status patterns established by earlier admin work.

**Contract and backend gate**: Consume only approved teacher workflow routes
and define upload, lifecycle, correction, download, empty-state, and conflict
behavior before implementation.

### 9. Student Self-Service UI

**Status**: Complete. Feature specification, quality checklist,
implementation plan, research, data model, frontend contract, quickstart, and
tasks, runtime implementation, and verification evidence are complete in
`specs/024-student-self-service-ui/` and `schoolmaster-frontend`.

**Purpose**: Define the frontend surfaces for assigned learning sets, own
grades, own attendance, authorized content access, and approved academic or
reporting views for students.

**Why next**: Student UI should consume mature contracts only after the admin,
classroom, and teacher workflow baselines are stable.

**Contract and backend gate**: Define authenticated student routing,
student-only visibility, unavailable-content handling, and report/status UI
behavior before implementation.

### 10. Guardian Self-Service UI

**Status**: Complete. Feature specification, implementation plan, research,
data model, frontend contract, quickstart, tasks, runtime implementation, and
verification evidence are complete in
`specs/025-guardian-self-service-ui/` and `schoolmaster-frontend`.
Guardian-facing linked student list/detail, current-period academic summary,
and approved contact views are implemented with tenant-safe authorization,
non-enumerating not-found handling, stale-response protection, diagnostics
redaction, and blocked unsupported guardian capabilities.

**Purpose**: Define guardian-facing list/detail, academic summary, and contact
views using the approved guardian self-service contract.

**Why next**: Guardian access requires a separate UX and authorization model
and should follow the more foundational student and administration surfaces.

**Contract and backend gate**: Consume only approved guardian self-service
operations and define association-missing, inactive-link, cross-tenant, and
not-found UI behavior before implementation.

### 11. Reporting Workspace UI

**Status**: Complete. Feature specification, quality checklist,
implementation plan, research, data model, frontend contract, quickstart,
tasks, runtime implementation, and verification evidence are complete in
`specs/026-reporting-workspace-ui/` and `schoolmaster-frontend`. Reporting
workspace catalog, request, history, detail, download, lifecycle, and custom
definition surfaces are implemented for approved reporting contracts, with
safe authorization handling, active-school timezone rendering,
stale-response protection, diagnostics redaction, and blocked unsupported
reporting actions.

**Purpose**: Define report catalog browsing, report request flows, report run
history, retry or cancellation where approved, custom report definitions, and
download surfaces for reporting users.

**Why next**: Reporting spans multiple modules and benefits from the shared
CRUD, filter, status, and async-operation patterns established earlier.

**Contract and backend gate**: Define report state rendering, retention
visibility, custom-definition ownership, output availability messaging, and
authorization behavior before implementation.

### 12. Platform Support Access and Cross-School Oversight UI

**Status**: Complete. Feature specification, quality checklist,
implementation plan, research, data model, frontend contract, quickstart,
tasks, runtime implementation, and verification evidence are complete in
`specs/027-platform-support-ui/` and `schoolmaster-frontend`. Platform-only
oversight, support access decision review, approval and revocation flows,
redacted diagnostics, and support audit review are implemented for approved
platform/support contracts with tenant-safe routing, stale-response
protection, suppression preservation, and blocked unsupported actions.

**Purpose**: Define the platform-facing frontend for minimized cross-school
summaries, support drill-down, approval state visibility, redacted details, and
support audit review.

**Why next**: This is a sensitive platform-only surface that must sit on top of
stable role, reporting, and authorization patterns.

**Contract and backend gate**: Consume only approved platform/support routes
and define redaction, small-count suppression, approval expiry, opt-in,
revocation, and denied-access UI behavior before implementation.

### 13. Advanced Assessment Frontend UX

**Status**: Not specified.

**Purpose**: Define questionnaire authoring, long-text or file-response
submission, manual grading, student result visibility, and related report
surfaces if and when advanced assessment contracts are approved.

**Why next**: The current assessment UI should remain narrow until the backend
contract for advanced assessment types, grading, and file handling is stable.

**Contract and backend gate**: Approve question schemas, answer validation,
file-response restrictions, malware-scan visibility, grading states, and
student/report display behavior before frontend implementation.

## Scope Still Outside Frontend Roadmap

These are not recommended frontend feature candidates until product scope
changes:

- Payroll, billing, accounting, and financial management.
- Messaging, live classroom, video conferencing, and parent portal behavior
  beyond explicitly approved guardian self-service.
- Mobile-native application work.
- Frontend consumption of undocumented backend routes, payloads, or statuses.
- Visual behavior that conflicts with approved authorization, tenant, or
  lifecycle rules.

## Usage Rules

- Start each roadmap item with `speckit-specify` before planning or tasks.
- Do not implement frontend behavior that depends on undocumented API
  operations, fields, statuses, filters, or errors.
- Link frontend work to the relevant feature id and to the OpenAPI operation
  IDs consumed.
- Keep Element Plus component tags in PascalCase.
- Keep backend and frontend repositories pinned to an approved
  `schoolmaster-specs` revision.

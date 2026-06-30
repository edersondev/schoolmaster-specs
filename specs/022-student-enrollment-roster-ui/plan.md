# Implementation Plan: Student Enrollment and Classroom Roster UI

**Branch**: `022-student-enrollment-roster-ui` | **Date**: 2026-06-30 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/022-student-enrollment-roster-ui/spec.md`

## Summary

Implement roadmap item 7 in `schoolmaster-frontend`: school administrator
student profile list/create/detail, enrollment lifecycle status, transfer,
class-section roster list/create/detail/update/status, all-or-nothing roster
membership batch add/end capped at 100 requested changes, and admin teacher
assignment list/create/detail/deactivate for the current active academic period.

The implementation extends the completed admin shell, CRUD foundation,
administration lifecycle, and account lifecycle UI patterns. Route pages remain
thin composition surfaces. Services own Axios calls and OpenAPI envelope
mapping. Composables coordinate list query state, route/query academic-period
selection, stale-response protection, form drafts, batch selection, conflict
feedback, and no-sensitive-data diagnostics. Student profile general edit and
teacher-facing own-assignment screens stay blocked until later approved
contracts or feature specs add them.

## Technical Context

**Language/Version**: JavaScript with Vue 3 SFCs using Composition API and `<script setup>`; TypeScript remains out of scope.
**Primary Dependencies**: Vue 3, Vue Router, Pinia session/shell stores, Axios service boundary, Element Plus, `@element-plus/icons-vue`, Vue I18n, Tailwind CSS, and approved OpenAPI-backed academic-period read, user list, student profile, class-section roster, membership, and teacher-assignment operations.
**Storage**: No backend or database change. List filters, pagination, active academic-period selection, form drafts, confirmation dialogs, batch selections, and lifecycle reasons remain route-local or composable-local. Existing Pinia auth/session state owns current user, permissions, and active school context. Sensitive student, guardian, role, permission, token, payload, and cross-tenant details must not be persisted in reusable frontend state or diagnostics.
**Testing**: Vitest and Vue Test Utils for contract mappers, services, composables, route metadata, list/detail pages, forms, dialogs, status tags, batch membership flows, transfer flows, teacher assignment flows, stale-response handling, route/query period restoration, authorization gates, conflict and denial mapping, and no-sensitive-data diagnostics. Redocly/OpenAPI validation is required only if contract files change.
**Target Platform**: `schoolmaster-frontend` responsive SPA consuming published `/api/v1` contracts from `schoolmaster-specs`.
**Project Type**: Frontend SPA feature with specification and cross-repository delivery artifacts.
**Performance Goals**: With mocked services settling within 1.5 seconds, student, roster, membership, and teacher-assignment list/detail routes render stable loading, populated, empty, validation, denied, conflict, or temporary-unavailable states within 2 seconds; batch add/end and transfer confirmations settle into success or recoverable error state within 2 seconds; stale requests are cancelled or ignored.
**Constraints**: No undocumented endpoint, field, filter, sort, include expansion, page-size behavior, status, permission code, capability flag, lifecycle action, batch mode, or error dependency; no direct Axios in pages/components/router; no general student profile edit without approved `updateStudentProfile`; no teacher-facing own-assignment route in this feature; no roster batch request above 100 requested changes; no client-side tenant inference; no persisted sensitive student, guardian, token, role, permission, reason, full payload, or cross-tenant details; WCAG 2.1 AA target at 390px, 768px, and 1440px; PascalCase Element Plus tags; centralized reusable text.
**Scale/Scope**: Four user-story slices, protected admin routes for students, class sections, roster memberships, and teacher assignments; one student enrollment service family; one classroom roster service family; contract mapping helpers; route/query academic-period selection; shared status, denial, conflict, batch, and confirmation components; focused frontend tests.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. This frontend feature consumes approved
  student profile, class-section roster, roster membership, and
  teacher-assignment operation IDs. No OpenAPI change is planned unless
  contract review finds a missing operation or incompatible response.
- PASS: Repository impacts are separated. `schoolmaster-specs` defines this
  plan; `schoolmaster-frontend` implements it; `schoolmaster-backend` remains
  unchanged unless a separate contract/backend feature is required.
- PASS: Backend architecture requirements are N/A because no Laravel route,
  Request, Policy, Resource, Service, DTO, Repository, schema, or public
  identifier change is approved by this UI slice.
- PASS: Frontend design uses Vue 3 Composition API, Pinia only for existing
  shared session/shell state, Vue Router, Tailwind CSS, Element Plus, Axios
  service modules, feature folders, and service-isolated API access.
- PASS: MySQL and database soft-delete behavior are unchanged. School-owned
  student, enrollment, roster, membership, and teacher-assignment records are
  consumed only through authenticated active school context and approved
  backend contracts.
- PASS: API compatibility, authentication, authorization, success envelopes,
  paginated envelopes, validation, conflict, forbidden, tenant, inactive,
  not-found, unsupported filter/sort/page-size, and temporary-unavailable
  outcomes are documented.
- PASS: Vitest covers changed critical frontend flows. OpenAPI validation is
  required only if contract review creates a separate contract change.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/022-student-enrollment-roster-ui/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── student-enrollment-roster-ui-contract.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (target repositories)

```text
# Specification repository
schoolmaster-specs/
├── api/
│   ├── openapi.yaml
│   ├── paths/student-profiles/
│   ├── paths/class-sections/
│   ├── paths/teacher-assignments/
│   └── components/schemas/
│       ├── student-profiles/
│       ├── classroom/
│       └── teacher-assignments/
├── docs/
│   ├── frontend-architecture.md
│   ├── frontend-admin-system-architecture.md
│   ├── frontend-guidelines.md
│   └── frontend-feature-roadmap.md
├── specs/022-student-enrollment-roster-ui/
└── AGENTS.md

# Frontend repository target shape
schoolmaster-frontend/
├── src/
│   ├── components/
│   │   ├── admin-system/
│   │   │   ├── students/
│   │   │   │   ├── StudentProfileForm.vue
│   │   │   │   ├── StudentProfileSummaryPanel.vue
│   │   │   │   ├── StudentEnrollmentStatusPanel.vue
│   │   │   │   └── StudentTransferDialog.vue
│   │   │   ├── class-sections/
│   │   │   │   ├── ClassSectionForm.vue
│   │   │   │   ├── ClassSectionSummaryPanel.vue
│   │   │   │   ├── RosterMembershipBatchPanel.vue
│   │   │   │   └── RosterMembershipTable.vue
│   │   │   ├── teacher-assignments/
│   │   │   │   ├── TeacherAssignmentForm.vue
│   │   │   │   └── TeacherAssignmentTable.vue
│   │   │   └── shared/
│   │   │       ├── AcademicPeriodScopeSelector.vue
│   │   │       ├── AdminLifecycleStatusTag.vue
│   │   │       └── AdminSafeFeedbackState.vue
│   │   └── ui/admin/
│   │       ├── AdminBatchActionDialog.vue
│   │       └── AdminLifecycleConfirmDialog.vue
│   ├── composables/admin-system/
│   │   ├── useAcademicPeriodScope.js
│   │   ├── useStudentProfiles.js
│   │   ├── useStudentProfileLifecycle.js
│   │   ├── useClassSections.js
│   │   ├── useRosterMemberships.js
│   │   └── useTeacherAssignments.js
│   ├── contracts/admin-system/
│   │   ├── student-profiles.js
│   │   ├── classroom-roster.js
│   │   └── teacher-assignments.js
│   ├── locales/
│   │   └── student-enrollment-roster.js
│   ├── pages/admin-system/
│   │   ├── students/
│   │   │   ├── StudentProfilesPage.vue
│   │   │   ├── StudentProfileCreatePage.vue
│   │   │   └── StudentProfileDetailPage.vue
│   │   ├── class-sections/
│   │   │   ├── ClassSectionsPage.vue
│   │   │   ├── ClassSectionCreatePage.vue
│   │   │   └── ClassSectionDetailPage.vue
│   │   └── teacher-assignments/
│   │       ├── TeacherAssignmentsPage.vue
│   │       └── TeacherAssignmentDetailPage.vue
│   ├── router/modules/
│   │   └── access-administration.routes.js
│   └── services/admin-system/
│       ├── studentProfiles.js
│       ├── classroomRoster.js
│       └── teacherAssignments.js
└── tests/unit/student-enrollment-roster/
    ├── contracts/
    ├── services/
    ├── composables/
    ├── components/
    ├── pages/
    └── routes/
```

**Structure Decision**: Extend existing admin-system feature folders instead
of creating a detached school operations module. Student profile pages own
list/create/detail, status, and transfer composition. Class-section pages own
roster metadata and membership composition. Teacher assignment review uses an
academic-period scoped admin list/detail surface because the approved OpenAPI
list operation supports period and status filters only, not a class-section
filter. Services stay split by backend resource family while sharing error and
pagination mapping helpers. No new Pinia store is planned unless implementation
finds shared state that crosses student, class-section, and teacher-assignment
routes; route-local composables are sufficient for filters, drafts, batch
selection, and lifecycle dialogs.

## Component Map

- `StudentProfilesPage.vue`: Route composition surface for student filters,
  pagination, loading, empty, denial, and list actions.
- `StudentProfileCreatePage.vue`: Route composition surface for approved
  student profile creation and validation feedback.
- `StudentProfileDetailPage.vue`: Route composition surface for student detail,
  enrollment status, transfer, status actions, and historical-only cues.
- `StudentProfileForm.vue`: Student creation form; emits approved create
  payload without owning service calls.
- `StudentProfileSummaryPanel.vue`: Frontend-safe identity, status, guardian
  summary, and history cues from approved responses.
- `StudentEnrollmentStatusPanel.vue`: Approved status transition actions,
  current eligibility, effective dates, and safe conflict feedback.
- `StudentTransferDialog.vue`: Transfer confirmation with effective date,
  reason, destination metadata where approved, and denial/conflict display.
- `ClassSectionsPage.vue`: Route composition surface for current active period
  default, route/query selected-period persistence, filters, pagination, and
  list feedback.
- `ClassSectionCreatePage.vue`: Route composition surface for approved
  class-section creation.
- `ClassSectionDetailPage.vue`: Route composition surface for roster metadata,
  roster lifecycle, memberships, and links into teacher-assignment workflows; it
  does not derive a section-scoped assignment list from period-wide pages.
- `ClassSectionForm.vue`: Class-section create/update form for approved code,
  name, and structured metadata fields.
- `ClassSectionSummaryPanel.vue`: Status, academic period, lifecycle
  constraints, dependency conflicts, and metadata display.
- `RosterMembershipBatchPanel.vue`: Batch add/end coordination, selection
  count cap, effective dates, reasons, all-or-nothing feedback, and oversized
  batch blocking.
- `RosterMembershipTable.vue`: Membership list, status tags, history rows,
  pagination, and empty states.
- `TeacherAssignmentsPage.vue`: Admin-only academic-period assignment list and
  create controls using approved period/status filters; no teacher-facing route
  and no section-scoped list inference.
- `TeacherAssignmentDetailPage.vue`: Assignment detail/deactivate composition
  for a known assignment ID.
- `TeacherAssignmentForm.vue`: Assignment create form for class-section,
  teacher, academic period, and effective date inputs. Teacher selection may
  use the approved same-school active user list boundary only with documented
  list parameters, and backend assignment creation remains authoritative for
  teacher-compatible role validation.
- `TeacherAssignmentTable.vue`: Assignment list rows, status, selected period,
  pagination, and safe feedback.
- `AcademicPeriodScopeSelector.vue`: Current active period default and
  route/query selected-period synchronization.
- `AdminBatchActionDialog.vue`: Shared batch confirmation shell for membership
  add/end.
- Route pages: Compose feature components with composables and services; pages
  contain no direct HTTP logic.

## Permission and Capability Gate

| Surface | Required confirmation before enabling |
|---------|---------------------------------------|
| Student profile list/detail | Approved school-scoped student administration permission or session capability and active permitted school context |
| Student profile create | Approved school-scoped student administration permission or session capability and active permitted school context |
| General student profile edit | Blocked unless OpenAPI adds `updateStudentProfile` before implementation |
| Student status update | Approved school-scoped student administration permission or session capability and eligible same-school student state |
| Student transfer | Approved school-scoped student administration permission or session capability, eligible source profile, and permitted destination metadata where submitted |
| Class-section list/detail | Approved school-scoped roster administration permission or session capability and active permitted school context |
| Class-section create/update/status | Approved school-scoped roster administration permission or session capability and active academic-period context |
| Roster membership batch add/end | Approved school-scoped roster administration permission or session capability, active roster, valid selected period, and maximum 100 requested changes |
| Teacher assignment list/detail/create/deactivate | Approved school-scoped roster or teacher-assignment administration permission or session capability, active selected academic period, and eligible teacher state |
| Section-scoped teacher assignment list | Blocked unless OpenAPI adds a documented `classSectionId` filter or class-section assignment include |
| Teacher own-assignment route | Blocked; deferred to Teacher Workflow Workspace |

Backend authorization remains authoritative. Client-side visibility improves
usability only after implementation confirms approved permission codes or
session capability flags.

## Phase 0: Research

Research output is captured in [research.md](research.md). No unresolved
technical clarifications remain.

## Phase 1: Design and Contracts

Design outputs are captured in:

- [data-model.md](data-model.md)
- [contracts/student-enrollment-roster-ui-contract.md](contracts/student-enrollment-roster-ui-contract.md)
- [quickstart.md](quickstart.md)

## Post-Design Constitution Check

- PASS: Design preserves API-first consumption and names every approved
  student, roster, membership, and teacher-assignment operation consumed by the
  frontend.
- PASS: Repository ownership remains explicit; no backend implementation or
  OpenAPI mutation is hidden in frontend work.
- PASS: Pages/components remain presentation and composition boundaries;
  services own transport; composables own list, period scope, form, batch,
  lifecycle, stale-response, and tenant coordination; existing Pinia stores own
  shared session only.
- PASS: Tenant behavior remains explicit: all school-owned UI surfaces use
  authenticated active school context and documented tenant/forbidden response
  states.
- PASS: Authentication, authorization, validation, forbidden, tenant-mismatch,
  inactive-school, inactive-record, not-found, conflict, unsupported
  filter/sort/page-size, oversized batch, and temporary-unavailable outcomes
  map to contract-safe frontend states.
- PASS: Future verification expectations are captured for Vitest and optional
  OpenAPI validation if contracts change.
- PASS: No complexity exception is introduced.

## Complexity Tracking

No constitution violations.

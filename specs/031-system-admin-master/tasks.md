# Tasks: System Administrator Master Access

**Input**: Design documents from `specs/031-system-admin-master/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`,
`contracts/system-admin-master-contract.md`, `quickstart.md`

**Tests**: Required. This backend slice changes global Laravel authorization,
REST authorization notes, tenant and identity-ownership gates, and audit
expectations.

**Organization**: Tasks are grouped by user story so each story can be
implemented and tested independently after shared master-access foundations are
complete.

**Repository Scope**: `schoolmaster-specs` owns security documentation,
multi-tenant documentation, OpenAPI authorization notes, and feature evidence.
`schoolmaster-backend` owns Laravel authorization, policy, tenant and
identity-ownership boundaries, audit, and PHPUnit coverage. Frontend adoption
is deferred to a separate feature and this task list changes no frontend files.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because the task changes different files or can
  be completed without depending on another incomplete task.
- **[Story]**: User story label for story phases only.
- Every task includes a concrete path or an inventory artifact that records the
  final concrete paths before implementation.

## Phase 1: Setup

**Purpose**: Confirm affected contracts, implementation targets, and evidence
locations before shared authorization work starts.

- [X] T001 Review current System Administrator, permission, tenant, support, and self-service authorization language and record contradictions in `specs/031-system-admin-master/quickstart.md`
- [X] T002 [P] Inventory protected OpenAPI operations requiring master-access notes in `api/openapi.yaml` and record the operation groups in `specs/031-system-admin-master/quickstart.md`
- [X] T003 [P] Record the deferred frontend route, navigation, and action-visibility handoff with no frontend repository changes in `specs/031-system-admin-master/quickstart.md`
- [X] T004 [P] Inventory backend authorization policy and permission-check entry points requiring centralized master-access treatment and record entry points in `specs/031-system-admin-master/quickstart.md`
- [X] T005 [P] Record backend fixture requirements for System Administrator, limited role, active school, inactive school, actor-owned student, active guardian link, and audit evidence in `specs/031-system-admin-master/quickstart.md`
- [X] T006 [P] Record the reusable zero-permission System Administrator fixture design following existing helpers in `specs/031-system-admin-master/quickstart.md`
- [X] T007 [P] Confirm Redocly and PHPUnit commands and evidence capture locations in `specs/031-system-admin-master/quickstart.md`

---

## Phase 2: Foundational

**Purpose**: Complete shared specs and every affected OpenAPI authorization
note before establishing the Laravel authorization foundation.

**Critical**: No user story implementation begins before this phase completes.

### Foundational Contract and Documentation Tasks

- [X] T008 Update System Administrator master-access rule and remove contradictory no-bypass wording in `docs/security.md`
- [X] T009 Update tenant selection, any-active-school access, selected-school scoping, and platform-wide exception rules in `docs/multi-tenant.md`
- [X] T010 [P] Document the deferred frontend handoff without changing frontend behavior in `specs/031-system-admin-master/quickstart.md`
- [X] T011 [P] Update backend policy and authorization guidance for System Administrator master access in `docs/backend-guidelines.md`
- [X] T012 Add canonical System Administrator authorization language to `api/openapi.yaml`, apply it to every protected operation under `api/paths/`, explicitly document all platform-wide exceptions, and mirror it in `specs/001-schoolmaster-platform/contracts/openapi.yaml` before any backend behavior changes
- [X] T013 Update common forbidden response examples to distinguish permission denial from tenant-context denial in `api/components/responses/common/AdministrationForbidden.yaml`
- [X] T014 Update classroom or module-specific forbidden response examples to preserve non-permission prerequisite denials in `api/components/responses/classroom/ClassroomRosterForbidden.yaml`
- [X] T015 Update administration foundation permission matrix notes for System Administrator master access in `specs/018-administration-foundation-ui/contracts/administration-foundation-ui-contract.md`
- [X] T016 [P] Update administration lifecycle UI authorization notes for System Administrator master access in `specs/020-administration-lifecycle-ui/contracts/administration-lifecycle-ui-contract.md`
- [X] T017 [P] Update platform support UI permission-gate notes for System Administrator master access without bypassing support approval gates in `specs/027-platform-support-ui/contracts/platform-support-ui-contract.md`
- [X] T018 [P] Update student self-service notes to preserve actor-owned profile authorization and explicitly exclude impersonation or inferred subject selection in `specs/024-student-self-service-ui/contracts/student-self-service-ui-contract.md`
- [X] T019 [P] Update guardian self-service notes to preserve active guardian-link authorization and explicitly exclude impersonation or inferred subject selection in `specs/025-guardian-self-service-ui/contracts/guardian-self-service-ui-contract.md`

### Foundational Backend Tasks

- [X] T020 [P] Add reusable System Administrator and limited-user fixtures through existing helpers in `schoolmaster-backend/tests/TestCase.php`
- [X] T021 [P] Add unit coverage for exact active platform-role detection and permission override boundaries in `schoolmaster-backend/tests/Unit/Models/UserSystemAdministratorAccessTest.php`
- [X] T022 Add `isSystemAdministrator()` and permission-only override semantics to the existing `hasPermission()` and `hasSchoolPermission()` boundary in `schoolmaster-backend/app/Models/User.php`
- [X] T023 Inventory policy preconditions that assume every school-authorized actor has a fixed `users.school_id` and record required tenant/owner-preserving updates for `schoolmaster-backend/app/Policies/ScopePolicy.php`, `schoolmaster-backend/app/Policies/AccountLifecyclePolicy.php`, `schoolmaster-backend/app/Policies/AcademicRecordPolicy.php`, `schoolmaster-backend/app/Policies/AcademicRecordImportPolicy.php`, `schoolmaster-backend/app/Policies/AssessmentPolicy.php`, `schoolmaster-backend/app/Policies/LearningSetPolicy.php`, `schoolmaster-backend/app/Policies/QuestionnairePolicy.php`, `schoolmaster-backend/app/Policies/TeacherContentItemPolicy.php`, and `schoolmaster-backend/app/Policies/TeacherWorkflowPolicy.php` in `specs/031-system-admin-master/quickstart.md`
- [X] T024 Preserve tenant context, actor ownership, guardian links, account/session state, school state, release state, approval, and safety gates around the permission-only override in the concrete policy and service files inventoried in `specs/031-system-admin-master/quickstart.md`

### Deferred Frontend Tasks

- [X] T025 Record as deferred: frontend route guard coverage belongs to a separate frontend implementation feature in `specs/031-system-admin-master/quickstart.md`
- [X] T026 Record as deferred: frontend navigation/action visibility coverage belongs to a separate frontend implementation feature in `specs/031-system-admin-master/quickstart.md`
- [X] T027 Record as deferred: frontend route guard implementation is outside this backend slice in `specs/031-system-admin-master/quickstart.md`
- [X] T028 Verify the existing authenticated-session roles collection already exposes the active platform `System Administrator` role without a new response field in `schoolmaster-backend/tests/Feature/CurrentUserApiTest.php`
- [X] T029 Record as deferred: frontend navigation helper implementation is outside this backend slice in `specs/031-system-admin-master/quickstart.md`

**Checkpoint**: Shared documentation and every affected OpenAPI operation are
updated before the backend override boundary is implemented.

---

## Phase 3: User Story 1 - Access Protected Backend Reads (Priority: P1) MVP

**Goal**: System Administrator can call every released protected backend
read operation without explicit feature-specific permissions, while
non-permission prerequisites still block access. State-changing operation
coverage is completed with audit evidence in US3.

**Independent Test**: Sign in as System Administrator with no extra
feature-specific permissions and verify protected backend reads are allowed
unless tenant, identity-ownership, account/session, release, or safety
prerequisites are missing. State-changing behavior is verified in US3.

### Tests for User Story 1

- [X] T030 [P] [US1] Add OpenAPI lint expectation for master-access authorization wording across protected operation groups in `specs/031-system-admin-master/quickstart.md`
- [X] T031 [P] [US1] Add backend feature tests proving System Administrator can call every released protected read-only operation group listed in `api/openapi.yaml` without feature-specific permissions in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/ProtectedOperationMasterAccessTest.php`
- [X] T032 [P] [US1] Add backend feature tests proving limited non-System Administrator users without required permissions still receive documented permission denial for every released protected operation group in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/LimitedRolePermissionDenialTest.php`
- [X] T033 [P] [US1] Add authenticated-session tests proving the existing role collection identifies System Administrator without a response schema change in `schoolmaster-backend/tests/Feature/CurrentUserApiTest.php`
- [X] T034 [P] [US1] Add backend tests proving actor-owned student access and active guardian-link rules remain enforced without inferred impersonation in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/IdentityOwnedSelfServiceBoundaryTest.php`

### Implementation for User Story 1

- [X] T035 [US1] Verify T012 applied the canonical authorization note to every protected operation group in `api/openapi.yaml` before backend behavior changed and record the operation IDs in `specs/031-system-admin-master/quickstart.md`
- [X] T036 [US1] Verify users, roles, permissions, schools, academics, guardians, reports, platform support, classroom, teacher, student, and assessment path descriptions use the canonical language under `api/paths/`
- [X] T037 [US1] Apply the `User` permission override and the tenant/owner-preserving policy updates inventoried in T004 and T023 to the concrete `schoolmaster-backend/app/Policies/` files recorded in `specs/031-system-admin-master/quickstart.md`
- [X] T038 [US1] Replace remaining direct service permission decisions inventoried in T004 with delegation to `User::hasPermission()` or `User::hasSchoolPermission()` while preserving service business gates in the concrete files under `schoolmaster-backend/app/Services/`
- [X] T039 [US1] Verify `AuthSessionResource` already exposes the active platform role through existing fields and make no response change in `schoolmaster-backend/app/Http/Resources/AuthSessionResource.php`
- [X] T040 [US1] Record as deferred: frontend session-store mapping is outside this backend slice in `specs/031-system-admin-master/quickstart.md`
- [X] T041 [US1] Preserve actor-owned student access in `schoolmaster-backend/app/Services/Concerns/AuthorizesStudentSelfView.php` and active guardian-link resolution in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianAccessResolver.php`
- [X] T042 [US1] Add backend safety-gate regression coverage for approval, confirmation, support opt-in, file safety, and closed-period checks in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/NonPermissionSafetyGateTest.php`
- [X] T043 [US1] Document protected route and operation allow evidence for System Administrator and limited-role denial evidence in `specs/031-system-admin-master/quickstart.md`

**Checkpoint**: User Story 1 is independently testable as a read-only MVP;
state-changing coverage and required audit evidence are delivered in US3.

---

## Phase 4: User Story 2 - Operate Inside School Contexts (Priority: P2)

**Goal**: System Administrator can select any active school for school-scoped
work, and all school-owned responses remain limited to the selected school
unless an operation is explicitly platform-wide.

**Independent Test**: Sign in as System Administrator, select any active school,
open school-owned resources, confirm only selected-school data appears, switch
to another active school, and confirm stale data clears before the new school's
data loads.

### Tests for User Story 2

- [X] T044 [P] [US2] Add backend feature tests proving System Administrator may select any active school context and is blocked from inactive or missing school context in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/SchoolContextSelectionTest.php`
- [X] T045 [P] [US2] Add backend tenant isolation tests proving school-scoped System Administrator operations return only selected-school data and never unscoped cross-school data in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/SchoolScopedTenantIsolationTest.php`
- [X] T046 [P] [US2] Add backend platform-wide operation tests proving any cross-school output is allowed only for documented platform-wide operations in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/PlatformWideOperationScopeTest.php`
- [X] T047 [P] [US2] Add backend tests proving missing, unknown, and inactive `X-School-Id` values fail before school-owned lookup in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/SchoolContextDenialTest.php`
- [X] T048 [P] [US2] Add backend regression coverage across existing module tenant guards in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/ModuleTenantBoundaryTest.php`

### Implementation for User Story 2

- [X] T049 [US2] Preserve and, only where tests require, update active-school selection for platform System Administrator in `schoolmaster-backend/app/Services/TenantContextResolver.php` and missing-context enforcement in `schoolmaster-backend/app/Services/TenantContextService.php`
- [X] T050 [US2] Preserve active-school validation and inactive-school denial for System Administrator in `schoolmaster-backend/app/Http/Middleware/ResolveSchoolContext.php`
- [X] T051 [US2] Enforce selected-school scoping through existing boundaries in `schoolmaster-backend/app/Services/Concerns/AssertsSchoolTenantScope.php`, `schoolmaster-backend/app/Services/Concerns/AssertsAdministrationTenantScope.php`, `schoolmaster-backend/app/Services/Concerns/AssertsStudentEnrollmentTenantScope.php`, `schoolmaster-backend/app/Services/ClassroomRoster/SchoolContextGuard.php`, `schoolmaster-backend/app/Services/TeacherWorkflow/SchoolContextGuard.php`, `schoolmaster-backend/app/Services/Reports/ReportTenantContextService.php`, and `schoolmaster-backend/app/Services/Assessment/AssessmentTenantScopeService.php`
- [X] T052 [US2] Verify the platform-wide exceptions and selected-school scoping documented by T012 in `api/openapi.yaml` against tenant-isolation test evidence
- [X] T053 [US2] Record as deferred: frontend active-school selection is outside this backend slice in `specs/031-system-admin-master/quickstart.md`
- [X] T054 [US2] Verify backend school-context middleware blocks school-owned operations until active context resolves in `schoolmaster-backend/app/Http/Middleware/ResolveSchoolContext.php`
- [X] T055 [US2] Record as deferred: frontend stale-data cleanup is outside this backend slice in `specs/031-system-admin-master/quickstart.md`
- [X] T056 [US2] Document any-active-school selection, inactive-school denial, selected-school isolation, and context-switch evidence in `specs/031-system-admin-master/quickstart.md`

**Checkpoint**: User Story 2 is independently testable for school selection,
tenant isolation, and platform-wide exception boundaries.

---

## Phase 5: User Story 3 - Audit Master Access Decisions (Priority: P3)

**Goal**: System Administrator writes and lifecycle actions record
master-access audit evidence, while read-only navigation keeps existing access
evidence unless an operation already requires audit records.

**Independent Test**: Review security, route, contract, and test evidence to
confirm System Administrator writes/lifecycle actions mark master access,
school context remains required, and non-System Administrator denial behavior
is unchanged.

### Tests for User Story 3

- [X] T057 [P] [US3] Add data-driven allow and audit tests for every state-changing protected operation group inventoried in T002, including create, update, delete, restore, activate, deactivate, import, retry, cancel, approve, and revoke, in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessAuditEvidenceTest.php`
- [X] T058 [P] [US3] Add backend tests proving read-only System Administrator list/detail operations do not create new master-access audit evidence unless the existing operation already audits reads in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessReadAuditBoundaryTest.php`
- [X] T059 [P] [US3] Add backend tests proving actor-owned student and active guardian-link rules plus approval workflows, support opt-ins, confirmations, file safety, closed-period checks, and other safety gates still block System Administrator when prerequisites are unmet in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessSafetyGateTest.php`
- [X] T060 [P] [US3] Add backend tests proving tenant, identity-ownership, approval, safety, validation, and permission denials remain distinct in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessDenialEnvelopeTest.php`
- [X] T061 [P] [US3] Add deterministic contract assertions for the canonical audit marker and non-permission prerequisite language in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessContractDocumentationTest.php`

### Implementation for User Story 3

- [X] T062 [US3] Add an explicit optional master-access flag to `schoolmaster-backend/app/DTOs/AuditEventData.php` and serialize it as tenant-safe `master_access_used` metadata in `schoolmaster-backend/app/Services/AuditEventService.php`
- [X] T063 [US3] Propagate the marker through generic audit adapters in `schoolmaster-backend/app/Services/AccountLifecycle/AccountLifecycleAuditService.php`, `schoolmaster-backend/app/Services/ClassroomRoster/RosterAuditLogger.php`, `schoolmaster-backend/app/Services/GuardianSelfService/GuardianAuditService.php`, `schoolmaster-backend/app/Services/TeacherWorkflow/TeacherWorkflowAuditLogger.php`, and services under `schoolmaster-backend/app/Services/AdministrationLifecycle/`
- [X] T064 [US3] Propagate the same canonical marker through `schoolmaster-backend/app/Services/Assessment/AssessmentAuditService.php`, `schoolmaster-backend/app/Services/Reports/ReportAuditService.php`, and `schoolmaster-backend/app/Services/PlatformSupport/PlatformSupportAuditService.php`, preserving each existing store and first-class tenant fields
- [X] T065 [US3] Wire the marker into every state-changing audit writer inventoried in T003 and preserve actor-owned self-service plus approval and safety gates in the concrete files recorded in `specs/031-system-admin-master/quickstart.md`
- [X] T066 [US3] Add audit context tests proving actor, action, target, outcome, timestamp, and selected school are recorded without cross-tenant leakage in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessAuditContextTest.php`
- [X] T067 [US3] Record as deferred: frontend error mapping and route adoption are outside this backend slice in `specs/031-system-admin-master/quickstart.md`
- [X] T068 [US3] Document audit evidence, safety-gate denial, read-audit boundary, actor-owned student denial, and guardian-link validation in `specs/031-system-admin-master/quickstart.md`

**Checkpoint**: User Story 3 is independently testable for audit evidence and
non-permission gate preservation.

---

## Phase 6: Polish and Cross-Cutting Concerns

**Purpose**: Full-flow verification, documentation cleanup, contract linting,
and delivery evidence across all stories.

- [X] T069 [P] Run `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1` from the specs repository and record both results in `specs/031-system-admin-master/quickstart.md`
- [X] T070 [P] Run backend PHPUnit suite for System Administrator master access and record result in `specs/031-system-admin-master/quickstart.md`
- [X] T071 [P] Run focused backend unit tests for master-role detection, policy override boundaries, and audit metadata and record results in `specs/031-system-admin-master/quickstart.md`
- [X] T072 [P] Review `docs/security.md`, `docs/multi-tenant.md`, affected feature specs, and affected contracts for contradictory no-bypass language and record result in `specs/031-system-admin-master/quickstart.md`
- [X] T073 [P] Review OpenAPI protected operations for consistent permission-denial, tenant-context, identity-ownership, and platform-wide language in `api/openapi.yaml`
- [X] T074 [P] Review backend implementation for direct permission checks that bypass centralized System Administrator master-access behavior and record findings in `specs/031-system-admin-master/quickstart.md`
- [X] T075 [P] Verify no frontend repository files changed and record the deferred frontend handoff in `specs/031-system-admin-master/quickstart.md`
- [X] T076 Verify quickstart scenarios for protected backend access, any-active-school selection, selected-school scoping, identity-owned self-service denial, safety gates, and audit markers in `specs/031-system-admin-master/quickstart.md`
- [X] T077 Update final implementation evidence and remaining known limitations in `specs/031-system-admin-master/quickstart.md`

---

## Dependencies and Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies. Complete before shared foundation.
- **Foundational (Phase 2)**: Depends on Setup. Blocks all user stories.
- **User Stories (Phases 3-5)**: Depend on Foundational completion.
- **Polish (Phase 6)**: Depends on all implemented user stories.

### User Story Dependencies

- **US1 (P1)**: Starts after Foundational. MVP scope.
- **US2 (P2)**: Starts after Foundational. It can be tested with active school
  fixtures independently, but integrates naturally with US1 route access.
- **US3 (P3)**: Starts after Foundational. It can be tested with write,
  lifecycle, and safety-gate fixtures independently, but depends on the same
  master-access decision boundary from US1.

### Within Each User Story

- Write contract and backend tests before implementation.
- Complete every affected contract and documentation update before backend
  behavior changes that depend on the changed rule.
- Backend authorization helper and policies before protected operation wiring.
- Story evidence recorded in quickstart before moving to polish.

### Parallel Opportunities

- Setup inventory tasks T002-T007 can run in parallel after T001 starts.
- Foundational documentation tasks T010-T019 can run in parallel after T008 and
  T009 define canonical wording.
- Foundational backend tasks T020-T024 begin only after T008-T019 complete;
  T025-T029 only record deferred frontend handoff
  or verify the existing session response.
- Story test tasks marked [P] can run in parallel within each story.
- US2 and US3 can begin after Foundational if separate implementers use mocked
  or fixture-backed master-access behavior.
- Polish validation tasks T069-T075 can run in parallel after implemented
  stories are ready.

---

## Parallel Example: User Story 1

```bash
Task: "T031 [P] [US1] Add backend feature tests for every released protected operation group in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/ProtectedOperationMasterAccessTest.php"
Task: "T032 [P] [US1] Add backend feature tests for limited-role permission denial in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/LimitedRolePermissionDenialTest.php"
Task: "T033 [P] [US1] Add existing-session role context tests in schoolmaster-backend/tests/Feature/CurrentUserApiTest.php"
Task: "T034 [P] [US1] Add identity-owned self-service boundary tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/IdentityOwnedSelfServiceBoundaryTest.php"
```

## Parallel Example: User Story 2

```bash
Task: "T044 [P] [US2] Add backend active-school selection tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/SchoolContextSelectionTest.php"
Task: "T045 [P] [US2] Add backend selected-school isolation tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/SchoolScopedTenantIsolationTest.php"
Task: "T047 [P] [US2] Add backend school-context denial tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/SchoolContextDenialTest.php"
Task: "T048 [P] [US2] Add module tenant-boundary tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/ModuleTenantBoundaryTest.php"
```

## Parallel Example: User Story 3

```bash
Task: "T057 [P] [US3] Add backend audit evidence tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessAuditEvidenceTest.php"
Task: "T058 [P] [US3] Add backend read-audit boundary tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessReadAuditBoundaryTest.php"
Task: "T059 [P] [US3] Add backend safety gate tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessSafetyGateTest.php"
Task: "T060 [P] [US3] Add backend denial-envelope tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessDenialEnvelopeTest.php"
```

---

## Implementation Strategy

### MVP First

1. Complete Phase 1 setup.
2. Complete Phase 2 foundation.
3. Complete Phase 3 / US1.
4. Validate that System Administrator can access protected read-only
   operation groups without explicit feature permissions, while limited roles
   remain denied.
5. Stop for review before expanding tenant-context and audited state-changing
   behavior.

### Incremental Delivery

1. US1: master permission-check override and protected read-only operation access.
2. US2: any-active-school selection and selected-school tenant isolation.
3. US3: master-access audit markers, release of audit-gated writes and
   lifecycle actions, and business safety gate preservation.
4. Polish: full contract/backend verification and implementation evidence.

### Parallel Team Strategy

With multiple implementers:

1. One owner updates `schoolmaster-specs` documentation and OpenAPI language.
2. One backend owner implements centralized master-access authorization and
   PHPUnit coverage.
3. Frontend implementation remains deferred to its own feature branch and task
   set.
4. Specification and backend owners reconcile quickstart evidence before final
   review.

## Notes

- [P] tasks touch different files or can be completed independently after
  their phase prerequisites.
- Keep tenant, identity-ownership, and guardian-link denial separate from
  permission denial.
- Do not add unscoped school-owned output unless the operation is documented
  as platform-wide.
- Do not add new success response fields for this feature.

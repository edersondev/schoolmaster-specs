# Tasks: Platform-Wide Reporting and Support Access

**Input**: Design documents from `specs/013-platform-support-access/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/backend-platform-support-access.md, quickstart.md

**Tests**: Critical business flow tests are required. This feature changes REST contracts and backend behavior, so tasks include OpenAPI validation and PHPUnit coverage. Frontend tasks are intentionally excluded because frontend implementation is out of scope for this slice.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because it touches different files or has no dependency on incomplete tasks
- **[Story]**: User story label for story phases only
- Every task includes an exact repository-relative file path

## Phase 1: Setup (Shared Contract and Repository Preparation)

**Purpose**: Establish shared contract files, feature folders, and traceability before backend implementation.

- [ ] T001 Create platform OpenAPI path directory in `schoolmaster-specs/api/paths/platform/`
- [ ] T002 Create platform support schema directory in `schoolmaster-specs/api/components/schemas/platform-support/`
- [ ] T003 [P] Create backend platform support DTO directory in `schoolmaster-backend/app/DTOs/PlatformSupport/`
- [ ] T004 [P] Create backend platform support service directory in `schoolmaster-backend/app/Services/PlatformSupport/`
- [ ] T005 [P] Create backend platform support request directory in `schoolmaster-backend/app/Http/Requests/Api/V1/Platform/`
- [ ] T006 [P] Create backend platform support resource directory in `schoolmaster-backend/app/Http/Resources/Platform/`
- [ ] T007 Create backend platform support test directories in `schoolmaster-backend/tests/Feature/PlatformSupport/` and `schoolmaster-backend/tests/Unit/PlatformSupport/`
- [ ] T008 Document that frontend implementation is out of scope for this slice in `schoolmaster-specs/specs/013-platform-support-access/quickstart.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core contract, contract validation, persistence, authorization, audit, and route foundations that MUST be complete before any user story can be implemented.

**CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T009 Add platform support permissions, operation IDs, request schemas, response schemas, pagination, filtering, sorting, authorization errors, validation errors, and conflict responses for all platform support operations in `schoolmaster-specs/api/openapi.yaml`
- [ ] T010 Add shared platform support error and conflict response schemas in `schoolmaster-specs/api/components/schemas/platform-support/errors.yaml`
- [ ] T011 Add shared platform support audit schemas including support escalation event types in `schoolmaster-specs/api/components/schemas/platform-support/audit.yaml`
- [ ] T012 Add shared platform support decision schemas in `schoolmaster-specs/api/components/schemas/platform-support/support-access-decision.yaml`
- [ ] T013 Add shared target-school support opt-in schemas in `schoolmaster-specs/api/components/schemas/platform-support/support-opt-in.yaml`
- [ ] T014 Add shared platform summary and reporting overview schemas in `schoolmaster-specs/api/components/schemas/platform-support/platform-summaries.yaml`
- [ ] T015 Mirror foundational platform support schemas and operation definitions in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T016 Create support access decision migration in `schoolmaster-backend/database/migrations/2026_06_06_000001_create_support_access_decisions_table.php`
- [ ] T017 Create target-school support opt-in migration in `schoolmaster-backend/database/migrations/2026_06_06_000002_create_target_school_support_opt_ins_table.php`
- [ ] T018 Create internal platform approval migration in `schoolmaster-backend/database/migrations/2026_06_06_000003_create_internal_platform_approvals_table.php`
- [ ] T019 Create platform support audit event migration in `schoolmaster-backend/database/migrations/2026_06_06_000004_create_platform_support_audit_events_table.php`
- [ ] T020 [P] Create SupportAccessDecision model in `schoolmaster-backend/app/Models/SupportAccessDecision.php`
- [ ] T021 [P] Create TargetSchoolSupportOptIn model in `schoolmaster-backend/app/Models/TargetSchoolSupportOptIn.php`
- [ ] T022 [P] Create InternalPlatformApproval model in `schoolmaster-backend/app/Models/InternalPlatformApproval.php`
- [ ] T023 [P] Create PlatformSupportAuditEvent model in `schoolmaster-backend/app/Models/PlatformSupportAuditEvent.php`
- [ ] T024 Add platform support relationships to School model in `schoolmaster-backend/app/Models/School.php`
- [ ] T025 Add platform support relationships to User model in `schoolmaster-backend/app/Models/User.php`
- [ ] T026 Seed platform support permissions in `schoolmaster-backend/database/seeders/PermissionSeeder.php`
- [ ] T027 Create PlatformSupportPolicy authorization boundaries in `schoolmaster-backend/app/Policies/PlatformSupportPolicy.php`
- [ ] T028 Register platform support policy mappings in `schoolmaster-backend/app/Providers/AuthServiceProvider.php`
- [ ] T029 Create PlatformSupportActorContext DTO in `schoolmaster-backend/app/DTOs/PlatformSupport/PlatformSupportActorContext.php`
- [ ] T030 Create PlatformSupportTargetSchool DTO in `schoolmaster-backend/app/DTOs/PlatformSupport/PlatformSupportTargetSchool.php`
- [ ] T031 Create PlatformSupportReasonCode DTO in `schoolmaster-backend/app/DTOs/PlatformSupport/PlatformSupportReasonCode.php`
- [ ] T032 Create PlatformSupportAuditService in `schoolmaster-backend/app/Services/PlatformSupport/PlatformSupportAuditService.php`
- [ ] T033 Create PlatformSupportRedactionService in `schoolmaster-backend/app/Services/PlatformSupport/PlatformSupportRedactionService.php`
- [ ] T034 Create PlatformSupportAuthorizationService in `schoolmaster-backend/app/Services/PlatformSupport/PlatformSupportAuthorizationService.php`
- [ ] T035 Create PlatformSupportController shell without exposing undocumented operations in `schoolmaster-backend/app/Http/Controllers/Api/V1/Platform/PlatformSupportController.php`
- [ ] T036 Run contract-first Redocly validation with `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1` and record results in `schoolmaster-specs/specs/013-platform-support-access/quickstart.md`

**Checkpoint**: Foundation ready. User story implementation can now begin.

---

## Phase 3: User Story 1 - Review Platform Operational Health (Priority: P1) MVP

**Goal**: Platform administrators can review minimized school operational summaries and cross-school reporting health without receiving school-owned detail records.

**Independent Test**: Authenticate as a platform administrator with platform-wide oversight permission, list schools with operational summary fields, request reporting overview, and verify minimized fields, count suppression, audit events, and permission denials.

### Tests for User Story 1

- [ ] T037 [P] [US1] Verify OpenAPI path coverage for `listPlatformSchoolSummaries` before backend route exposure in `schoolmaster-specs/api/paths/platform/schools.yaml`
- [ ] T038 [P] [US1] Verify OpenAPI path coverage for `getPlatformReportingOverview` before backend route exposure in `schoolmaster-specs/api/paths/platform/reporting-overview.yaml`
- [ ] T039 [P] [US1] Verify platform summary and reporting overview operation parity in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T040 [P] [US1] Add PHPUnit feature tests for platform school summary success and minimization in `schoolmaster-backend/tests/Feature/PlatformSupport/ListPlatformSchoolSummariesTest.php`
- [ ] T041 [P] [US1] Add PHPUnit feature tests for cross-school reporting overview success and minimization in `schoolmaster-backend/tests/Feature/PlatformSupport/GetPlatformReportingOverviewTest.php`
- [ ] T042 [P] [US1] Add PHPUnit feature tests for platform overview permission denial and no cross-school disclosure in `schoolmaster-backend/tests/Feature/PlatformSupport/PlatformOverviewAuthorizationTest.php`
- [ ] T043 [P] [US1] Add PHPUnit unit tests for protected count suppression below 5 in `schoolmaster-backend/tests/Unit/PlatformSupport/PlatformSupportRedactionServiceTest.php`

### Implementation for User Story 1

- [ ] T044 [P] [US1] Implement ListPlatformSchoolSummariesRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Platform/ListPlatformSchoolSummariesRequest.php`
- [ ] T045 [P] [US1] Implement GetPlatformReportingOverviewRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Platform/GetPlatformReportingOverviewRequest.php`
- [ ] T046 [P] [US1] Implement PlatformSchoolSummaryResource in `schoolmaster-backend/app/Http/Resources/Platform/PlatformSchoolSummaryResource.php`
- [ ] T047 [P] [US1] Implement PlatformReportingOverviewResource in `schoolmaster-backend/app/Http/Resources/Platform/PlatformReportingOverviewResource.php`
- [ ] T048 [US1] Implement platform school summary aggregation in `schoolmaster-backend/app/Services/PlatformSupport/PlatformSchoolSummaryService.php`
- [ ] T049 [US1] Implement cross-school reporting overview aggregation in `schoolmaster-backend/app/Services/PlatformSupport/PlatformReportingOverviewService.php`
- [ ] T050 [US1] Apply protected count suppression below 5 in `schoolmaster-backend/app/Services/PlatformSupport/PlatformSupportRedactionService.php`
- [ ] T051 [US1] Wire platform school summary and reporting overview controller actions in `schoolmaster-backend/app/Http/Controllers/Api/V1/Platform/PlatformSupportController.php`
- [ ] T052 [US1] Register platform summary and reporting overview routes in `schoolmaster-backend/routes/api.php`
- [ ] T053 [US1] Add tenant-safe audit writes for platform summary and reporting overview reads in `schoolmaster-backend/app/Services/PlatformSupport/PlatformSupportAuditService.php`

**Checkpoint**: User Story 1 is independently functional and is the MVP.

---

## Phase 4: User Story 2 - Perform Authorized Support Drill-Down (Priority: P2)

**Goal**: Platform support users can retrieve only approved redacted diagnostics for one target school after target-school opt-in and internal platform approval.

**Independent Test**: Authenticate as a support user with support drill-down permission, target-school opt-in approved by a same-school administrator with explicit support opt-in permission, and internal platform approval; request diagnostics and verify redaction, 24-hour expiration, denial behavior, and audit events.

### Tests for User Story 2

- [ ] T054 [P] [US2] Verify OpenAPI path coverage for `requestSupportAccess` and `getSupportAccessDecision` before backend route exposure in `schoolmaster-specs/api/paths/platform/support-access/index.yaml`
- [ ] T055 [P] [US2] Verify OpenAPI path coverage for `approveSupportAccess` and `revokeSupportAccess` before backend route exposure in `schoolmaster-specs/api/paths/platform/support-access/actions.yaml`
- [ ] T056 [P] [US2] Verify OpenAPI path coverage for `createSchoolSupportOptIn` and `revokeSchoolSupportOptIn` before backend route exposure in `schoolmaster-specs/api/paths/schools/support-opt-ins.yaml`
- [ ] T057 [P] [US2] Verify OpenAPI path coverage for `getSupportSchoolDiagnostics` before backend route exposure in `schoolmaster-specs/api/paths/platform/support-diagnostics.yaml`
- [ ] T058 [P] [US2] Verify support access, support opt-in, and diagnostics operation parity in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T059 [P] [US2] Add PHPUnit feature tests for support drill-down success in `schoolmaster-backend/tests/Feature/PlatformSupport/GetSupportSchoolDiagnosticsTest.php`
- [ ] T060 [P] [US2] Add PHPUnit feature tests for target-school opt-in create and revoke authorization in `schoolmaster-backend/tests/Feature/PlatformSupport/SchoolSupportOptInTest.php`
- [ ] T061 [P] [US2] Add PHPUnit feature tests for internal platform approval and revocation in `schoolmaster-backend/tests/Feature/PlatformSupport/PlatformSupportApprovalTest.php`
- [ ] T062 [P] [US2] Add PHPUnit feature tests for 24-hour expiration, stale approval, revoked approval, mismatched school, and concurrent revocation/access denials in `schoolmaster-backend/tests/Feature/PlatformSupport/SupportAccessGateDenialTest.php`
- [ ] T063 [P] [US2] Add PHPUnit feature tests for support diagnostics blocked outputs, emergency access, impersonation, search, and writes in `schoolmaster-backend/tests/Feature/PlatformSupport/SupportDiagnosticsBlockedBehaviorTest.php`
- [ ] T064 [P] [US2] Add PHPUnit unit tests for support access decision validation in `schoolmaster-backend/tests/Unit/PlatformSupport/SupportAccessDecisionServiceTest.php`

### Implementation for User Story 2

- [ ] T065 [P] [US2] Implement RequestSupportAccessRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Platform/RequestSupportAccessRequest.php`
- [ ] T066 [P] [US2] Implement ApproveSupportAccessRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Platform/ApproveSupportAccessRequest.php`
- [ ] T067 [P] [US2] Implement RevokeSupportAccessRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Platform/RevokeSupportAccessRequest.php`
- [ ] T068 [P] [US2] Implement CreateSchoolSupportOptInRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Platform/CreateSchoolSupportOptInRequest.php`
- [ ] T069 [P] [US2] Implement RevokeSchoolSupportOptInRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Platform/RevokeSchoolSupportOptInRequest.php`
- [ ] T070 [P] [US2] Implement GetSupportSchoolDiagnosticsRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Platform/GetSupportSchoolDiagnosticsRequest.php`
- [ ] T071 [P] [US2] Implement SupportAccessDecisionResource in `schoolmaster-backend/app/Http/Resources/Platform/SupportAccessDecisionResource.php`
- [ ] T072 [P] [US2] Implement SchoolSupportOptInResource in `schoolmaster-backend/app/Http/Resources/Platform/SchoolSupportOptInResource.php`
- [ ] T073 [P] [US2] Implement SupportDiagnosticResource in `schoolmaster-backend/app/Http/Resources/Platform/SupportDiagnosticResource.php`
- [ ] T074 [US2] Implement target-school support opt-in lifecycle in `schoolmaster-backend/app/Services/PlatformSupport/SchoolSupportOptInService.php`
- [ ] T075 [US2] Implement internal platform approval lifecycle in `schoolmaster-backend/app/Services/PlatformSupport/InternalPlatformApprovalService.php`
- [ ] T076 [US2] Implement support access decision lifecycle, 24-hour gate validation, and atomic stale/revoked/mismatched/concurrently changed approval checks in `schoolmaster-backend/app/Services/PlatformSupport/SupportAccessDecisionService.php`
- [ ] T077 [US2] Implement redacted support diagnostic aggregation in `schoolmaster-backend/app/Services/PlatformSupport/SupportDiagnosticService.php`
- [ ] T078 [US2] Enforce blocked generated report downloads, raw outputs, private metadata, emergency access, impersonation, unrestricted search, and writes in `schoolmaster-backend/app/Services/PlatformSupport/SupportDiagnosticService.php`
- [ ] T079 [US2] Wire support access, support opt-in, approval, revocation, and diagnostics controller actions in `schoolmaster-backend/app/Http/Controllers/Api/V1/Platform/PlatformSupportController.php`
- [ ] T080 [US2] Register support access, support opt-in, approval, revocation, and diagnostics routes in `schoolmaster-backend/routes/api.php`
- [ ] T081 [US2] Add audit writes for support access request, opt-in, approval, revocation, expiration, escalation, diagnostics, and denied outcomes in `schoolmaster-backend/app/Services/PlatformSupport/PlatformSupportAuditService.php`

**Checkpoint**: User Stories 1 and 2 work independently with explicit approval gates.

---

## Phase 5: User Story 3 - Govern Support Exceptions and Audit Review (Priority: P3)

**Goal**: Authorized platform compliance actors can review minimized support-access audit summaries without exposing protected data.

**Independent Test**: Create allowed and denied platform/support access attempts, retrieve audit summaries as an authorized platform compliance actor, and verify minimized metadata, permission denials, target-school attribution, reason codes, and redaction rules.

### Tests for User Story 3

- [ ] T082 [P] [US3] Verify OpenAPI path coverage for `listSupportAuditEvents` before backend route exposure in `schoolmaster-specs/api/paths/platform/support-audit-events.yaml`
- [ ] T083 [P] [US3] Verify support audit operation parity in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T084 [P] [US3] Add PHPUnit feature tests for support audit summary listing including support escalation events in `schoolmaster-backend/tests/Feature/PlatformSupport/ListSupportAuditEventsTest.php`
- [ ] T085 [P] [US3] Add PHPUnit feature tests for support audit review permission denials in `schoolmaster-backend/tests/Feature/PlatformSupport/SupportAuditAuthorizationTest.php`
- [ ] T086 [P] [US3] Add PHPUnit unit tests for audit metadata redaction in `schoolmaster-backend/tests/Unit/PlatformSupport/PlatformSupportAuditServiceTest.php`

### Implementation for User Story 3

- [ ] T087 [P] [US3] Implement ListSupportAuditEventsRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Platform/ListSupportAuditEventsRequest.php`
- [ ] T088 [P] [US3] Implement PlatformSupportAuditEventResource in `schoolmaster-backend/app/Http/Resources/Platform/PlatformSupportAuditEventResource.php`
- [ ] T089 [US3] Implement support audit summary listing in `schoolmaster-backend/app/Services/PlatformSupport/PlatformSupportAuditQueryService.php`
- [ ] T090 [US3] Enforce audit metadata redaction for credentials, tokens, private paths, raw outputs, private content, full records, and unauthorized cross-tenant details in `schoolmaster-backend/app/Services/PlatformSupport/PlatformSupportAuditService.php`
- [ ] T091 [US3] Wire support audit controller action in `schoolmaster-backend/app/Http/Controllers/Api/V1/Platform/PlatformSupportController.php`
- [ ] T092 [US3] Register support audit route in `schoolmaster-backend/routes/api.php`

**Checkpoint**: All user stories are independently functional.

---

## Phase 6: Polish & Cross-Cutting Validation

**Purpose**: Contract validation, regression coverage, traceability, and readiness review across the full slice.

- [ ] T093 [P] Run final Redocly validation with `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1` and record results in `schoolmaster-specs/specs/013-platform-support-access/quickstart.md`
- [ ] T094 [P] Review route-to-operation traceability for every platform support operation in `schoolmaster-specs/specs/013-platform-support-access/contracts/backend-platform-support-access.md`
- [ ] T095 [P] Review aggregate OpenAPI publication for undocumented platform/support behavior in `schoolmaster-specs/api/openapi.yaml`
- [ ] T096 [P] Review platform-local OpenAPI mirror for operation parity in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T097 Run backend PHPUnit suite with `docker exec schoolmaster-backend-app-1 php artisan test` and record results in `schoolmaster-specs/specs/013-platform-support-access/quickstart.md`
- [ ] T098 Review tenant-safe audit payloads across platform support services in `schoolmaster-backend/app/Services/PlatformSupport/`
- [ ] T099 Review response resources for redaction and small-count suppression in `schoolmaster-backend/app/Http/Resources/Platform/`
- [ ] T100 Update backend implementation notes for platform support access in `schoolmaster-backend/README.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies; starts immediately.
- **Phase 2 Foundation**: Depends on Phase 1; blocks all user stories and includes contract-first Redocly validation before backend route exposure.
- **Phase 3 US1**: Depends on Phase 2; MVP.
- **Phase 4 US2**: Depends on Phase 2; can start after foundation, but diagnostics should reuse redaction/audit foundations from US1 where available.
- **Phase 5 US3**: Depends on Phase 2; can start after foundation, but needs audit events produced by US1/US2 for full integration verification.
- **Phase 6 Polish**: Depends on selected user stories and contract changes being complete.

### User Story Dependencies

- **US1 Review Platform Operational Health**: No dependency on other stories after foundation.
- **US2 Perform Authorized Support Drill-Down**: No dependency on US1 for contract shape, but should reuse shared redaction and audit services.
- **US3 Govern Support Exceptions and Audit Review**: Can implement after foundation, but complete validation needs audit event producers from US1 and US2.

### Within Each User Story

- Contract tasks and Redocly validation before backend route exposure.
- Tests before implementation tasks.
- Requests, resources, DTOs, and policies before service/controller wiring.
- Services before route completion.
- Audit and redaction checks before marking the story complete.

---

## Parallel Opportunities

- Setup directory tasks T003-T007 can run in parallel.
- Foundational schema tasks T010-T014 can run in parallel after T009 operation boundaries are defined.
- Foundational model tasks T020-T023 can run in parallel.
- US1 tests T040-T043 can run in parallel after OpenAPI path tasks T037-T039.
- US1 request/resource tasks T044-T047 can run in parallel.
- US2 OpenAPI tasks T054-T058 can run in parallel.
- US2 tests T059-T064 can run in parallel after OpenAPI task ownership is clear.
- US2 request/resource tasks T065-T073 can run in parallel.
- US3 tests T084-T086 can run in parallel after OpenAPI tasks T082-T083.
- US3 request/resource tasks T087-T088 can run in parallel.
- Polish reviews T093-T096 can run in parallel after implementation contract work is complete; T093 is a final validation repeat after the foundational T036 contract gate.

---

## Parallel Example: User Story 1

```bash
Task: "T040 Add PHPUnit feature tests for platform school summary success and minimization"
Task: "T041 Add PHPUnit feature tests for cross-school reporting overview success and minimization"
Task: "T042 Add PHPUnit feature tests for platform overview permission denial and no cross-school disclosure"
Task: "T043 Add PHPUnit unit tests for protected count suppression below 5"
```

---

## Parallel Example: User Story 2

```bash
Task: "T059 Add PHPUnit feature tests for support drill-down success"
Task: "T060 Add PHPUnit feature tests for target-school opt-in create and revoke authorization"
Task: "T061 Add PHPUnit feature tests for internal platform approval and revocation"
Task: "T062 Add PHPUnit feature tests for 24-hour approval and opt-in expiration denials"
Task: "T063 Add PHPUnit feature tests for support diagnostics blocked behavior"
```

---

## Parallel Example: User Story 3

```bash
Task: "T084 Add PHPUnit feature tests for support audit summary listing"
Task: "T085 Add PHPUnit feature tests for support audit review permission denials"
Task: "T086 Add PHPUnit unit tests for audit metadata redaction"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 setup.
2. Complete Phase 2 foundation.
3. Complete Phase 3 User Story 1.
4. Validate platform summaries and reporting overview independently.
5. Stop before support drill-down until MVP behavior and contract validation pass.

### Incremental Delivery

1. Deliver US1 for minimized platform operational oversight.
2. Deliver US2 for read-only support drill-down with approval gates.
3. Deliver US3 for support audit review.
4. Run Phase 6 validation before backend exposure is considered implementation-ready.

### Cross-Repository Sequencing

1. `schoolmaster-specs`: OpenAPI contract and feature artifacts.
2. `schoolmaster-backend`: Laravel implementation and PHPUnit verification.
3. `schoolmaster-frontend`: No work in this slice; future consumption requires a separate approved plan.

---

## Notes

- [P] tasks use different files or can proceed without depending on incomplete tasks.
- All backend routes must map to OpenAPI operation IDs before implementation is considered complete.
- No task authorizes frontend implementation, support writes, generated report downloads, emergency access, unrestricted impersonation, raw report output access, private file metadata access, or unrestricted record search.

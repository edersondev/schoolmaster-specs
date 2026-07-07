# Tasks: School Fields Tabs

**Input**: Design documents from `specs/029-school-fields-tabs/`  
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/school-fields-tabs-contract.md`, `quickstart.md`

**Tests**: Required. This feature changes REST contracts, backend validation and persistence, frontend create/edit UI, upload behavior, tenant boundaries, and critical school administration flows.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after shared foundations are complete.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish contract, backend, and frontend work locations before story work begins.

- [X] T001 Create OpenAPI school lookup path directory in `schoolmaster-specs/api/paths/school-lookups/`
- [X] T002 Create OpenAPI school lookup schema directory in `schoolmaster-specs/api/components/schemas/school-lookups/`
- [X] T003 [P] Create backend school DTO directory in `schoolmaster-backend/app/DTOs/School/`
- [X] T004 [P] Create backend school service directory in `schoolmaster-backend/app/Services/School/`
- [X] T005 [P] Create frontend school module directories in `schoolmaster-frontend/src/modules/schools/`
- [X] T006 [P] Create frontend school unit test directory in `schoolmaster-frontend/tests/unit/schools/`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Define shared API shape, storage, lookup data, and module boundaries required by all user stories.

**Critical**: No user story work should begin until this phase is complete.

- [X] T007 Update school OpenAPI schemas to replace `cnpj` with `document`, add numeric `status`, required `address`, Basic fields, Institutional fields, Branding fields, and uniqueness descriptions in `schoolmaster-specs/api/components/schemas/schools/School.yaml`
- [X] T008 Update create request OpenAPI schema for JSON and multipart school creation, required tabs, legacy `cnpj` rejection, CNPJ validation, uniqueness rules, defaults, and logo file fields in `schoolmaster-specs/api/components/schemas/schools/SchoolCreateRequest.yaml`
- [X] T009 Update update request OpenAPI schema for JSON and multipart school update, read-only `document`, required address, uniqueness rules, defaulted fields, and no optimistic-locking requirement in `schoolmaster-specs/api/components/schemas/schools/SchoolUpdateRequest.yaml`
- [X] T010 [P] Add Institutional lookup option schema in `schoolmaster-specs/api/components/schemas/school-lookups/SchoolLookupOption.yaml`
- [X] T011 [P] Add Institutional lookup list response schema in `schoolmaster-specs/api/components/schemas/school-lookups/SchoolLookupOptionList.yaml`
- [X] T012 Add Institutional lookup path refs to `/api/v1/school-lookups/*` in `schoolmaster-specs/api/openapi.yaml`
- [X] T013 [P] Add administrative type lookup operation in `schoolmaster-specs/api/paths/school-lookups/administrative-types.yaml`
- [X] T014 [P] Add legal nature lookup operation in `schoolmaster-specs/api/paths/school-lookups/legal-natures.yaml`
- [X] T015 [P] Add management type lookup operation in `schoolmaster-specs/api/paths/school-lookups/management-types.yaml`
- [X] T016 [P] Add pedagogical approach lookup operation in `schoolmaster-specs/api/paths/school-lookups/pedagogical-approaches.yaml`
- [X] T017 [P] Add education level lookup operation in `schoolmaster-specs/api/paths/school-lookups/education-levels.yaml`
- [X] T018 [P] Add modality lookup operation in `schoolmaster-specs/api/paths/school-lookups/modalities.yaml`
- [X] T019 Update school create/update path content types and validation responses for JSON and multipart requests in `schoolmaster-specs/api/paths/schools/index.yaml`
- [X] T020 Update school detail/update path content types, read-only `document`, conflict non-scope notes, and validation responses in `schoolmaster-specs/api/paths/schools/school.yaml`
- [X] T021 Add or update backend migration for school profile columns, unique indexes for `inep_code`, `document`, and normalized email, address requirement, institutional references, branding colors, and `logo_path` in `schoolmaster-backend/database/migrations/`
- [X] T022 [P] Add Institutional lookup migration in `schoolmaster-backend/database/migrations/`
- [X] T023 [P] Add Institutional lookup seed data from `docs/school_new_fields.md` in `schoolmaster-backend/database/seeders/SchoolInstitutionalLookupSeeder.php`
- [X] T024 Update backend `School` model fillable/casts/relations for profile, address, institutional, branding, soft-deleted uniqueness checks, and normalized email in `schoolmaster-backend/app/Models/School.php`
- [X] T025 [P] Add Institutional lookup model in `schoolmaster-backend/app/Models/SchoolInstitutionalLookup.php`
- [X] T026 Add shared school profile DTO covering Basic, Address, Institutional, Branding, and logo file input in `schoolmaster-backend/app/DTOs/School/SchoolProfileData.php`
- [X] T027 Add shared school logo storage service with MIME, size, filename sanitization, replacement, and cleanup behavior in `schoolmaster-backend/app/Services/School/SchoolLogoService.php`
- [X] T028 Add shared school profile service shell for create/update orchestration and no optimistic-locking changes in `schoolmaster-backend/app/Services/School/SchoolProfileService.php`
- [X] T029 Update school authorization policy to preserve existing school administration and tenant-root rules for create, detail, update, logo upload, and lookup operations in `schoolmaster-backend/app/Policies/SchoolPolicy.php`
- [X] T030 [P] Add frontend school API types for Basic, Address, Institutional, Branding, lookup options, field errors, and form mode in `schoolmaster-frontend/src/modules/schools/types/school.ts`
- [X] T031 [P] Add frontend school API service shell for JSON/multipart create/update and lookup operations in `schoolmaster-frontend/src/modules/schools/services/schoolService.ts`
- [X] T032 [P] Add frontend tab-to-field error mapping utility in `schoolmaster-frontend/src/modules/schools/utils/schoolTabErrors.ts`

**Checkpoint**: Foundation ready. User story implementation can now begin.

---

## Phase 3: User Story 1 - Create school with grouped fields (Priority: P1) MVP

**Goal**: Authorized administrators can create schools from Basic, Address, Institutional, and Branding tabs with required validation, defaults, lookup options, logo upload, and preserved values.

**Independent Test**: Open school creation, complete all required tabs, submit a valid school, then repeat with invalid values in each tab and verify field/tab feedback without lost values.

### Tests for User Story 1

- [X] T033 [P] [US1] Run Redocly validation for create school required fields, JSON request, multipart `logo_file`, lookup refs, defaults, and legacy `cnpj` rejection against `schoolmaster-specs/api/openapi.yaml` and record create-contract evidence in `schoolmaster-specs/specs/029-school-fields-tabs/quickstart.md`
- [X] T034 [P] [US1] Add backend feature tests for successful create with all tabs, no-logo create, logo create, required Address, required Institutional fields, numeric status, CNPJ check digits, duplicate INEP/document/email including soft-deleted records, and case-insensitive email uniqueness in `schoolmaster-backend/tests/Feature/School/SchoolCreateTest.php`
- [X] T035 [P] [US1] Add backend unit tests for logo validation, storage, MIME rejection, SVG rejection, executable rejection, and 2 MB limit in `schoolmaster-backend/tests/Unit/School/SchoolLogoServiceTest.php`
- [X] T036 [P] [US1] Add frontend service tests for create JSON payloads, multipart payloads, default colors, numeric status, lookup requests, and validation error mapping in `schoolmaster-frontend/tests/unit/schools/schoolService.create.spec.ts`
- [X] T037 [P] [US1] Add frontend form/composable tests for create tabs, required field feedback on inactive tabs, value preservation, Element Plus `ElColorPicker`, logo validation display, and stale lookup response handling in `schoolmaster-frontend/tests/unit/schools/SchoolCreateForm.spec.ts`

### Implementation for User Story 1

- [X] T038 [US1] Implement backend create request validation for Basic, required Address, Institutional, Branding colors, logo file, unique identity fields, case-insensitive email, soft-deleted uniqueness scope, CNPJ check digits, numeric status, and legacy `cnpj` rejection in `schoolmaster-backend/app/Http/Requests/Api/V1/SchoolCreateRequest.php`
- [X] T039 [US1] Implement school creation persistence for profile fields, required address, institutional references, branding defaults, optional logo file, tenant-root behavior, and normalized email in `schoolmaster-backend/app/Services/School/SchoolProfileService.php`
- [X] T040 [US1] Implement create response output for Basic, Address, Institutional, Branding, `document`, numeric `status`, and `logo_path` in `schoolmaster-backend/app/Http/Resources/SchoolResource.php`
- [X] T041 [US1] Wire create route/controller to Form Request, Policy, DTO, service, resource, JSON, and multipart behavior in `schoolmaster-backend/app/Http/Controllers/Api/V1/SchoolController.php`
- [X] T042 [US1] Implement Institutional lookup controller actions that return full seeded option lists in `schoolmaster-backend/app/Http/Controllers/Api/V1/SchoolLookupController.php`
- [X] T043 [US1] Register Institutional lookup routes under `/api/v1/school-lookups/*` in `schoolmaster-backend/routes/api.php`
- [X] T044 [US1] Implement frontend school create/update service methods for JSON create, multipart create with `logo_file`, lookup loading, and documented error normalization in `schoolmaster-frontend/src/modules/schools/services/schoolService.ts`
- [X] T045 [US1] Implement route-local school form composable for create mode, tab state, defaults, dirty tracking, field errors, stale response guards, and selected logo file in `schoolmaster-frontend/src/modules/schools/composables/useSchoolForm.ts`
- [X] T046 [US1] Implement Basic tab fields for `inep_code`, numeric `status`, `name`, `trade_name`, `legal_name`, `document`, `email`, `phone`, `website`, and `description` in `schoolmaster-frontend/src/modules/schools/components/SchoolBasicFields.vue`
- [X] T047 [US1] Implement required Address tab fields using current address field names and numeric-only rules in `schoolmaster-frontend/src/modules/schools/components/SchoolAddressFields.vue`
- [X] T048 [US1] Implement Institutional tab lookup loading, single-select fields, multi-select fields, full option unavailable state, timezone default, and language default in `schoolmaster-frontend/src/modules/schools/components/SchoolInstitutionalFields.vue`
- [X] T049 [US1] Implement Branding tab logo selector, current/no-logo state, validation feedback, and Element Plus `ElColorPicker` controls for `primary_color` and `secondary_color` without alpha in `schoolmaster-frontend/src/modules/schools/components/SchoolBrandingFields.vue`
- [X] T050 [US1] Implement create route/page that assembles tabs, validation summary, inactive-tab error indicators, submit handling, loading/denied/not-found states, and value preservation in `schoolmaster-frontend/src/modules/schools/routes/SchoolCreatePage.vue`

**Checkpoint**: User Story 1 is independently testable as the MVP.

---

## Phase 4: User Story 2 - Edit school without changing immutable document (Priority: P2)

**Goal**: Authorized administrators can edit existing schools through the same tabs while `document` remains read-only and all required Address/Institutional rules still apply.

**Independent Test**: Open existing school edit, verify read-only document, update valid fields across tabs, confirm original document stays unchanged, and verify incomplete address blocks save.

### Tests for User Story 2

- [X] T051 [P] [US2] Run Redocly validation for update school read-only `document`, required address, JSON/multipart update, no-new-logo preservation, logo replacement, and unchanged optimistic-locking behavior against `schoolmaster-specs/api/openapi.yaml` and record update-contract evidence in `schoolmaster-specs/specs/029-school-fields-tabs/quickstart.md`
- [X] T052 [P] [US2] Add backend feature tests for edit with all tabs, changed `document` rejection, omitted/incomplete address rejection, no-new-logo preservation, logo replacement with old-logo cleanup, duplicate identity rejection, and existing simultaneous edit behavior in `schoolmaster-backend/tests/Feature/School/SchoolUpdateTest.php`
- [X] T053 [P] [US2] Add frontend service tests for edit load, JSON update, multipart update, read-only document omission, no-new-logo payload, replacement payload, and backend validation preservation in `schoolmaster-frontend/tests/unit/schools/schoolService.update.spec.ts`
- [X] T054 [P] [US2] Add frontend edit form tests for read-only document display, incomplete address blocking, inactive-tab errors, unsaved-change route guard, stale save ignore, and value preservation in `schoolmaster-frontend/tests/unit/schools/SchoolEditForm.spec.ts`

### Implementation for User Story 2

- [X] T055 [US2] Implement backend update request validation for read-only `document`, required address on every edit, editable Basic fields, Institutional fields, Branding colors, logo file, uniqueness excluding current school, case-insensitive email, soft-deleted uniqueness scope, numeric status, and legacy `cnpj` rejection in `schoolmaster-backend/app/Http/Requests/Api/V1/SchoolUpdateRequest.php`
- [X] T056 [US2] Implement backend update persistence for editable fields, required address repair, institutional references, color defaults, no-new-logo preservation, logo replacement, old-logo cleanup after successful storage, and existing update conflict behavior in `schoolmaster-backend/app/Services/School/SchoolProfileService.php`
- [X] T057 [US2] Update backend resource/detail behavior so edit load returns all Basic, Address, Institutional, Branding, `document`, numeric status, and current `logo_path` fields in `schoolmaster-backend/app/Http/Resources/SchoolResource.php`
- [X] T058 [US2] Wire detail/update controller behavior to policy checks, update Form Request, service update method, resource output, JSON requests, multipart requests, and documented error responses in `schoolmaster-backend/app/Http/Controllers/Api/V1/SchoolController.php`
- [X] T059 [US2] Implement frontend edit load and submit mapping with read-only `document`, address completion checks, no-new-logo preservation, logo replacement, and stale save ignore in `schoolmaster-frontend/src/modules/schools/composables/useSchoolForm.ts`
- [X] T060 [US2] Implement edit route/page that reuses the tab components, renders read-only document, handles incomplete address saves, displays current logo, and preserves values on errors in `schoolmaster-frontend/src/modules/schools/routes/SchoolEditPage.vue`
- [X] T061 [US2] Register or update frontend school create/edit routes and route guards for dirty state, permission denied, not-found, tenant mismatch, and school context changes in `schoolmaster-frontend/src/modules/schools/routes/index.ts`

**Checkpoint**: User Stories 1 and 2 both work independently.

---

## Phase 5: User Story 3 - Keep contracts and existing school behavior aligned (Priority: P3)

**Goal**: Product, frontend, and backend teams share one synchronized contract for fields, tabs, validation, payload shape, create/edit differences, and unchanged surrounding behavior.

**Independent Test**: Review spec and OpenAPI to confirm all field groups, rules, defaults, lookup sources, upload behavior, and create/edit differences are documented before implementation proceeds.

### Tests for User Story 3

- [X] T062 [P] [US3] Run Redocly contract validation and capture output for school create/update/read, lookup endpoints, multipart upload, validation errors, and no remaining `cnpj` schema in `schoolmaster-specs/specs/029-school-fields-tabs/quickstart.md`
- [X] T063 [P] [US3] Add backend contract/resource shape tests proving create, detail, update, and list responses match OpenAPI field names and exclude `cnpj` in `schoolmaster-backend/tests/Feature/School/SchoolContractTest.php`
- [X] T064 [P] [US3] Add frontend contract mapping tests proving services submit only documented fields, never submit `cnpj`, never submit string statuses, and never hardcode Institutional labels in `schoolmaster-frontend/tests/unit/schools/schoolContractMapping.spec.ts`

### Implementation for User Story 3

- [X] T065 [US3] Update OpenAPI aggregate metadata and tags for active school fields tabs contract in `schoolmaster-specs/api/openapi.yaml`
- [X] T066 [US3] Document coordinated OpenAPI/backend/frontend merge and release gates for the breaking `/api/v1` school contract in `schoolmaster-specs/specs/029-school-fields-tabs/quickstart.md`
- [X] T067 [US3] Remove or replace all remaining `cnpj` references in school OpenAPI paths and schemas in `schoolmaster-specs/api/paths/schools/`
- [X] T068 [US3] Remove or replace all remaining `cnpj` references in school OpenAPI component schemas in `schoolmaster-specs/api/components/schemas/schools/`
- [X] T069 [US3] Update feature quickstart evidence checklist with actual OpenAPI, PHPUnit, Vitest, manual validation, and coordinated rollout gate outputs in `schoolmaster-specs/specs/029-school-fields-tabs/quickstart.md`
- [X] T070 [US3] Confirm no backend code path still accepts legacy `cnpj`, string status, nullable address, frontend-hardcoded lookup labels, or undocumented logo fields in `schoolmaster-backend/app/`
- [X] T071 [US3] Confirm no frontend code path still submits legacy `cnpj`, string status, nullable address, frontend-hardcoded lookup labels, or undocumented logo fields in `schoolmaster-frontend/src/modules/schools/`

**Checkpoint**: All user stories are independently functional and contract-aligned.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final verification, documentation, and release readiness across repositories.

- [X] T072 [P] Run OpenAPI lint and resolve any school contract issues in `schoolmaster-specs/api/openapi.yaml`
- [X] T073 [P] Run backend PHPUnit school test suite and resolve failures in `schoolmaster-backend/tests/Feature/School/`
- [X] T074 [P] Run frontend Vitest school test suite and resolve failures in `schoolmaster-frontend/tests/unit/schools/`
- [X] T075 [P] Capture manual desktop and mobile screenshots or notes for create, edit, validation, lookup, and logo upload flows in `schoolmaster-specs/specs/029-school-fields-tabs/quickstart.md`
- [X] T076 Review tenant isolation, authorization denial, private storage path redaction, upload sanitization, and cross-tenant existence leakage across `schoolmaster-backend/app/`
- [X] T077 Review frontend responsive layout, keyboard operation, inactive-tab error discoverability, dirty-state guard, and text fit across `schoolmaster-frontend/src/modules/schools/`
- [X] T078 Update implementation notes and any changed task evidence in `schoolmaster-specs/specs/029-school-fields-tabs/tasks.md`
- [X] T079 Time a representative authorized-admin school create flow and record whether all required Basic, Address, and Institutional fields can be completed in under 5 minutes in `schoolmaster-specs/specs/029-school-fields-tabs/quickstart.md`
- [ ] T080 Run a representative administrator field-location check and record whether at least 90% of participants locate required school profile fields without assistance in `schoolmaster-specs/specs/029-school-fields-tabs/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- Phase 1 Setup has no dependencies.
- Phase 2 Foundational depends on Setup and blocks all user stories.
- Phase 3 User Story 1 depends on Foundational and is the MVP.
- Phase 4 User Story 2 depends on Foundational; it can begin after shared edit/load contracts exist, but validates best after User Story 1 service/resource patterns are in place.
- Phase 5 User Story 3 depends on Foundational and should run continuously as contract verification while US1/US2 are implemented; coordinated rollout gate task T066 must pass before any dependent repository merges the breaking contract.
- Phase 6 Polish depends on desired user stories being complete.

### User Story Dependencies

- User Story 1 (P1): No dependency on other stories after Foundation.
- User Story 2 (P2): No functional dependency on US3, but reuses form/resource patterns from US1.
- User Story 3 (P3): Can run after Foundation and must finish before cross-repository merge.

### Within Each User Story

- Write and run contract/PHPUnit/Vitest tests first; verify they fail before implementation.
- Complete persistence/model updates before service logic.
- Complete service logic before controller/resource wiring.
- Complete frontend services before composables and route pages.
- Validate each story independently before starting lower-priority polish.

---

## Parallel Opportunities

- Setup tasks T003-T006 can run in parallel.
- Foundational OpenAPI lookup files T010-T018 can run in parallel after directory setup.
- Backend lookup seed/model tasks T022-T025 can run in parallel after migrations are agreed.
- Frontend type/service/error utility tasks T030-T032 can run in parallel.
- US1 tests T033-T037 can run in parallel.
- US1 tab component tasks T046-T049 can run in parallel after `useSchoolForm.ts` interfaces are stable.
- US2 tests T051-T054 can run in parallel.
- US3 tests T062-T064 can run in parallel.
- Polish verification tasks T072-T075 can run in parallel.

---

## Parallel Example: User Story 1

```bash
# Contract/backend/frontend tests can be authored together:
Task T033: Redocly create-contract validation against openapi.yaml with quickstart evidence
Task T034: Backend create feature tests in SchoolCreateTest.php
Task T036: Frontend create service tests in schoolService.create.spec.ts
Task T037: Frontend create form tests in SchoolCreateForm.spec.ts

# Vue tab fields can be implemented together after form model is stable:
Task T046: SchoolBasicFields.vue
Task T047: SchoolAddressFields.vue
Task T048: SchoolInstitutionalFields.vue
Task T049: SchoolBrandingFields.vue
```

## Parallel Example: User Story 2

```bash
# Tests can be authored together:
Task T052: Backend update feature tests in SchoolUpdateTest.php
Task T053: Frontend update service tests in schoolService.update.spec.ts
Task T054: Frontend edit form tests in SchoolEditForm.spec.ts

# Backend and frontend update work can proceed after request/resource contracts are stable:
Task T055: SchoolUpdateRequest.php
Task T056: SchoolProfileService.php
Task T059: useSchoolForm.ts
Task T060: SchoolEditPage.vue
```

## Parallel Example: User Story 3

```bash
# Contract alignment checks can run together:
Task T062: Redocly contract validation evidence
Task T063: Backend contract/resource shape tests
Task T064: Frontend contract mapping tests

# Cleanup scans can run by repository:
Task T067: school paths cleanup
Task T068: school schema cleanup
Task T070: backend source cleanup
Task T071: frontend source cleanup
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 Setup.
2. Complete Phase 2 Foundational contract/storage/module prerequisites.
3. Complete Phase 3 User Story 1.
4. Validate create workflow through OpenAPI, PHPUnit, Vitest, and manual create checks.
5. Stop for review before edit workflow expansion.

### Incremental Delivery

1. Foundation: OpenAPI, storage, lookup seeds, backend service boundaries, frontend module shell.
2. US1: school create with all tabs, defaults, validations, lookups, and logo upload.
3. US2: school edit with immutable document, required address repair, logo replacement, and existing update conflict semantics.
4. US3: contract/resource/service alignment, coordinated rollout gates, and no legacy `cnpj` or string status behavior.
5. Polish: full regression, screenshots/notes, security and accessibility review.

### Team Parallel Strategy

1. One engineer owns OpenAPI and specs updates.
2. One backend engineer owns migrations, requests, services, resources, policies, and PHPUnit.
3. One frontend engineer owns services, composables, tabs, routes, and Vitest.
4. Contract owner reconciles all implementation differences before merge.

## Notes

- `[P]` tasks use separate files or can be done after shared interfaces are stable.
- `[US1]`, `[US2]`, and `[US3]` labels map directly to spec user stories.
- Tests are required because this feature changes critical REST, backend, and frontend behavior.
- Do not add migration/backfill requirements for existing data; data reset is only pre-rollout/seed cleanup scope.
- Do not add new optimistic-locking behavior for simultaneous edits.
- Do not accept legacy `cnpj` payloads or emit `cnpj` responses.

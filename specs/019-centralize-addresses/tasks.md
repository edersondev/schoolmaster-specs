# Tasks: Centralize Addresses

**Input**: Design documents from `specs/019-centralize-addresses/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`,
`contracts/centralize-addresses-contract.md`, `quickstart.md`

**Tests**: Required. This feature changes OpenAPI contracts, Laravel API
behavior, tenant-owned data, validation, lifecycle/removal behavior, migration,
and frontend consumption.

**Organization**: Tasks are grouped by user story to enable independent
implementation and testing. Paths are repository-relative and include the
target repository name when outside the current backend repository.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because the task changes different files and
  does not depend on another incomplete task.
- **[Story]**: Maps the task to a user story from `spec.md`.
- Setup and Foundational phases have no story label.

## Phase 1: Setup

**Purpose**: Prepare contract and implementation file locations before feature
work starts.

- [X] T001 Create shared address schema directory with placeholder index notes in `schoolmaster-specs/api/components/schemas/addresses/.gitkeep`
- [X] T002 [P] Create backend address DTO directory marker in `schoolmaster-backend/app/DTOs/Addresses/.gitkeep`
- [X] T003 [P] Create backend address service directory marker in `schoolmaster-backend/app/Services/Addresses/.gitkeep`
- [X] T004 [P] Create backend address feature test directory marker in `schoolmaster-backend/tests/Feature/Schools/.gitkeep`
- [X] T005 [P] Create backend address unit test directory marker in `schoolmaster-backend/tests/Unit/Addresses/.gitkeep`
- [X] T006 [P] Create frontend school address test directory marker in `schoolmaster-frontend/tests/unit/admin-system/administration/address/.gitkeep`

---

## Phase 2: Foundational

**Purpose**: Contract, persistence, shared model, validation, resource, and
service foundations that block all user stories.

**CRITICAL**: No user story work begins before this phase is complete.

- [X] T007 Add reusable `Address` response schema with digit-only `number` and `zip_code` in `schoolmaster-specs/api/components/schemas/addresses/Address.yaml`
- [X] T008 Add reusable `AddressInput` request schema without `id` and with required structured fields in `schoolmaster-specs/api/components/schemas/addresses/AddressInput.yaml`
- [X] T009 Update school response schema to replace `address_summary` with nullable structured `address` in `schoolmaster-specs/api/components/schemas/schools/School.yaml`
- [X] T010 Update school create request schema to accept optional structured `address` and reject `address_summary` in `schoolmaster-specs/api/components/schemas/schools/SchoolCreateRequest.yaml`
- [X] T011 Update school update request schema to support omitted `address`, structured address replacement, and explicit `address: null` removal in `schoolmaster-specs/api/components/schemas/schools/SchoolUpdateRequest.yaml`
- [X] T012 Verify authenticated session school embedding uses the updated `School` schema in `schoolmaster-specs/api/components/schemas/auth/AuthSession.yaml`
- [X] T013 Mirror the structured address school contract changes in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [X] T014 Run Redocly validation for the updated school address contract and record the result in `schoolmaster-specs/specs/019-centralize-addresses/quickstart.md`
- [X] T015 Create addresses table migration with UUID id, structured fields, direct `school_id`, `$table->morphs('addressable')`, generated active-owner marker or equivalent MySQL-safe uniqueness, timestamps, and soft deletes in `schoolmaster-backend/database/migrations/2026_06_26_000001_create_addresses_table.php`
- [X] T016 Removed as unnecessary: development data can be deleted and recreated, so no address migration exceptions table is required
- [X] T017 Create migration for safely dropping `schools.address_summary` without backfill in `schoolmaster-backend/database/migrations/2026_06_26_000003_retire_school_address_summary_column.php`
- [X] T018 [P] Create Address model with owner relationship and soft deletes in `schoolmaster-backend/app/Models/Address.php`
- [X] T019 Removed as unnecessary: no migration exception model is required for development reset
- [X] T020 [P] Create Address factory with valid digit-only defaults in `schoolmaster-backend/database/factories/AddressFactory.php`
- [X] T021 Removed as unnecessary: no migration exception factory is required for development reset
- [X] T022 Add address relationship to School in `schoolmaster-backend/app/Models/School.php`
- [X] T023 Create AddressData DTO for coordinated structured address input in `schoolmaster-backend/app/DTOs/Addresses/AddressData.php`
- [X] T024 Create AddressResource for response shaping in `schoolmaster-backend/app/Http/Resources/AddressResource.php`
- [X] T025 Create AddressValidationRules helper for required fields and digit-only constraints in `schoolmaster-backend/app/Services/Addresses/AddressValidationRules.php`
- [X] T026 Create SchoolAddressService for create, replace, omitted no-op, explicit null removal, and tenant-safe owner handling in `schoolmaster-backend/app/Services/Addresses/SchoolAddressService.php`

**Checkpoint**: OpenAPI contract, persistence, model, DTO, resource, and shared
service foundation are ready.

---

## Phase 3: User Story 1 - Maintain Structured School Addresses (Priority: P1) MVP

**Goal**: Authorized administrators create, review, update, validate, and remove
structured school addresses without `address_summary`.

**Independent Test**: Create a school with a structured address, retrieve it,
replace one address field, submit invalid numeric fields, remove it with
`address: null`, and confirm responses never depend on `address_summary`.

### Tests for User Story 1

- [X] T027 [P] [US1] Add OpenAPI contract assertions for school address request and response schemas in `schoolmaster-specs/specs/019-centralize-addresses/contracts/centralize-addresses-contract.md`
- [X] T028 [P] [US1] Add feature tests for create, detail, list, update replacement, omitted-address no-op, explicit null removal, duplicate active-owner rejection, and response shape in `schoolmaster-backend/tests/Feature/Schools/SchoolAddressManagementTest.php`
- [X] T029 [P] [US1] Add feature tests for required address fields, digit-only `number` and `zip_code`, optional `country`, optional `complement`, and rejected `address_summary` in `schoolmaster-backend/tests/Feature/Schools/SchoolAddressValidationTest.php`
- [X] T030 [P] [US1] Add feature tests for school address authorization, direct `school_id` tenant scoping, and cross-school denial without address-existence leakage in `schoolmaster-backend/tests/Feature/Schools/SchoolAddressAuthorizationTest.php`
- [X] T031 [P] [US1] Add unit tests for create, replace, omitted no-op, explicit null removal, soft-delete behavior, and internal recovery eligibility in `schoolmaster-backend/tests/Unit/Addresses/SchoolAddressServiceTest.php`

### Implementation for User Story 1

- [X] T032 [US1] Update school create request validation to accept structured address and reject `address_summary` in `schoolmaster-backend/app/Http/Requests/Schools/StoreSchoolRequest.php`
- [X] T033 [US1] Update school update request validation for structured address replacement, omitted no-op, and explicit null removal in `schoolmaster-backend/app/Http/Requests/Schools/UpdateSchoolRequest.php`
- [X] T034 [US1] Update school service create/update flows to call SchoolAddressService in `schoolmaster-backend/app/Services/SchoolService.php`
- [X] T035 [US1] Update SchoolResource to emit structured `address` and remove `address_summary` in `schoolmaster-backend/app/Http/Resources/SchoolResource.php`
- [X] T036 [US1] Update AuthSessionResource to preserve updated embedded School shape in `schoolmaster-backend/app/Http/Resources/AuthSessionResource.php`
- [X] T037 [US1] Update school model eager loading or query usage for address response efficiency in `schoolmaster-backend/app/Services/SchoolService.php`
- [X] T038 [US1] Update school management API assertions away from `address_summary` in `schoolmaster-backend/tests/Feature/SchoolManagementApiTest.php`
- [X] T039 [US1] Run focused backend tests with `docker exec schoolmaster-backend-app-1 php artisan test --compact --filter=SchoolAddress` and record the result in `schoolmaster-specs/specs/019-centralize-addresses/quickstart.md`

**Checkpoint**: User Story 1 is independently functional and is the MVP.

---

## Phase 4: User Story 2 - Reuse Address Handling for Other Records (Priority: P2)

**Goal**: The address model and backend boundaries are reusable for future
addressable records while exposing only School ownership in this slice.

**Independent Test**: Verify address ownership is defined once, School is the
only approved owner, no standalone address endpoint exists, and unauthorized
owners cannot access or mutate address data.

### Tests for User Story 2

- [X] T040 [P] [US2] Add unit tests proving only School is an approved address owner in `schoolmaster-backend/tests/Unit/Addresses/AddressOwnerRegistryTest.php`
- [X] T041 [P] [US2] Add feature tests proving no standalone address routes are registered in `schoolmaster-backend/tests/Feature/Schools/StandaloneAddressRouteAbsenceTest.php`
- [X] T042 [P] [US2] Add unit tests for owner-bound authorization delegation in `schoolmaster-backend/tests/Unit/Addresses/AddressAuthorizationBoundaryTest.php`

### Implementation for User Story 2

- [X] T043 [US2] Create AddressOwnerRegistry that approves School as the only owner for this slice in `schoolmaster-backend/app/Services/Addresses/AddressOwnerRegistry.php`
- [X] T044 [US2] Update SchoolAddressService to validate owner type through AddressOwnerRegistry in `schoolmaster-backend/app/Services/Addresses/SchoolAddressService.php`
- [X] T045 [US2] Update route definitions to avoid standalone address endpoints while keeping address behavior under school operations in `schoolmaster-backend/routes/api.php`
- [X] T046 [US2] Document future owner extension rules and blocked standalone endpoints in `schoolmaster-specs/specs/019-centralize-addresses/contracts/centralize-addresses-contract.md`

**Checkpoint**: Address ownership is reusable by design, but only School is
exposed and testable in this slice.

---

## Phase 5: User Story 3 - Drop Legacy Address Summary During Development Reset (Priority: P3)

**Goal**: Legacy `schools.address_summary` is removed from the development
schema without preserving existing data.

**Independent Test**: Run migrations from scratch and verify the public school
API exposes structured `address` only.

### Tests for User Story 3

- [X] T047 Removed as unnecessary: no legacy summary parser is required
- [X] T048 Removed as unnecessary: no legacy summary preservation test is required
- [X] T049 Covered by structured address response tests in `schoolmaster-backend/tests/Feature/Schools/SchoolAddressManagementTest.php`

### Implementation for User Story 3

- [X] T050 Removed as unnecessary: no legacy summary parser service is required
- [X] T051 Wire direct legacy summary column removal into `schoolmaster-backend/database/migrations/2026_06_26_000003_retire_school_address_summary_column.php`
- [X] T052 Removed as unnecessary: no migration exception DTO is required
- [X] T053 Update quickstart migration verification commands and expected outcomes in `schoolmaster-specs/specs/019-centralize-addresses/quickstart.md`

**Checkpoint**: Legacy school address summaries are removed from the
development schema.

---

## Phase 6: Frontend Follow-Up

**Purpose**: Update frontend school administration after OpenAPI and backend
behavior are contract-compliant.

- [X] T054 [P] Add frontend contract tests for structured school address mapping and removed `addressSummary` usage in `schoolmaster-frontend/tests/unit/admin-system/administration/address/schools-address.contract.spec.js`
- [X] T055 [P] Add frontend service tests for create/update address object, omitted no-op, explicit null removal, and validation mapping in `schoolmaster-frontend/tests/unit/admin-system/administration/address/schools-address.service.spec.js`
- [X] T056 [P] Add frontend component tests for required address fields, digit-only number and zip code errors, optional country/complement, and remove-address intent in `schoolmaster-frontend/tests/unit/admin-system/administration/address/SchoolAddressForm.spec.js`
- [X] T057 Update school frontend contract mappers from `addressSummary` to structured `address` in `schoolmaster-frontend/src/contracts/admin-system/schools.js`
- [X] T058 Update school frontend service payload/response mapping for structured address and explicit null removal in `schoolmaster-frontend/src/services/admin-system/schools.js`
- [X] T059 Update school form component to render structured address fields and remove legacy summary input in `schoolmaster-frontend/src/components/admin-system/schools/SchoolForm.vue`
- [X] T060 Update school list/table display to render structured address summary from approved fields without using legacy payload fields in `schoolmaster-frontend/src/components/admin-system/schools/SchoolTable.vue`
- [X] T061 Update school create/edit page composition for address validation display and explicit remove-address intent in `schoolmaster-frontend/src/pages/admin-system/schools/CreateSchoolPage.vue`

---

## Phase 7: Polish & Cross-Cutting Validation

**Purpose**: Final contract, backend, frontend, documentation, and traceability
checks across the full feature.

- [X] T062 [P] Run final Redocly validation with `npx @redocly/cli lint specs/api/openapi.yaml` and record the result in `schoolmaster-specs/specs/019-centralize-addresses/quickstart.md`
- [X] T063 [P] Run focused backend address tests with `docker exec schoolmaster-backend-app-1 php artisan test --compact --filter=SchoolAddress` and record the result in `schoolmaster-specs/specs/019-centralize-addresses/quickstart.md`
- [X] T064 Run full backend test suite with `docker exec schoolmaster-backend-app-1 php artisan test --compact` and record the result in `schoolmaster-specs/specs/019-centralize-addresses/quickstart.md`
- [X] T065 Run frontend school address unit tests with `npm run test:unit -- --run tests/unit/admin-system/administration/address` and record the result in `schoolmaster-specs/specs/019-centralize-addresses/quickstart.md`
- [X] T066 [P] Verify no `address_summary` or `addressSummary` references remain in public contracts or school frontend/backend request/response code in `schoolmaster-specs/api/`, `schoolmaster-backend/app/`, and `schoolmaster-frontend/src/`
- [X] T067 [P] Review tenant-safe denial and authorization coverage for address operations in `schoolmaster-backend/tests/Feature/Schools/SchoolAddressAuthorizationTest.php`
- [X] T068 Update completion notes, commands, and affected operation IDs in `schoolmaster-specs/specs/019-centralize-addresses/quickstart.md`
- [X] T069 Record timed acceptance evidence for SC-003 create/update under 2 minutes in `schoolmaster-specs/specs/019-centralize-addresses/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: Starts immediately.
- **Phase 2 Foundational**: Depends on Setup; blocks all user stories.
- **Phase 3 US1**: Depends on Foundational; MVP and primary backend behavior.
- **Phase 4 US2**: Depends on Foundational and should reuse US1 address service behavior.
- **Phase 5 US3**: Depends on Foundational and benefits from US1 response/resource behavior.
- **Phase 6 Frontend Follow-Up**: Depends on OpenAPI and backend behavior from US1.
- **Phase 7 Polish**: Depends on selected implementation phases.

### User Story Dependencies

- **US1 Maintain Structured School Addresses**: No story dependency after foundation.
- **US2 Reuse Address Handling for Other Records**: Can start after foundation, but service integration depends on the shared address service used by US1.
- **US3 Preserve Address History During Migration**: Can start after foundation, but full response validation depends on US1 resource shape.

### Within Each Story

- Contract and tests before implementation.
- Models and migrations before services.
- Services before request/controller/resource wiring.
- Backend behavior before frontend consumption.
- Story checkpoint before moving to the next priority when working sequentially.

## Parallel Opportunities

- Setup markers T002-T006 can run in parallel.
- OpenAPI schema work T007-T013 can be split by file after address schema ownership is agreed.
- Backend foundational model/factory tasks T018-T021 can run in parallel.
- US1 tests T027-T031 can run in parallel.
- US2 tests T040-T042 can run in parallel.
- US3 tests T047-T049 can run in parallel.
- Frontend tests T054-T056 can run in parallel after backend contract readiness.
- Final validation tasks T062-T063 and review tasks T066-T067 can run in parallel after implementation.

## Parallel Example: User Story 1

```bash
Task: "T028 Add feature tests for create/detail/list/update replacement/omitted no-op/null removal"
Task: "T029 Add validation tests for required fields, digit-only fields, optional fields, and rejected address_summary"
Task: "T030 Add authorization and cross-school denial tests"
Task: "T031 Add unit tests for SchoolAddressService lifecycle behavior"
```

## Parallel Example: User Story 2

```bash
Task: "T040 Add approved-owner registry tests"
Task: "T041 Add no-standalone-route feature tests"
Task: "T042 Add owner-bound authorization tests"
```

## Parallel Example: User Story 3

```bash
Task: "T047 Add summary migration service unit tests"
Task: "T048 Add migration preservation feature tests"
Task: "T049 Add migrated response shape feature tests"
```

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 Setup.
2. Complete Phase 2 Foundational.
3. Complete Phase 3 US1.
4. Validate school create/read/update/remove with structured address.
5. Stop before frontend follow-up unless backend contract is complete.

### Incremental Delivery

1. Foundation establishes OpenAPI and backend primitives.
2. US1 delivers the user-visible structured school address flow.
3. US2 hardens reusable ownership boundaries without exposing new owners.
4. US3 migrates legacy data safely.
5. Frontend follow-up consumes the completed contract and backend behavior.

### Parallel Team Strategy

1. Specs owner completes OpenAPI schema tasks T007-T014.
2. Backend owner completes persistence and service foundation T015-T026.
3. Test owners write US1-US3 tests in parallel after foundation.
4. Frontend owner starts Phase 6 only after OpenAPI/backend behavior is stable.

## Notes

- Keep contract work before backend exposure.
- Do not create standalone address endpoints.
- Do not approve non-school address owners in this slice.
- Keep `number` and `zip_code` as digit-only strings, not integers.
- Treat omitted `address` and explicit `address: null` differently.
- Development data reset allows dropping legacy summaries without preservation.

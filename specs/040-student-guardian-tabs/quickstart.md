# Quickstart: Student Guardian Tabs

Use this checklist to implement or review feature 040.

## 1. Confirm Scope

- Read `spec.md`, `plan.md`, `research.md`, `data-model.md`, and
  `contracts/student-guardian-tabs-contract.md`.
- Keep Guardians absent from the administration sidebar.
- Keep Create Student as the only user-facing guardian capture entry point.
- Allow student creation with zero, one, or two guardians.
- Support both new guardian creation and existing same-school guardian linking.
- Allow duplicate relationship labels for the two guardian entries.
- Require guardian management authority before a user can capture guardians.
- Do not add packages unless separately approved.

## 2. Backend Contract Gate

Before backend implementation, update and validate OpenAPI:

- `POST /api/v1/student-profiles` supports zero to two
  `guardian_associations`.
- Each guardian entry accepts either an existing active same-school
  `guardian_id` or new guardian fields.
- New guardian fields are limited to approved identity/contact data.
- `relationship_type` is required for every guardian entry.
- Duplicate relationship labels are accepted when guardian identities or
  references are otherwise valid.
- Student-with-guardians create is atomic.
- `GET /api/v1/guardians` remains the only existing-guardian lookup source for
  this feature.
- Standalone `POST /api/v1/guardians` is not used by the frontend as a chained
  step in this workflow.

Suggested contract checks from `schoolmaster-specs`:

```bash
npx @redocly/cli lint api/openapi.yaml
npx @redocly/cli bundle api/openapi.yaml --output /tmp/feature-040-openapi.yaml
rg "maxItems: 2|guardian_associations|StudentProfileGuardianInput|createStudentProfile|listGuardians" api
```

## 3. Backend Implementation Gate

In `schoolmaster-backend`:

- Create matching branch `040-student-guardian-tabs`.
- Update student profile create validation for zero to two guardian entries.
- Validate each entry uses exactly one mode: existing guardian or new guardian.
- Validate existing guardian references are active and same-school.
- Validate guardian management authority when guardian entries are present.
- Create student, any new guardians, and associations in one transaction.
- Roll back the whole operation when any student or guardian validation fails.
- Shape success through approved resources.
- Keep controllers thin; place workflow logic in services.

Suggested backend checks:

```bash
php artisan test
```

Focused tests should cover zero, one, and two guardians, mixed new/existing
entries, duplicate references, third guardian rejection, missing permission,
duplicate relationship label acceptance, tenant mismatch, inactive school,
inactive guardian, invalid contact, invalid relationship, rollback on failure,
and response shape.

## 4. Frontend Implementation Gate

In `schoolmaster-frontend` after backend readiness:

- Create matching branch `040-student-guardian-tabs`.
- Remove Guardians from sidebar navigation metadata.
- Redirect former guardian administration routes to Create Student.
- Convert Create Student to a Student and Guardians tab workflow.
- Keep student fields in the Student tab.
- Let Guardians tab add zero to two entries.
- Support new guardian and existing same-school guardian modes.
- Use `listGuardians` only for existing guardian lookup.
- Submit through the student create service only.
- Keep route-local form state and stale-response protection.
- Show tab-level validation indicators and field errors.
- Block guardian capture for users without guardian management authority while
  leaving student-only creation available.

Suggested frontend checks:

```bash
npm run test:unit
npm run build
rg "axios" src/pages src/components src/composables src/router
rg "/api/v1/" src/pages src/components src/composables src/router
rg "Guardians" src/router src/components src/pages
```

Expected:

- tests and build pass
- no direct Axios outside services/API client
- no endpoint strings in pages/components/composables/router guards
- no visible Guardians sidebar entry
- direct guardian routes redirect without loading guardian list data

## 5. Manual Scenario Review

- Sign in as an administrator with student create authority and guardian
  management authority.
- Confirm Guardians is absent from the sidebar.
- Open Create Student and confirm Student and Guardians tabs.
- Add no guardians, submit valid student data, and verify success.
- Add one new guardian, submit, and verify association display.
- Add two guardians, one new and one existing same-school guardian, submit, and
  verify both associations.
- Add two guardians with the same relationship label and verify submission is
  accepted when both guardian identities are valid.
- Attempt a third guardian and confirm it is blocked before submit.
- Submit invalid student and guardian data and confirm affected tab/field
  feedback while values remain.
- Sign in as a user with student create authority but without guardian
  management authority and confirm student-only creation is available while
  guardian capture is unavailable.
- Open former guardian list/create/detail/lifecycle URLs and confirm redirect
  to Create Student without guardian list data.
- Switch active school during a dirty workflow and confirm existing discard
  behavior and tenant reset.

## 6. Acceptance Evidence

Record in implementation PRs:

- OpenAPI diff showing maximum two guardian entries and new/existing guardian
  entry shape.
- Backend tests proving atomic rollback and permission behavior.
- Backend response-shape verification for created student and guardians.
- Frontend evidence that sidebar excludes Guardians for all permission sets.
- Frontend evidence that direct guardian routes redirect to Create Student.
- Frontend tests for tab state, tab validation, zero/one/two guardian cases,
  blocked third guardian, new/existing guardian modes, and permission gating.
- Diagnostics review proving no protected student, guardian, contact, token,
  permission, full payload, or cross-tenant data appears in visible errors,
  logs, or test output.
- Responsive and keyboard review for 390px, 768px, and 1440px.

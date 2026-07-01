# Implementation Notes: Teacher Workflow Workspace

## Contract Review

- Aggregate OpenAPI reviewed at `specs/api/openapi.yaml`.
- Confirmed approved operation IDs exist for teacher content, questionnaires,
  learning sets, grades, attendance, imports, academic periods, class-section
  memberships, and teacher assignments.
- Confirmed `createLearningSet` still requires direct `student_profile_ids`;
  learning-set create remains blocked.
- Confirmed `listLearningSets`, `listGrades`, and `listAttendance` do not
  document academic-period or roster/class-section filters; scoped lists remain
  blocked.
- Confirmed grade and attendance imports enforce 1 to 500 JSON rows.

## Verification

- `npm run test:unit -- tests/teacher-workflow`: 17 files passed, 31 tests
  passed.
- `npm run test:unit`: 202 files passed, 347 tests passed.
- `npm run build`: passed. Vite reported existing vendor pure-annotation and
  chunk-size warnings.
- `env CI=1 npm run test:e2e -- e2e/teacher-workflow.spec.js --project=chromium`:
  3 tests passed, covering 390px, 768px, and 1440px responsive audits,
  keyboard reachability or read-only states, timed route/workflow budgets, and
  status-language distinguishability.
- `git diff --check`: passed.
- Direct Axios scan in `src/modules/teacher-workflow`: passed, no direct Axios
  imports or calls.
- Unsupported-surface scan found only explicit blocked/absent policy text and
  tests.
- OpenAPI validation skipped because `specs/api/openapi.yaml` was not changed.

## Playwright Evidence

- Responsive and keyboard audit covered teacher content list/detail,
  questionnaire list/detail, learning-set list/detail, grade, attendance,
  admin-observed material, admin academic record, and admin import routes at
  390px, 768px, and 1440px.
- Timed evidence covered learning-set detail under the 2-minute budget,
  content/questionnaire/learning-set workflow under the 6-minute budget, and
  grade validation workflow under the 4-minute budget.
- Status evidence covered active, inactive, deleted, restored, scan-pending,
  scan-failed, unavailable, read-only legacy, conflict, unsupported-contract,
  and JSON-only import states.

# Quickstart: Teacher Workflow Workspace

## Prerequisites

- Feature 016 System Administrator Shell and Dashboard Foundation is
  implemented.
- Feature 017 Authentication and Session Foundation UI is implemented.
- Feature 018 Administration Foundation UI is implemented.
- Feature 020 Administration Lifecycle UI is implemented.
- Feature 021 Account Lifecycle Workflows UI is implemented.
- Feature 022 Student Enrollment and Classroom Roster UI is implemented.
- Backend teacher workflow foundation from
  `specs/004-backend-teacher-workflows/` is deployed and contract-compliant.
- Backend teacher workflow lifecycle and correction behavior from
  `specs/010-teacher-workflow-lifecycle/` is deployed and
  contract-compliant.
- Classroom roster and teacher assignment operations from
  `specs/009-classroom-roster-foundation/` are deployed and
  contract-compliant.
- Implementation confirms approved permission codes or session capability
  flags before teacher workspace, admin observation, correction, import,
  lifecycle, download, and upload action visibility is enabled.

## Contract Review

Before frontend implementation:

1. Confirm these teacher content operations exist in `api/openapi.yaml`:
   - `listTeacherContent`
   - `createTeacherContent`
   - `getTeacherContent`
   - `updateTeacherContent`
   - `updateTeacherContentStatus`
   - `deleteTeacherContent`
   - `restoreTeacherContent`
   - `downloadTeacherContent`
2. Confirm content upload accepts only approved fields and file categories:
   `pdf`, `image`, `text`, `office_document`, maximum 25 MB.
3. Confirm these questionnaire operations exist:
   - `listQuestionnaires`
   - `createQuestionnaire`
   - `getQuestionnaire`
   - `updateQuestionnaire`
   - `updateQuestionnaireStatus`
   - `deleteQuestionnaire`
   - `restoreQuestionnaire`
4. Confirm questionnaire response review, response grading, and response file
   downloads remain excluded from this feature.
5. Confirm these learning-set operations exist:
   - `listLearningSets`
   - `createLearningSet`
   - `getLearningSet`
   - `updateLearningSet`
   - `updateLearningSetStatus`
   - `deleteLearningSet`
   - `restoreLearningSet`
6. Confirm `createLearningSet` supports roster-aware request behavior before
   enabling learning-set create UI. If the operation still requires direct
   `student_profile_ids`, keep create UI blocked and update OpenAPI first.
7. Confirm `listLearningSets` supports documented academic-period and
   roster/class-section filters before implementing scoped learning-set lists.
   Do not send undocumented filters or load broad unscoped lists as a
   workaround.
8. Confirm these grade operations exist:
   - `listGrades`
   - `createGrade`
   - `getGrade`
   - `correctGrade`
   - `updateGradeStatus`
   - `deleteGrade`
   - `restoreGrade`
   - `importGrades`
9. Confirm `listGrades` supports documented academic-period and roster/class-
   section filters before implementing scoped grade lists. Do not send
   undocumented filters.
10. Confirm these attendance operations exist:
   - `listAttendance`
   - `createAttendance`
   - `getAttendance`
   - `correctAttendance`
   - `updateAttendanceStatus`
   - `deleteAttendance`
   - `restoreAttendance`
   - `importAttendance`
11. Confirm `listAttendance` supports documented academic-period and
    roster/class-section filters before implementing scoped attendance lists.
    Do not send undocumented filters.
12. Confirm the approved academic-period, class-section, and teacher-assignment
   operations needed for scope exist:
   - `listAcademicPeriods`
   - `listClassSections`
   - `listTeacherAssignments`
   - `getTeacherAssignment`
13. Confirm grade and attendance correction reasons require 10 to 500
    characters.
14. Confirm grade and attendance imports accept JSON payloads only, contain 1
    to 500 rows, and reject invalid imports all-or-nothing.
15. Confirm validation, unauthorized, forbidden, tenant-mismatch,
    inactive-school, inactive-record, not-found, conflict, scan-unavailable,
    download-denied, import-validation, unsupported filter, unsupported sort,
    unsupported page-size, stale-record, and temporary-unavailable envelopes
    are documented for consumed actions.

## Manual Scenario Review

### Teacher Workspace Scope

- Sign in as an authorized teacher.
- Select or confirm active school context.
- Open teacher workspace root.
- Verify workspace defaults to current active academic period and active
  teacher rosters.
- Change selected academic period or roster and verify route/query persistence
  across refresh and back navigation.
- Verify no-current-active-period and no-active-teacher-roster states block
  scoped loading and show safe feedback.

### Teacher Content

- Open teacher content list.
- Verify only visible same-school content appears.
- Verify pagination, loading, empty, scan status, lifecycle status, and denial
  states.
- Upload valid PDF, image, text, and office document files up to 25 MB.
- Attempt unsupported, oversized, content-type mismatch, failed-scan, and
  unauthorized uploads.
- Open content detail and update approved metadata.
- Attempt edit that changes historical student-facing meaning after use and
  verify conflict feedback.
- Download clean authorized content and verify only transient download metadata
  is used.
- Attempt download for pending-scan, failed-scan, inactive, deleted,
  unauthorized, cross-school, or unavailable content.
- Delete, restore to inactive, activate, and deactivate where approved.

### Questionnaires

- Open questionnaire list.
- Create questionnaire with supported authoring questions for this slice.
- Verify question ordering and field-level validation.
- Open questionnaire detail and update approved fields.
- Attempt unsupported response review or grading routes and verify absence.
- Attempt edit that changes historical student-facing meaning after use and
  verify conflict feedback.
- Delete, restore to inactive, activate, and deactivate where approved.

### Learning Sets

- Open learning-set list with selected academic period and roster context.
- Verify learning-set list remains blocked until approved academic-period and
  roster/class-section filters exist in OpenAPI.
- Create or update a learning set with ordered clean content and active
  questionnaires.
- Verify learning-set create remains blocked if OpenAPI still requires direct
  selected-student assignment writes.
- Verify roster-aware audience display.
- Verify legacy direct selected-student assignments display read-only when
  returned.
- Attempt inactive, deleted, unclean, cross-school, unauthorized, duplicate, or
  dependency-conflicting entries and verify no partial success appears.
- Delete, restore to inactive, activate, and deactivate where approved.

### Grades and Attendance

- Open grade and attendance screens for selected academic period and roster.
- Verify grade and attendance lists remain blocked until approved
  academic-period and roster/class-section filters exist in OpenAPI.
- Record a valid grade from 0 to 100 for an eligible student.
- Record valid attendance using documented statuses.
- Attempt invalid values, ineligible students, inactive roster membership,
  closed period, duplicate record, cross-school target, and unauthorized actor.
- Open detail and submit valid correction reason from 10 to 500 characters.
- Attempt blank, short, long, unauthorized, and teacher closed-period
  corrections.
- Verify school administrator closed-period correction is available only where
  approved.
- Verify correction history redacts private notes and unauthorized actor
  metadata.

### Admin Observation and Imports

- Sign in as an authorized school administrator.
- Open admin-observed teacher workflow views.
- Verify read/detail observation works for same-school records.
- Verify teacher-owned create, update, lifecycle, download, and learning-set
  management controls remain hidden unless contract explicitly grants them.
- Open grade/attendance import screen.
- Submit valid structured JSON grade import with 1 to 500 rows.
- Submit valid structured JSON attendance import with 1 to 500 rows.
- Submit import with one invalid row and verify all-or-nothing rejection and no
  partial success.
- Verify CSV, spreadsheet, archive, and file-upload import controls are absent.
- Verify teacher, student, guardian, platform-without-school-context, and
  unauthorized administrator actors cannot access import controls.

## Automated Verification Expectations

Run in `schoolmaster-frontend` after implementation:

```bash
npm run test:unit
```

Focused Vitest coverage should include:

- teacher content contract mappers and service calls
- questionnaire contract mappers and service calls
- learning-set contract mappers and service calls
- grade and attendance contract mappers and service calls
- teacher workflow import mappers and service calls
- teacher workspace current active period and route/query selected-period
  restoration
- teacher active roster scope and selected-roster restoration
- content upload validation, scan status, and scan-gated availability
- clean authorized download and download-denied mapping
- questionnaire authoring and blocked response/grading surfaces
- learning-set roster-aware audience and read-only legacy direct assignment
  behavior
- grade and attendance create, correction, lifecycle, delete, and restore flows
- correction reason 10-500 character validation
- closed-period teacher denial and school-administrator correction authority
- structured JSON grade and attendance import, 500-row cap, and all-or-nothing
  feedback
- admin-observed read/detail scope and blocked teacher-owned management
  controls
- stale response protection
- validation, forbidden, tenant-mismatch, inactive-school, inactive-record,
  not-found, conflict, unsupported filter, unsupported sort, unsupported
  page-size, scan-unavailable, download-denied, import-validation,
  stale-record, and temporary-unavailable mapping
- no-sensitive-data diagnostics

Run build checks if available:

```bash
npm run build
```

Run OpenAPI validation only if teacher workflow contracts change:

```bash
npx @redocly/cli lint api/openapi.yaml
```

## Acceptance Evidence

Record in implementation PR:

- Operation ID to UI surface mapping.
- Permission code or session capability source used for each teacher and admin
  surface.
- Evidence that current active academic period and active teacher rosters are
  the default teacher workspace scope.
- Evidence that selected period and selected roster persist in route/query
  state.
- Evidence that teacher content upload accepts only approved fields and file
  categories.
- Evidence that private file paths and persisted download URLs do not appear in
  UI state or diagnostics.
- Evidence that questionnaire response review, grading, and response file
  download routes remain absent.
- Evidence that learning-set legacy direct assignments are read-only and new
  writes use approved roster-aware behavior.
- Evidence that learning-set create and scoped learning-set, grade, and
  attendance lists consume only approved OpenAPI filters/request shapes or stay
  blocked until those contracts are added.
- Evidence that grade and attendance correction reason validation uses 10 to
  500 characters and closed-period teacher correction is denied.
- Evidence that admin observation does not expose broader teacher-owned
  management controls.
- Evidence that imports use structured JSON payload only, enforce the 500-row
  cap, and show no partial success for rejected imports.
- Evidence that stale responses do not overwrite route, filter, pagination,
  active-school, selected-period, selected-roster, target, correction, import,
  or permission changes.
- Evidence that student, teacher, guardian, token, permission, role, private
  file path, full upload metadata, full import payload, correction private
  note, and cross-tenant details are absent from diagnostics.
- Responsive and keyboard review for 390px, 768px, and 1440px.

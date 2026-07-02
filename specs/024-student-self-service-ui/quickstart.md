# Quickstart: Student Self-Service UI

## Prerequisites

- Feature 015 Frontend Architecture Baseline is implemented.
- Feature 016 System Administrator Shell and Dashboard Foundation is
  implemented where shared protected shell patterns are required.
- Feature 017 Authentication and Session Foundation UI is implemented,
  including current-user hydration, active school context, unauthorized,
  forbidden, inactive-user, inactive-school, no-active-school, session-expired,
  and tenant-mismatch behavior.
- Backend student and reporting foundation from
  `specs/005-backend-student-reporting/` is deployed and contract-compliant
  for student learning sets, content download, grades, and attendance.
- Implementation confirms how the authenticated session exposes active linked
  student profile and current active academic period before enabling student
  data requests.

## Contract Review

Before frontend implementation:

1. Confirm `api/openapi.yaml` includes `listStudentLearningSets`.
2. Confirm `listStudentLearningSets` requires tenant context,
   `academic_period_id`, page, and per-page only.
3. Confirm `StudentLearningSetTimelineItem` includes `id`,
   `academic_period_id`, `title`, `status`, `published_at`, and `entries`.
4. Confirm `StudentLearningSetEntry` includes `entry_type`,
   `entry_reference_id`, `sequence`, `title`, and optional student content
   metadata.
5. Confirm questionnaire entries in learning sets have no submit/review UI in
   this slice.
6. Confirm `downloadStudentTeacherContent` exists and returns binary content
   for authorized assigned content.
7. Confirm `TeacherContentStudentMetadata` includes `download_available`; use
   that flag to gate the download action.
8. Confirm `listStudentGrades` exists and supports tenant context, optional
   academic-period filter, page, and per-page.
9. Confirm `StudentGradeRecord` includes only student-visible grade fields.
10. Confirm `listStudentAttendance` exists and supports tenant context,
    optional academic-period filter, page, and per-page.
11. Confirm `StudentAttendanceRecord` includes only student-visible attendance
    fields.
12. Confirm no standalone student learning-set, grade, or attendance detail
    operations are approved for this slice.
13. Confirm no student-facing report-run, report download, transcript,
    period-picker source, or academic-summary endpoint is approved for this
    slice.
14. Confirm validation, unauthorized, forbidden where applicable,
    tenant-mismatch, inactive-school, not-found, unsupported page-size, and
    temporary-unavailable envelopes are documented or normalized by existing
    frontend error mapping.

## Manual Scenario Review

### Student Context Gates

- Sign in as an authenticated student with one active school, active linked
  student profile, and current active academic period.
- Open student workspace root.
- Verify Assigned Learning Sets opens by default.
- Sign in with no active school and verify no-active-school state blocks
  student data requests.
- Sign in with active school but no active linked student profile and verify
  no-student-profile state blocks student data requests.
- Sign in with active school and student profile but no current active
  academic period and verify no-current-period state blocks assigned
  learning-set loading.

### Assigned Learning Sets

- Open Assigned Learning Sets.
- Verify current active academic-period timeline loads with pagination.
- Verify empty response shows empty state, not denial.
- Open loaded learning-set detail.
- Verify ordered entries appear in returned sequence.
- Verify questionnaire entries are read-only.
- Attempt direct detail route without loaded record and verify not-found or
  contract-unavailable state without standalone detail request.

### Authorized Content

- Open a learning set with content entries.
- Verify content with `download_available: true` enables download.
- Download clean authorized content.
- Verify pending, failed, inactive, missing, unassigned, unauthorized, and
  cross-school content states show unavailable-content or safe denial.
- Verify private file paths, storage keys, and scan internals are absent from
  UI state, visible errors, and diagnostics.

### Grades

- Open Grades.
- Verify only own student grade records appear.
- Verify pagination, loading, empty, status, and read-only loaded detail.
- Verify no correction, import, restore, teacher, or administrator controls.
- Attempt stale direct detail route and verify not-found without standalone
  detail request.

### Attendance

- Open Attendance.
- Verify only own student attendance records appear.
- Verify pagination, loading, empty, status, and read-only loaded detail.
- Verify no correction, import, restore, teacher, or administrator controls.
- Attempt stale direct detail route and verify not-found without standalone
  detail request.

### Academic Overview

- Open Academic Overview.
- Verify only counts and statuses from approved student responses appear.
- Verify no calculated GPA, attendance rate, ranking, trend, transcript,
  report-run, report-download, or custom-report controls.
- Verify overview respects no-active-school, no-student-profile, and
  no-current-period gates.

## Automated Verification Expectations

Run in `schoolmaster-frontend` after implementation:

```bash
npm run test:unit
```

Focused Vitest coverage should include:

- student service mappers for `listStudentLearningSets`
- student service mappers for `downloadStudentTeacherContent`
- student service mappers for `listStudentGrades`
- student service mappers for `listStudentAttendance`
- no undocumented request parameters
- active school gate before student data requests
- active linked student profile gate before student data requests
- current active academic period gate before assigned learning-set requests
- default Assigned Learning Sets landing
- read-only questionnaire entries
- loaded-list-backed learning-set, grade, and attendance details
- content download enabled only for `download_available`
- unavailable-content, not-found, unauthorized, forbidden, tenant-mismatch,
  inactive-school, validation, unsupported page-size, stale-response, and
  temporary-unavailable mapping
- academic overview counts/statuses only
- stale response protection
- no-sensitive-data diagnostics

Run build checks if available:

```bash
npm run build
```

Run OpenAPI validation only if contracts change:

```bash
npx @redocly/cli lint api/openapi.yaml
```

## Acceptance Evidence

Record in implementation PR:

- Operation ID to UI surface mapping.
- Evidence that student workspace root lands on Assigned Learning Sets.
- Evidence that no-active-school, no-student-profile, and no-current-period
  gates block data requests.
- Evidence that questionnaire entries are read-only.
- Evidence that content download is gated by `download_available`.
- Evidence that detail views do not call undocumented standalone detail
  endpoints.
- Evidence that academic overview uses counts/statuses only.
- Evidence that no private file paths, storage keys, token values, role
  internals, scan internals, guardian data, other-student data, or
  cross-tenant details appear in diagnostics or test output.

## Out of Scope Verification

Confirm none of these appear in the student self-service UI slice:

- manual academic-period switch
- student questionnaire response submit/review
- standalone student learning-set detail API use
- standalone student grade detail API use
- standalone student attendance detail API use
- calculated GPA
- calculated attendance rate
- ranking or trend analytics
- report-run creation, listing, retry, cancel, or download
- transcript UI
- teacher authoring
- administrator management
- guardian self-service
- platform support
- grade or attendance correction
- imports
- restore, purge, legal hold, or anonymization
- billing, messaging, or notification-center behavior

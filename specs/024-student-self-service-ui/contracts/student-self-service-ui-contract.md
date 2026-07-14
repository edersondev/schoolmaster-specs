# UI Contract: Student Self-Service UI

## Purpose

This contract maps Student Self-Service UI surfaces to approved OpenAPI
operations, frontend state boundaries, and blocked behavior. It is a frontend
consumption contract, not a backend API change.

## Approved Operations

| UI surface | Operation ID | Method/path | Required context |
|------------|--------------|-------------|------------------|
| Assigned Learning Sets | `listStudentLearningSets` | `GET /api/v1/student/learning-sets` | Authenticated session, active school, active linked student profile, current active academic period |
| Student content download | `downloadStudentTeacherContent` | `GET /api/v1/student/teacher-content/{contentItemId}/download` | Authenticated session, active school, returned assigned content item with `download_available` |
| Grades | `listStudentGrades` | `GET /api/v1/student/grades` | Authenticated session, active school, active linked student profile |
| Attendance | `listStudentAttendance` | `GET /api/v1/student/attendance` | Authenticated session, active school, active linked student profile |

## Route Surfaces

| Route intent | Behavior |
|--------------|----------|
| Student workspace root | Redirect or render Assigned Learning Sets after session, school, student profile, and current period are confirmed |
| Assigned Learning Sets | Load current active academic-period timeline with pagination |
| Learning-set detail | Use already loaded timeline item; no standalone request |
| Content download | Call approved binary download operation only when `download_available` is true |
| Grades | Load own records with pagination and required current active academic period |
| Grade detail | Use already loaded grade record; no standalone request |
| Attendance | Load own records with pagination and required current active academic period |
| Attendance detail | Use already loaded attendance record; no standalone request |
| Academic overview | Display counts/statuses from loaded approved responses only |

## Service Boundary

All HTTP access must go through student service modules. Components and route
views must not call Axios directly.

Service functions:

- `listAssignedLearningSets({ schoolId, academicPeriodId, page, perPage })`
- `downloadAssignedContent({ schoolId, contentItemId })`
- `listStudentGrades({ schoolId, academicPeriodId, page, perPage })`
- `listStudentAttendance({ schoolId, academicPeriodId, page, perPage })`

Mapping requirements:

- Submit only documented parameters.
- Parse paginated envelopes through shared pagination mappers.
- Normalize error envelopes into safe feedback states.
- Keep binary download handling transient.
- Drop undocumented response fields.

## UI State Contract

Student surfaces must distinguish:

- loading
- empty
- unauthorized
- forbidden
- tenant-mismatch
- inactive-school
- no-active-school
- no-student-profile
- no-current-period
- unavailable-content
- validation
- not-found
- unsupported page-size
- temporary-unavailable
- stale-response

True empty states must not be shown for missing school, missing profile,
missing current period, denial, unavailable content, validation, or not-found.

## Capability Gates

System Administrator master access does not create student impersonation or a
selected-subject transport. These identity-owned operations continue to require
the authenticated actor's active linked student profile. A missing actor-owned
profile remains a non-permission denial.

- No student data request before active school is confirmed.
- No student data request before active linked student profile is confirmed.
- No assigned learning-set request before current active academic period is
  confirmed.
- No grade, attendance, or academic overview data request before current active
  academic period is confirmed.
- No content download action unless returned metadata says
  `download_available` is true.
- No manual academic-period switch.
- No report-run, report-download, transcript, custom-report, ranking, trend,
  GPA, or attendance-rate controls.
- No questionnaire response submit/review controls.
- No teacher, administrator, guardian, platform, correction, import, restore,
  purge, messaging, notification-center, billing, or undocumented controls.

## Loaded-List Detail Contract

Learning-set, grade, and attendance details in this slice are not independent
API resources.

Rules:

- Opening detail from a loaded list uses the loaded record.
- Direct route access without a matching loaded record shows not-found or
  contract-unavailable state.
- Implementation must not call `/api/v1/learning-sets/{id}`,
  `/api/v1/grades/{id}`, or `/api/v1/attendance/{id}` for student detail.

## Academic Overview Contract

Allowed:

- Assigned learning-set count.
- Downloadable content count.
- Unavailable content count.
- Grade lifecycle/status counts.
- Attendance status counts.

Blocked:

- Calculated GPA.
- Calculated attendance rate.
- Rankings.
- Trend analytics.
- Transcripts.
- Report runs.
- Report output downloads.
- Custom report filters.

## Sensitive Data Contract

UI state, diagnostics, visible errors, and test output must not include:

- private file paths
- storage keys
- token values
- role internals
- scan internals
- guardian data
- other-student data
- cross-tenant details
- raw binary file data

Allowed diagnostics:

- operation ID
- generic state kind
- field label for validation
- current route name
- safe correlation/request ID if provided by shared error normalization

## Verification Contract

Implementation must include Vitest coverage for:

- approved operation mapping
- no undocumented parameter submission
- no data requests before active school/profile/period gates
- default Assigned Learning Sets landing
- read-only questionnaire entries
- unavailable-content gating
- loaded-list-backed details
- academic overview counts/statuses only
- denial and not-found state mapping
- stale-response protection
- safe diagnostics redaction

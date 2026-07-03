# UI Contract: Guardian Self-Service UI

## Purpose

This contract maps Guardian Self-Service UI surfaces to approved OpenAPI
operations, frontend state boundaries, and blocked behavior. It is a frontend
consumption contract, not a backend API change.

## Approved Operations

| UI surface | Operation ID | Method/path | Required context |
|------------|--------------|-------------|------------------|
| Linked students | `listGuardianStudents` | `GET /api/v1/guardian/students` | Authenticated session, active school, active guardian-user link, active guardian record |
| Linked student detail | `getGuardianStudent` | `GET /api/v1/guardian/students/{studentProfileId}` | Authenticated session, active school, active guardian-user link, active guardian-student association |
| Academic summary | `getGuardianStudentAcademics` | `GET /api/v1/guardian/students/{studentProfileId}/academics` | Authenticated session, active school, active guardian-student association, current active academic period |
| Contact view | `getGuardianStudentContacts` | `GET /api/v1/guardian/students/{studentProfileId}/contacts` | Authenticated session, active school, active guardian-student association |

## Route Surfaces

| Route intent | Behavior |
|--------------|----------|
| Guardian workspace root | Redirect or render Linked Students after session and school context are confirmed |
| Linked Students | Load guardian-visible student collection with pagination |
| Student Detail | Load limited profile and enrollment summary for selected linked student |
| Academic Summary | Load current active academic-period summary for selected linked student |
| Contact View | Load guardian-owned and student primary contact values for selected linked student |
| Direct target route | Show safe not-found for missing, unassociated, inactive, transferred, deleted, or cross-tenant target student |

## Service Boundary

All HTTP access must go through guardian service modules. Components and route
views must not call Axios directly.

Service functions:

- `listGuardianStudents({ schoolId, page, perPage })`
- `getGuardianStudent({ schoolId, studentProfileId })`
- `getGuardianStudentAcademics({ schoolId, studentProfileId, academicPeriodId })`
- `getGuardianStudentContacts({ schoolId, studentProfileId })`

Mapping requirements:

- Submit only documented parameters.
- Parse paginated envelopes through shared pagination mappers.
- Parse success envelopes through guardian contract mappers.
- Normalize error envelopes into safe feedback states.
- Drop undocumented response fields.
- Preserve uniform target-specific not-found behavior.

## UI State Contract

Guardian surfaces must distinguish:

- loading
- empty
- unauthorized
- forbidden
- tenant-mismatch
- inactive-school
- no-active-school
- no-guardian-link, only from approved safe session or access behavior
- no-linked-students
- no-academic-period
- unavailable-summary
- validation
- not-found
- unsupported page-size
- temporary-unavailable
- stale-response

True empty states must not be shown for missing school, missing guardian link,
missing academic period, denial, validation, target not-found, unavailable
summary, or stale response.

No-guardian-link must not be inferred from generic unauthorized, forbidden,
tenant-mismatch, validation, not-found, or temporary-unavailable responses.

## Capability Gates

- No guardian data request before active school is confirmed.
- No no-guardian-link state unless approved session or access behavior safely
  identifies a missing guardian-user link.
- No target-specific guardian request before a selected student profile ID is
  present from route or approved UI action.
- No academic summary request before current active academic period is
  confirmed.
- No manual academic-period switch.
- No free-form academic period query parameter.
- No guardian write, profile update, association request, or guardian-user-link
  provisioning controls.
- No detailed grade rows, detailed attendance rows, correction history,
  teacher content downloads, questionnaire answers, student activity
  submission, report-run, report-download, transcript, ranking, custom-report,
  messaging, notification-center, school-admin, student self-service, platform,
  restore, purge, legal hold, billing, or undocumented controls.

## Target Not-Found Contract

Target-specific student routes must preserve backend non-enumeration.

Rules:

- Missing student returns not-found UI.
- Unassociated student returns same not-found UI.
- Inactive student returns same not-found UI.
- Transferred or deleted student returns same not-found UI.
- Cross-tenant student returns same not-found UI.
- Visible copy and diagnostics must not distinguish these conditions.

## Academic Summary Contract

Allowed:

- Current active academic-period grade summary.
- Current active academic-period attendance summary.
- Current active academic-period learning-set title, status, progress, and last
  activity values returned by contract.
- Safe missing-value rendering for null summary values.

Blocked:

- Manual period switching.
- Client-side academic-period selection from arbitrary URL/query values.
- Detailed grade records.
- Detailed attendance records.
- Correction history.
- Teacher content.
- Questionnaire answers.
- Student submissions.
- Reports, transcripts, rankings, trends, or custom report filters.

## Contact View Contract

Allowed:

- Authenticated guardian's own contact fields returned by contract.
- School-approved relationship label.
- Selected student's primary school-approved contact details returned by
  contract.
- Safe missing-value rendering for null contact values.

Blocked:

- Other guardian records.
- Non-primary student contacts.
- Restricted emergency handling details.
- Custody, legal, or dispute records.
- School-only notes.
- Contact edit controls.
- Guardian profile update controls.

## Sensitive Data Contract

UI state, diagnostics, visible errors, and test output must not include:

- unassociated student identifiers
- other guardian records
- non-primary contacts
- restricted emergency handling details
- school-only notes
- custody or legal details
- correction details
- teacher-private data
- report data
- token values
- role internals
- cross-tenant details
- raw backend denial reasons

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
- no data requests before active school gate
- linked student list pagination and no-linked-students empty state
- guardian student detail field visibility
- current active academic period gate before academic summary
- no manual period picker
- contact view field visibility and missing-value rendering
- target-specific not-found non-enumeration
- no-guardian-link only from approved safe signal
- unavailable-summary mapping
- denial and validation state mapping
- stale-response protection
- safe diagnostics redaction

# Quickstart: Guardian Self-Service UI

## Prerequisites

- Feature 015 Frontend Architecture Baseline is implemented.
- Feature 016 System Administrator Shell and Dashboard Foundation is
  implemented where shared protected shell patterns are required.
- Feature 017 Authentication and Session Foundation UI is implemented,
  including current-user hydration, active school context, unauthorized,
  forbidden, inactive-user, inactive-school, no-active-school, session-expired,
  and tenant-mismatch behavior.
- Feature 024 Student Self-Service UI is complete and available as a reference
  for protected self-service state handling, stale-response protection, and
  no-sensitive-data diagnostics.
- Backend guardian self-service from `specs/011-guardian-self-service/` is
  deployed and contract-compliant for guardian student list, student detail,
  academic summary, and contact view.
- Implementation confirms how authenticated session or approved access behavior
  exposes active school context, safely identified missing guardian-user link,
  guardian access failures, and current active academic period before enabling
  guardian academic summary requests.

## Contract Review

Before frontend implementation:

1. Confirm `api/openapi.yaml` includes `listGuardianStudents`.
2. Confirm `listGuardianStudents` requires tenant context, page, and per-page
   only.
3. Confirm `GuardianStudentSummary` includes `id`, `school_id`,
   `registration_number`, `full_name`, `status`, `relationship_label`,
   optional `enrolled_at`, and optional `current_academic_year_id`.
4. Confirm `api/openapi.yaml` includes `getGuardianStudent`.
5. Confirm `GuardianStudentDetail` includes only guardian-visible profile and
   enrollment summary fields.
6. Confirm target-specific missing, unassociated, inactive, transferred,
   deleted, and cross-tenant target students return the same not-found
   envelope.
7. Confirm `api/openapi.yaml` includes `getGuardianStudentAcademics`.
8. Confirm `getGuardianStudentAcademics` requires tenant context, target
   student UUID, and explicit academic-period filter.
9. Confirm frontend source for guardian academic summary uses current active
   same-school academic period only.
10. Confirm `GuardianAcademicSummary` includes `student`,
    `academic_period_id`, `grade_summary`, `attendance_summary`, and
    `learning_sets`.
11. Confirm no detailed grade, attendance, correction, teacher content,
    questionnaire, report, transcript, ranking, or custom-report operations are
    approved for guardian UI in this slice.
12. Confirm `api/openapi.yaml` includes `getGuardianStudentContacts`.
13. Confirm `GuardianContactView` includes only authenticated guardian contact,
    relationship label, and student primary contact fields.
14. Confirm no guardian write, profile update, association request, manual
    academic-period picker, notification, messaging, or guardian-user-link
    provisioning UI is approved for this slice.
15. Confirm validation, unauthorized, forbidden where applicable,
    tenant-mismatch, inactive-school, not-found, unsupported page-size, and
    temporary-unavailable envelopes are documented or normalized by existing
    frontend error mapping.

## Component Boundary Review

- Route views remain composition surfaces.
- Guardian services own all HTTP access and contract mapping.
- Guardian composables coordinate school context, selected student, current
  academic period, pagination, stale-response protection, and safe feedback.
- Components receive mapped data through props and emit user intent.
- Element Plus component tags remain PascalCase.
- Display text is centralized through Vue I18n.
- Tailwind handles layout and spacing around Element Plus primitives.

## Manual Scenario Review

### Guardian Context Gates

- Sign in as an authenticated guardian with one active school and active
  guardian-user link.
- Open guardian workspace root.
- Verify Linked Students opens by default.
- Sign in with no active school and verify no-active-school state blocks
  guardian data requests.
- Sign in without active guardian-user link where approved session or access
  behavior safely identifies it and verify no-guardian-link blocks
  guardian-specific data requests.
- Sign in without active guardian-user link where only a generic denial
  envelope is available and verify the UI follows that envelope without
  inferring no-guardian-link.
- Verify guardian navigation is visibly distinct from student self-service.

### Linked Students

- Open Linked Students.
- Verify guardian-visible student list loads with pagination.
- Verify only active same-school linked students appear.
- Verify relationship labels and limited summary fields render.
- Verify empty response shows no-linked-students, not generic denial.
- Verify active associations in another school do not appear under current
  school context.

### Linked Student Detail

- Select a student from Linked Students.
- Verify limited profile and enrollment summary appears.
- Verify school-only notes, other guardian data, detailed academics, report
  output, teacher-private data, and unapproved fields are absent.
- Attempt direct routes for missing, unassociated, inactive, transferred,
  deleted, and cross-tenant targets.
- Verify every target-specific denial renders the same safe not-found state.

### Academic Summary

- Open Academic Summary for a linked student with current active academic
  period.
- Verify grade summary, attendance summary, and learning-set summary values
  render from the approved response.
- Verify no manual academic-period picker appears.
- Sign in or configure context with no current active academic period and
  verify no-academic-period blocks the academic summary request.
- Verify null or absent summary values show safe missing or unavailable-summary
  states.
- Verify detailed grade rows, attendance rows, correction history, teacher
  content, questionnaire answers, reports, transcripts, rankings, and custom
  report controls are absent.

### Contact View

- Open Contact View for a linked student.
- Verify authenticated guardian contact fields, relationship label, and
  student primary school-approved contact fields render.
- Verify null contact values show safe missing-value state.
- Verify other guardian records, non-primary contacts, restricted emergency
  handling details, custody details, legal details, school-only notes, and
  contact edit controls are absent.

### Stale Response and Diagnostics

- Start a guardian list, detail, academic, or contact request, then change
  route, selected student, active school, academic period, authentication, or
  session state before response applies.
- Verify stale response does not overwrite current visible state.
- Verify visible errors, diagnostics, and test output omit unassociated student
  identifiers, other guardian records, non-primary contacts, school-only notes,
  correction details, teacher-private data, report data, token values, role
  internals, raw denial reasons, and cross-tenant details.

## Automated Verification Expectations

Run in `schoolmaster-frontend` after implementation:

```bash
npm run test:unit
```

Focused Vitest coverage should include:

- guardian service mappers for `listGuardianStudents`
- guardian service mappers for `getGuardianStudent`
- guardian service mappers for `getGuardianStudentAcademics`
- guardian service mappers for `getGuardianStudentContacts`
- no undocumented request parameters
- active school gate before guardian data requests
- no-guardian-link only from approved safe session or access behavior
- generic denial behavior without no-guardian-link inference
- linked student list pagination and no-linked-students state
- guardian student detail field visibility
- uniform target-specific not-found behavior
- current active academic period gate before academic summary requests
- no manual academic-period picker
- academic summary values and unavailable-summary mapping
- contact view field visibility and missing-value rendering
- unauthorized, forbidden, tenant-mismatch, inactive-school, validation,
  unsupported page-size, stale-response, and temporary-unavailable mapping
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
- Evidence that guardian workspace root lands on Linked Students.
- Evidence that no-active-school and safely identified no-guardian-link gates
  block data requests where applicable.
- Evidence that generic denials do not infer no-guardian-link.
- Evidence that no-linked-students is distinct from denial.
- Evidence that academic summary uses current active academic period only.
- Evidence that target-specific denials render the same not-found state.
- Evidence that contact view exposes only approved contact field groups.
- Evidence that no unassociated student identifiers, other guardian records,
  non-primary contacts, school-only notes, correction details, teacher-private
  data, report data, token values, role internals, raw denial reasons, or
  cross-tenant details appear in diagnostics or test output.

## Out of Scope Verification

Confirm none of these appear in the guardian self-service UI slice:

- manual academic-period switch
- free-form academic period URL/query source
- guardian writes
- guardian profile/contact updates
- guardian association requests
- guardian-user-link provisioning
- detailed grade rows
- detailed attendance rows
- correction history
- teacher content downloads
- questionnaire answers or student activity submission
- report-run creation, listing, retry, cancel, or download
- transcript UI
- messaging
- notification-center behavior
- school administration
- student self-service downloads or activities
- teacher workflows
- platform support
- imports
- restore, purge, legal hold, anonymization, custody, or legal workflows
- billing, payment, payroll, or accounting

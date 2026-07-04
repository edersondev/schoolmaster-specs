# Frontend Manual Test Plan

This checklist covers the implemented frontend feature set documented in
[frontend-feature-roadmap.md](frontend-feature-roadmap.md). Use it for a
complete manual pass in `schoolmaster-frontend` against a contract-compliant
backend.

## Test Run Record

| Field | Value |
|-------|-------|
| Tester |  |
| Date |  |
| Frontend commit |  |
| Backend commit |  |
| Specs commit |  |
| Browser and version |  |
| Viewports tested | 390px, 768px, 1440px |
| Result | Pass / Fail / Blocked |

For each scenario, record `Pass`, `Fail`, or `Blocked`, plus screenshots or
notes for failures. A failed authorization, validation, tenant, stale-response,
or unsupported-action check is a product defect if protected data appears or an
undocumented control is available.

## Required Test Accounts

- Platform administrator with platform support permissions.
- School administrator with school administration, reporting, and assessment
  permissions.
- Teacher assigned to at least one class section and learning set.
- Student with active enrollment, assigned learning sets, grades, attendance,
  and advanced assessments.
- Guardian linked to at least one student.
- User with limited or no permissions for denial checks.
- Inactive, locked, or expired-token user for account/session checks.

## Required Seed Data

- At least two schools to verify tenant isolation.
- Users, roles, permissions, academic years, academic periods, guardians,
  students, enrollments, class sections, rosters, and teacher assignments.
- Published and draft teacher content, questionnaires, learning sets, grades,
  attendance, correction history, downloads, and JSON import fixtures.
- Student and guardian academic summaries.
- Report catalog entries, report definitions, report runs, downloadable report
  outputs, and custom report definitions.
- Platform support access requests, approvals, revocations, suppressed
  cross-school summaries, and support audit entries.
- Advanced questionnaires with existing, `long_text`, and `file_response`
  questions; submitted responses with pending, clean, failed, unavailable,
  graded, ungraded, exempt, and conflict states.

## Global Regression Checks

- [ ] Application loads without console errors on initial visit.
- [ ] Authenticated route refresh preserves session, active actor, permissions,
  and active school context.
- [ ] Protected routes deny unauthenticated users and return to intended route
  after login where approved.
- [ ] Login submits only email and password; stale persisted school context
  must not add `school_id` to the `/api/v1/auth/login` payload.
- [ ] Sidebar, top header, breadcrumbs or page titles, actions, loading states,
  empty states, and error states stay aligned across implemented modules.
- [ ] Permission-hidden navigation entries do not appear for unauthorized
  users.
- [ ] Direct URL access to unauthorized or cross-tenant records shows safe
  denied or not-found feedback without protected record details.
- [ ] Forms show field-level validation and do not submit invalid payloads.
- [ ] Pagination, filters, sorting, refresh, and clear-filter behavior preserve
  documented contract semantics.
- [ ] Stale responses do not overwrite the visible screen after route, active
  school, filter, permission, or selected-record changes.
- [ ] Diagnostics and visible error output omit tokens, full payloads, private
  IDs, storage paths, answer keys, private notes, and cross-tenant identifiers.
- [ ] Keyboard navigation reaches all actionable controls in a logical order.
- [ ] Responsive layouts at 390px, 768px, and 1440px have no horizontal
  overflow, clipped text, overlapping controls, or unusable dialogs.

## Role-Based Complete Flow Tests

These flows group the implemented features by the role that exercises them.
Feature references map back to `specs/015` through `specs/028` and
[frontend-feature-roadmap.md](frontend-feature-roadmap.md).

## Platform Administrator Flow

Primary coverage: admin shell, authentication, reporting oversight, platform
support, tenant-safe denial, privacy boundaries.

- [ ] Sign in as platform administrator and verify current-user, permissions,
  session context, shell navigation, header, user menu, and responsive sidebar.
- [ ] Open platform oversight and verify minimized cross-school summaries,
  small-count suppression, redacted details, loading states, and empty states.
- [ ] Review support access requests and verify approval, expiry, revocation,
  opt-in, denied-access, and audit states.
- [ ] Open support audit review and verify filters, detail, pagination, empty
  states, denied states, and stale-response handling.
- [ ] Open reporting surfaces allowed to platform users and verify only
  approved report catalog, request, history, detail, lifecycle, custom
  definition, and download behavior appears.
- [ ] Attempt direct access to school-owned student, guardian, teacher,
  assessment response, file, and private report routes; verify safe denied or
  not-found feedback without detailed answer/file access.
- [ ] Switch or remove active school context where possible and verify tenant
  data does not load before an approved context exists.
- [ ] Verify diagnostics, support views, report views, and audit details omit
  student answer text, uploaded files, private file metadata, storage paths,
  private grading notes, tokens, full payloads, and cross-tenant identifiers.
- [ ] Sign out and verify protected platform URLs return to authentication
  handling.

## School Administrator Flow

Primary coverage: admin shell, school administration, lifecycle actions,
account lifecycle, enrollment/roster, teacher-observed workflows, reporting,
advanced assessment review, tenant isolation.

- [ ] Sign in as school administrator and verify admin shell, active school,
  permissions, navigation visibility, dashboard cards, quick actions, loading
  states, and responsive behavior.
- [ ] List schools, users, roles, permissions, academic years, academic
  periods, and guardians where approved; verify filters, pagination, empty
  states, and list refresh.
- [ ] Create approved administration resources with valid data and verify
  success state, validation mapping, and refreshed list state.
- [ ] Create a school with CNPJ entered as `56.563.930/0001-08`; verify the
  frontend sends unmasked digits, backend validates check digits and
  uniqueness, and list/detail views display the masked CNPJ.
- [ ] Submit invalid create forms and verify field-level validation with no
  malformed request accepted.
- [ ] Open supported administration detail pages and update editable fields;
  verify status tags and refreshed detail/list state.
- [ ] Activate, deactivate, soft-delete, restore, and run approved bulk
  lifecycle actions; verify confirm dialogs, conflict handling, partial
  failure summaries, and constrained schools/permissions behavior.
- [ ] Send or inspect account invitation, password setup, reset, inactive,
  locked, and reactivation workflows where available; verify safe token and
  recovery messaging.
- [ ] Create or open student profiles, enrollments, class sections, roster
  membership, guardian links, and teacher assignments; verify academic-period
  scoping, duplicate handling, inactive states, transfer behavior, and
  conflicts.
- [ ] Open same-school teacher-observed content, questionnaire, grades,
  attendance, correction history, downloads, and JSON import surfaces where
  approved; verify unsupported teacher-only workflows are absent.
- [ ] Open reporting workspace and verify authorized report catalog, requests,
  history, detail, lifecycle, custom definitions, downloads, active-school
  timezone rendering, retention visibility, unavailable outputs, and
  diagnostics redaction.
- [ ] Create or update an advanced questionnaire with existing, `long_text`,
  and `file_response` questions where authoring permission is granted; verify
  schema controls, lifecycle locks, unsupported type blocking, and validation.
- [ ] Open advanced assessment review queue and verify only documented filters
  are used: `questionnaire_id`, `learning_set_id`, and `grading_status`.
- [ ] Review and grade authorized same-school `long_text` and clean
  `file_response` answers from 0 to 100; verify failed-scan zero/exempt
  actions, clean download-only behavior, no inline preview, conflict handling,
  and stale-response protection.
- [ ] Attempt direct access to another school's administration, roster,
  reporting, teacher, student, guardian, and assessment records; verify safe
  denied or not-found feedback without protected existence leaks.
- [ ] Sign out and verify protected school administrator URLs require login.

## Teacher Flow

Primary coverage: teacher workspace, content, questionnaires, learning
workflow surfaces, downloads/imports, advanced assessment authoring/review,
manual grading.

- [ ] Sign in as assigned teacher and verify teacher navigation, active school,
  class/learning context, and denied state for unrelated schools.
- [ ] Open teacher workspace and verify content list, detail, create, update,
  loading, empty, validation, unavailable, and denied states.
- [ ] Open questionnaire list/detail/create/update workflows and verify existing
  question behavior remains unchanged.
- [ ] Open learning-set, grade, attendance, correction history, downloads, and
  JSON import surfaces that are implemented; verify unsupported or
  undocumented workflows are absent.
- [ ] Upload or import valid and invalid JSON files and verify progress,
  validation, success, failure, and safe diagnostics.
- [ ] Create or update an advanced questionnaire with existing, `long_text`,
  and `file_response` questions; verify approved schema controls, grading
  expectations, file rules, lifecycle lock notices, and unsupported type
  blocking.
- [ ] Open assigned advanced assessment review queue and verify filters,
  pagination, response state, grading state, scan state, and safe student
  identity fields.
- [ ] Open response detail and verify authorized `long_text` answer content is
  visible only in review/grading surfaces.
- [ ] Download clean answer files and verify inline preview is absent.
- [ ] Verify pending, failed, unavailable, inactive, deleted, unauthorized, and
  stale file states hide download actions.
- [ ] Grade eligible answers from 0 to 100 with student-visible feedback; verify
  invalid score, conflict, stale, forbidden, and cross-tenant states.
- [ ] Apply failed-scan zero-score and exempt actions where authorized; verify
  these actions are hidden for pending, clean, unavailable, or unauthorized
  files.

## Student Flow

Primary coverage: authentication, student self-service, assigned work, student
submission, advanced assessment results, authorization denial.

- [ ] Sign in as active student and verify student-only navigation, session
  context, active school, loading states, and responsive layout.
- [ ] Open assigned learning sets and verify unavailable content, missing
  assignments, empty states, and inactive student states.
- [ ] View own grades, attendance, authorized content, academic summaries, and
  approved reporting views.
- [ ] Open an assigned advanced questionnaire response before due date.
- [ ] Enter valid `long_text` answers and verify local-only draft behavior.
- [ ] Select one allowed PDF, image, text, or office file under 25 MB for each
  `file_response` question and verify no upload happens before final submit.
- [ ] Submit before due date and verify submitted state, pending scan
  visibility, and no resubmission action.
- [ ] Attempt blank, whitespace-only, over-10,000-character, invalid encoding,
  unsafe control-character, unsupported file, oversized file, multiple file,
  unsafe filename, mismatched category, duplicate, after-due-date, unassigned,
  inactive, cross-tenant, stale, and partial-success submissions.
- [ ] Open own advanced assessment result and verify only submission status,
  grading status, score summary, student-visible feedback summary, and safe
  file availability metadata appear.
- [ ] Verify private grading notes, hidden answer keys, storage paths, raw file
  contents, scanner internals, and other students' responses are hidden.
- [ ] Attempt direct access to another student's records across student,
  teacher, guardian, reporting, and assessment routes; verify safe denied or
  not-found feedback.

## Guardian Flow

Primary coverage: guardian self-service, linked student summaries, contact
views, tenant-safe denial, blocked unsupported capabilities.

- [ ] Sign in as guardian and verify guardian navigation, session state,
  loading states, and responsive layout.
- [ ] Open linked student list and verify only approved linked students appear.
- [ ] Open linked student detail and verify approved current-period academic
  summary and contact views.
- [ ] Verify inactive link, missing association, cross-tenant, and not-found
  states do not enumerate protected records.
- [ ] Verify unsupported guardian capabilities are absent, including advanced
  assessment answer/file visibility unless a later approved contract adds it.
- [ ] Attempt direct access to unlinked student, teacher, school
  administration, reporting, platform, and assessment routes; verify safe
  denied or not-found feedback.
- [ ] Verify visible summaries and diagnostics omit private student data,
  answer text, uploaded files, storage paths, private grading notes, and
  unsupported records.

## Reporting User Flow

Primary coverage: reporting workspace, custom definitions, downloads,
advanced assessment aggregate fields, privacy boundaries.

- [ ] Sign in as reporting user and verify reporting navigation, active school,
  and denied state for non-reporting modules.
- [ ] Open report catalog and verify only authorized reports appear.
- [ ] Request reports with valid and invalid parameters; verify validation,
  loading, success, unavailable, denied, and failed states.
- [ ] View report run history, detail, lifecycle state, retry or cancellation
  where approved, and downloadable output where available.
- [ ] Create, update, or use custom report definitions where approved; verify
  ownership, validation, conflict, and unsupported action behavior.
- [ ] Verify active-school timezone rendering and retention visibility.
- [ ] Verify advanced assessment fields expose only counts, completion status,
  grading status, and score summaries.
- [ ] Confirm raw answer text, uploaded files, file links, private metadata,
  storage paths, private grading notes, unsupported joins, arbitrary query
  text, and cross-tenant references are blocked from reporting.
- [ ] Change report filters or route while requests are pending and verify
  stale responses do not overwrite current state.

## Limited, Inactive, and Unauthenticated User Flow

Primary coverage: authorization boundaries, account/session edge cases, safe
denial across all implemented modules.

- [ ] Attempt login with invalid credentials, inactive user, locked account,
  inactive school, malformed input, expired session, and revoked token; verify
  safe feedback.
- [ ] Use forgot-password and reset-password entry points with existing,
  unknown, expired, malformed, and already-used token cases; verify approved
  non-enumerating messages.
- [ ] Sign in as limited user and verify navigation hides unauthorized admin,
  teacher, student, guardian, reporting, platform, and assessment entries.
- [ ] Directly open protected URLs for all implemented modules and verify
  unauthorized, forbidden, tenant-mismatch, not-found, stale, and unavailable
  states do not expose protected record details.
- [ ] Verify forms, actions, downloads, imports, lifecycle controls, grading
  controls, report requests, and support actions are disabled or absent when
  permissions are missing.
- [ ] Sign out, refresh protected URLs, and verify authentication handling
  returns to the approved login flow.

## Final Acceptance

- [ ] All applicable feature sections pass.
- [ ] All failures have issue links or documented follow-up.
- [ ] No undocumented API behavior is required to complete the manual pass.
- [ ] No unsupported controls appear for payroll, billing, messaging, live
  classroom, mobile-native behavior, public file sharing, legal hold,
  anonymization, purge, raw answer reporting, or platform/support detailed
  answer/file access.
- [ ] Screenshots or notes are attached for responsive, keyboard,
  authorization, stale-response, and privacy checks.

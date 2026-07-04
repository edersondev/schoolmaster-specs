# Quickstart: Advanced Assessment Frontend UX

## Prerequisites

- Feature 015 Frontend Architecture Baseline is implemented.
- Feature 017 Authentication and Session Foundation UI is implemented.
- Feature 023 Teacher Workflow Workspace is implemented where questionnaire
  and learning-set patterns are required.
- Feature 024 Student Self-Service UI is implemented where student-owned
  surfaces are required.
- Feature 026 Reporting Workspace UI is implemented where report catalog and
  result patterns are required.
- Backend feature `014-advanced-assessment-content` is implemented and
  contract-compliant.
- `api/openapi.yaml` and
  `specs/001-schoolmaster-platform/contracts/openapi.yaml` include approved
  advanced assessment operations before frontend runtime consumption.

## Contract Review

Before frontend implementation:

1. Confirm `createQuestionnaire`, `updateQuestionnaire`, and
   `getQuestionnaire` expose approved `long_text` and `file_response`
   schemas.
2. Confirm unsupported advanced question types remain rejected.
3. Confirm questionnaire lifecycle lock state is documented for historical
   student-facing meaning.
4. Confirm `submitStudentQuestionnaireResponse` accepts final assessment
   submission with text answers and file-response files.
5. Confirm no backend draft response or staged file upload operation is
   consumed by this UI slice.
6. Confirm long-text limits are 1 to 10,000 characters and blank or
   whitespace-only answers are rejected.
7. Confirm file-response limits are one PDF, image, text, or office file per
   question, maximum 25 MB.
8. Confirm file-response scan states and unavailable-file responses are
   documented.
9. Confirm `getStudentQuestionnaireResponse` returns only student-safe own
   result fields.
10. Confirm `listQuestionnaireResponses` and `getQuestionnaireResponse` return
    teacher/admin review fields without private storage paths or scanner
    internals.
11. Confirm `gradeQuestionnaireResponse` supports 0-100 manual grading and
    failed-scan zero/exempt outcomes only.
12. Confirm `downloadQuestionnaireResponseFile` returns only clean authorized
    answer files and supports denied, not-found, conflict, scan-pending, and
    scan-failed states.
13. Confirm no inline file preview contract is required or consumed.
14. Confirm report catalog/report definition/report result contracts expose
    only assessment counts, completion status, grading status, and score
    summaries.
15. Confirm guardian, platform support, raw answer reporting, generated report
    file packaging, public file URLs, legal hold, anonymization, purge,
    messaging, notification, and billing behavior remain outside this slice.

## Component Boundary Review

- Route views remain composition surfaces.
- Advanced assessment services own all HTTP access and contract mapping.
- Advanced assessment composables coordinate active school, permission gates,
  authoring state, local-only text drafts, final-submit file selection,
  response submission, review lists, grading state, file availability, student
  result state, report aggregate state, stale-response protection, and safe
  feedback.
- Components receive mapped data through props and emit user intent.
- Element Plus component tags remain PascalCase.
- Display text is centralized through Vue I18n.
- Tailwind handles layout and spacing around Element Plus primitives.

## Manual Scenario Review

### Authoring

- Sign in as an authorized same-school teacher or school administrator.
- Create a questionnaire with existing question types, one `long_text`
  question, and one `file_response` question.
- Verify only approved schema fields, grading expectations, and file rules
  appear.
- Attempt unsupported question types and malformed schemas.
- Verify field validation blocks submission.
- Open a lifecycle-locked questionnaire.
- Verify historical-meaning edits are blocked safely.

### Student Submission

- Sign in as a student with an assigned same-school advanced questionnaire.
- Enter a valid long-text answer.
- Select one approved file under 25 MB for a file-response question.
- Verify the file is not uploaded before final submit.
- Submit before the due date.
- Verify submitted state, pending scan visibility, and no resubmission action.
- Attempt blank, whitespace-only, over-limit, unsupported file, oversized file,
  multiple files, after-due-date, duplicate, unassigned, and stale submissions.
- Verify safe feedback and no partial answer state.

### Review and Grading

- Sign in as an owning or assigned teacher, or same-school school
  administrator with assessment review authority.
- Open the review queue.
- Verify server-side filters are limited to documented contract fields
  (`questionnaire_id`, `learning_set_id`, and `grading_status` in the current
  contract), while returned response states, grading states, scan states, empty
  states, and denied states display safely.
- Open a clean file-response answer.
- Verify download action is available and inline preview is absent.
- Open pending, failed, unavailable, inactive, deleted, and unauthorized file
  states.
- Verify download is hidden.
- Verify authorized graders can view long-text answer content in the grading
  surface and grade long-text and clean file-response answers from 0 to 100.
- For failed-scan file-response answers, verify only zero-score and exempt
  actions appear.
- Verify stale/conflict grading feedback does not overwrite current state.

### Student Results

- Sign in as the owning student.
- Open submitted advanced assessment results.
- Verify submission status, grading status, score summary, student-visible
  teacher feedback summary, and safe file availability metadata.
- Verify private grading notes, hidden answer keys, storage paths, and file
  contents do not appear.
- Sign in as another student and verify safe denial or not-found behavior.

### Reporting

- Sign in as a reporting user.
- Open report catalog, report definitions, report requests, report history,
  and report results where advanced assessment fields are approved.
- Verify only assessment counts, completion status, grading status, and score
  summaries are available.
- Verify raw answer text is blocked from reports, while uploaded files, file
  links, private metadata, storage paths, private grading notes, unsupported
  joins, arbitrary query text, and cross-tenant references are blocked.

### Stale Response and Diagnostics

- Start authoring, response submission, review, grading, download, result, or
  reporting requests, then change route, active school, selected
  questionnaire, selected response, selected student, report filters,
  authentication, permissions, due date, scan state, or grading state before
  the response applies.
- Verify stale responses do not overwrite current visible state.
- Verify diagnostics, validation labels, filenames, student/result/report
  surfaces, guardian/platform/support views, and test output omit raw answer
  text outside authorized grading displays, student-visible feedback, or
  another explicitly approved display surface; uploaded file contents, private
  storage paths, hidden answer keys, private grading notes, scanner internals,
  credentials, full payloads, and unauthorized identifiers remain omitted.

## Automated Verification Expectations

Run in `schoolmaster-frontend` after implementation:

```bash
npm run test:unit
```

Focused Vitest coverage should include:

- advanced assessment service mappers for questionnaire create/update/detail
- student response submission mappers for final-submit text/files
- local-only draft behavior with no service request before final submit
- long-text validation states
- file-response selection validation states
- one-attempt and due-date blocking
- scan-pending, scan-failed, unavailable-file, and clean file states
- clean file download action and no inline preview
- response review list/detail state
- manual grading 0-100 validation
- failed-scan zero/exempt actions
- student result privacy
- reporting aggregate-only fields
- authorization and active-school gates
- contract-unavailable states
- stale-response guards
- safe diagnostics and no-sensitive-data assertions

Run contract validation if OpenAPI files change:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

## Acceptance Evidence

Record implementation evidence in the frontend repository once available:

- unit test command and result
- focused service/composable/component test list
- manual responsive review at 390px, 768px, and 1440px
- keyboard review for authoring, submission, upload selection, download,
  grading, and reporting controls
- no-sensitive-data diagnostics review
- confirmation that no undocumented APIs, staged upload, backend draft,
  inline preview, guardian visibility, raw answer reporting, generated report
  file packaging, platform/support detail access, messaging, notification,
  billing, legal hold, anonymization, or purge behavior was added

## Out of Scope Verification

Confirm implementation does not add:

- backend behavior
- staged file upload operations
- backend draft attempts
- resubmission or reopened attempts
- advanced question types beyond `long_text` and `file_response`
- inline file preview
- guardian advanced assessment visibility
- raw answer text or file links in reports
- generated report file packaging for response attachments
- platform/support detailed answer or file access
- messaging, notifications, billing, public file sharing, permanent purge,
  legal hold, anonymization, or undocumented APIs

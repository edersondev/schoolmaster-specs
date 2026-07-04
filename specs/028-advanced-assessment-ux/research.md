# Research: Advanced Assessment Frontend UX

## Decision: Consume approved advanced assessment OpenAPI contracts only

**Rationale**: Backend feature `014-advanced-assessment-content` and
`api/openapi.yaml` define the operation IDs, schemas, states, file rules, and
error envelopes. The frontend must not infer fields, routes, filters, or scan
behavior from implementation details.

**Alternatives considered**:

- Build from backend spec prose only: rejected because OpenAPI is the
  cross-repository source of truth.
- Stub frontend routes before contract readiness: rejected because it can
  freeze undocumented payloads.
- Fold missing contract behavior into frontend planning: rejected because
  contract gaps must be resolved in specs/OpenAPI first.

## Decision: Keep advanced assessment UX school-scoped

**Rationale**: Advanced questionnaires, response attempts, answer files,
grading outcomes, and report aggregates are school-owned records. Keeping the
workspace school-scoped prevents guardian, platform, or support contexts from
appearing to inherit detailed answer access.

**Alternatives considered**:

- Add platform support access to advanced answer details: rejected because
  support access remains minimized and does not grant answer/file detail.
- Add guardian visibility now: rejected because guardian advanced assessment
  visibility is explicitly out of scope.
- Make advanced assessment a global reporting tool: rejected because report
  exposure is aggregate-only.

## Decision: Model authoring around approved schemas and lifecycle locks

**Rationale**: Authoring must show only existing question types plus
`long_text` and `file_response`, with validation and lock state driven by
contracted questionnaire lifecycle behavior. This preserves historical
student-facing meaning after publication, assignment, response, grading, or
report generation.

**Alternatives considered**:

- Let frontend construct free-form question schemas: rejected because it would
  bypass contract validation.
- Hide locked advanced questions: rejected because authors need to understand
  why editing is blocked.
- Duplicate backend validation rules only in UI: rejected because UI
  validation is advisory and backend remains authoritative.

## Decision: Keep unsubmitted text drafts local-only

**Rationale**: The backend contract allows one submitted attempt and excludes
draft attempts, reopened attempts, and attempt versioning. Local-only text
drafts reduce accidental same-device text loss without creating backend
partial response state.

**Alternatives considered**:

- Persist backend drafts: rejected because it contradicts the one-attempt
  contract and adds unapproved response states.
- Disable drafts entirely: rejected because long-text answers are high-friction
  and local recovery improves usability.
- Store file drafts locally: rejected because file-response uploads are
  final-submit only and files should not persist beyond user selection.

## Decision: Upload file-response files only with final submission

**Rationale**: Final-submit-only upload keeps file handling aligned with the
one-attempt rule, avoids staged file cleanup, and prevents backend partial
answer state before submission.

**Alternatives considered**:

- Stage files before submit: rejected because no approved staged-file contract
  exists.
- Allow both staged and final uploads: rejected because it doubles validation
  and stale-state paths.
- Upload files immediately on selection: rejected because it creates backend
  state before the assessment is submitted.

## Decision: Use download-only clean file access

**Rationale**: The approved operation is a clean answer-file download for
authorized review. Avoiding inline preview reduces private-file rendering,
browser plugin, and content sniffing risks.

**Alternatives considered**:

- Preview clean images and PDFs inline: rejected because it expands the UI
  scope beyond the download operation.
- Preview every supported file type: rejected because office/text rendering
  would require more conversion and security behavior.
- Block all file downloads: rejected because authorized reviewers need access
  to clean file-response answers.

## Decision: Represent scan states explicitly

**Rationale**: Pending, failed, unavailable, blocked, and clean file states
drive student, reviewer, grader, and reporting behavior. The UI must not show
download controls or imply file availability before a clean state is
authorized.

**Alternatives considered**:

- Hide file rows until clean: rejected because users need status visibility.
- Treat failed and pending scans the same: rejected because failed scans allow
  zero/exempt grading while pending scans do not.
- Show scanner internals: rejected because scanner details are not part of the
  user contract.

## Decision: Expose zero-score and exempt actions for failed-scan answers

**Rationale**: The backend contract allows failed-scan file-response answers
to be graded as zero or exempt. Exposing both actions makes the outcome
explicit and testable, and avoids hidden policy choices in the frontend.

**Alternatives considered**:

- Only zero-score failed scans: rejected because the contract allows exempt.
- Only exempt failed scans: rejected because the contract allows zero.
- Allow manual 0-100 scores for failed scans: rejected because failed-scan
  files remain unavailable for content review.

## Decision: Keep review, grading, student result, and reporting surfaces separate

**Rationale**: Each surface has distinct permissions and privacy boundaries.
Review and grading may see same-school response details, student results show
only own safe summaries, and reporting exposes only aggregate fields.

**Alternatives considered**:

- Reuse one response detail for all roles: rejected because it risks leaking
  private grading or file metadata.
- Put grading controls in reporting: rejected because reports are aggregate
  views.
- Put student result state in teacher review components: rejected because
  student self-view privacy differs.

## Decision: Use route-local composables and minimal Pinia coordination

**Rationale**: Advanced answers and file metadata are sensitive. Keeping most
state route-local limits persistence across unrelated screens. Pinia remains
for session, active school, shell, navigation, and permission state.

**Alternatives considered**:

- Store all response and grading data in Pinia: rejected because it persists
  sensitive data too broadly.
- Put HTTP and state logic in components: rejected by service isolation and
  component-boundary rules.
- Use only global stores for stale-response handling: rejected because request
  identity is route and view specific.

## Decision: Centralize stale-response and sensitive diagnostics rules

**Rationale**: Route, active school, selected questionnaire, selected response,
student, report filters, due date, scan state, grading state, auth, or
permission changes can make in-flight responses stale. Safe diagnostics must
avoid raw answers, file contents, paths, answer keys, private notes, and
unauthorized identifiers.

**Alternatives considered**:

- Rely on shared error handling only: rejected because advanced assessment
  introduces files, scan states, and answer privacy.
- Let late responses update visible state: rejected because stale grades or
  file availability can mislead users.
- Log raw response data for debugging: rejected because diagnostics can become
  a disclosure channel.

## Decision: Keep Vue component boundaries explicit

**Rationale**: Vue 3 Composition API, `<script setup>`, focused SFCs,
props-down/events-up, service-isolated HTTP access, and feature composables
are the approved frontend baseline. Route views compose focused components and
do not own service calls, file validation, grading dialogs, or report mapping
directly.

**Alternatives considered**:

- One large advanced assessment route component: rejected because it mixes
  authoring, submission, review, grading, results, and reporting concerns.
- Direct Axios in components: rejected by frontend architecture rules.
- Add TypeScript-only contracts: rejected because the current frontend
  baseline uses JavaScript with mapping helpers.

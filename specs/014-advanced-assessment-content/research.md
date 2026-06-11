# Research: Advanced Assessment and Content Types

## Decision: Expand OpenAPI before backend exposure

**Decision**: All advanced question types, answer schemas, student response submission, file-response upload handling, response review, grading, scan state, report catalog fields, and response/error semantics must be added to `api/openapi.yaml` and affected platform contracts before backend routes expose the behavior.

**Rationale**: The constitution requires API-first contract governance. Advanced assessment changes request payloads, response shapes, file handling, grading behavior, and reporting data, so undocumented backend behavior would create frontend/backend drift and privacy risk.

**Alternatives considered**:

- Backend-local routes first: rejected because it would create undocumented public API behavior.
- Feature-local contract notes only: rejected because backend and frontend consume the aggregate OpenAPI contract.
- Reusing existing questionnaire schemas with free-form fields: rejected because advanced answer shapes require explicit validation, visibility, and grading rules.

## Decision: Limit advanced question types to `long_text` and `file_response`

**Decision**: This slice approves only `long_text` and `file_response` beyond existing `multiple_choice`, `true_false`, and `short_text`.

**Rationale**: The roadmap explicitly calls out long-text and file-response answers. Limiting v1 scope keeps schemas, grading, reporting, and file-security behavior reviewable while avoiding a broad assessment engine.

**Alternatives considered**:

- Add numeric/date/rating questions: rejected for this slice because they add structured auto-grading and reporting behavior not requested by the roadmap.
- Add matching/ordering/multi-select questions: rejected because they materially expand assessment authoring and scoring complexity.
- Add media/audio/video responses: rejected because storage, scanning, preview, accessibility, and retention rules need a separate specification.

## Decision: Long-text answers allow 1-10,000 characters

**Decision**: Long-text answers accept 1 to 10,000 characters and reject blank or whitespace-only answers.

**Rationale**: This supports short written responses and longer essay-style work while bounding payload size, validation, storage, audit, and report-summary behavior.

**Alternatives considered**:

- 20-5,000 characters: rejected because some valid short answers would be rejected and longer essays may be too constrained.
- Teacher-configurable 1-20,000 characters: rejected for v1 because per-question length limits expand authoring, validation, and testing scope.
- No explicit maximum: rejected because it creates storage and denial-of-service risk.

## Decision: File-response answers reuse teacher-content file categories and limit

**Decision**: File responses allow PDF, image, text, and office files, with a maximum size of 25 MB and one file per question.

**Rationale**: Existing teacher-content behavior already uses these launch-scope file categories and limit. Reusing them reduces security risk and keeps validation, scanning, storage, and test patterns aligned.

**Alternatives considered**:

- PDF/image/text only at 10 MB: rejected because it is more restrictive than existing school content patterns without a product need.
- Audio/video/archive support: rejected because media handling and archive safety materially expand scanning, preview, storage, and retention concerns.
- Multiple files per question: rejected because v1 can satisfy the file-response need with one private scanned attachment per question.

## Decision: File-response attachments are scan-gated private files

**Decision**: Student answer files are stored in private tenant-scoped storage with scan status initialized to `pending`; they remain unavailable to teacher review, grading, student display, reports, platform summaries, support diagnostics, and downloads until scan status is `clean`.

**Rationale**: Existing teacher-content safety rules already require malware-scan gating. Student-uploaded answer files carry similar or higher risk and must not leak through alternate views while pending or failed.

**Alternatives considered**:

- Let teachers view pending files with warnings: rejected because unsafe files would become available before scan completion.
- Allow report access to file metadata before clean scan: rejected because metadata can reveal protected file existence.
- Public temporary URLs: rejected because this slice requires private tenant-safe file delivery only.

## Decision: Failed scans block the answer and allow zero or exemption

**Decision**: A failed malware scan keeps the file unavailable and blocks content-based review. Authorized teachers or school administrators may grade the affected file-response answer as zero or exempt it; replacement uploads are outside this slice.

**Rationale**: This preserves file safety while allowing academic closure for blocked submissions without adding resubmission/versioning behavior.

**Alternatives considered**:

- Allow one replacement upload before due date: rejected because it reintroduces resubmission and attempt versioning complexity.
- Auto-exempt failed-scan answers: rejected because grading responsibility should remain with authorized school actors.
- Allow teacher download after failed scan: rejected because unsafe files must remain unavailable.

## Decision: One submission attempt per assigned questionnaire

**Decision**: A student may submit one attempt per assigned questionnaire. Duplicate submissions and resubmissions after submit are rejected unless a future specification approves attempt reopening or versioning.

**Rationale**: One attempt keeps v1 state, grading, audit, reporting, and conflict handling simple and testable.

**Alternatives considered**:

- Resubmit until due date: rejected because it requires attempt versioning and grading invalidation rules.
- Teacher-configurable attempt limits: rejected because it adds authoring and reporting complexity outside this slice.
- Unlimited drafts with final submit: rejected because draft persistence and recovery need separate UX and contract rules.

## Decision: Submission closes at the learning-set due date

**Decision**: Advanced questionnaire responses use the assigned learning set due date as the submission deadline. This slice does not add a separate assessment open/close window.

**Rationale**: Learning sets are already the student assignment context, so using their due date keeps submission eligibility aligned with existing student workflow boundaries and avoids a second scheduling model.

**Alternatives considered**:

- Separate questionnaire-entry open/close window: rejected because it adds scheduling and conflict rules beyond the current slice.
- Per-student extension window: rejected because extensions require separate approval, audit, and reporting rules.
- No deadline enforcement: rejected because the spec already requires closed-window rejection and teachers need predictable submission boundaries.

## Decision: Advanced answers use manual 0-100 grading

**Decision**: `long_text` and `file_response` answers are manually graded on a 0-100 point scale. Existing auto-gradable question behavior remains unchanged.

**Rationale**: Manual grading fits open-ended responses and aligns with existing grade value conventions without introducing rubric engines or custom grading scales.

**Alternatives considered**:

- Status-only grading: rejected because it does not support numeric score summaries or consistent academic reporting.
- Per-question rubric scales: rejected because v1 does not need rubric authoring, normalization, or multi-criterion grading.
- AI or automatic grading: rejected because it introduces unsupported model, review, explainability, and compliance concerns.

## Decision: Students see teacher feedback summaries, not private grading notes

**Decision**: Student-visible assessment responses expose score/status plus teacher feedback summary. Private grading notes remain internal and hidden from student, guardian, report, platform, support, and audit payloads.

**Rationale**: Feedback summary gives students actionable grading context while preserving an internal note boundary for review and audit-sensitive information.

**Alternatives considered**:

- Score/status only: rejected because it removes normal educational feedback value from graded long-form work.
- Teacher chooses visibility per grading outcome: rejected for v1 because it adds per-outcome visibility branching and harder test coverage.
- Expose all grading notes: rejected because private notes may contain internal or sensitive information.

## Decision: Audit successful and denied answer-file downloads

**Decision**: Both successful and denied answer-file download attempts are audited with tenant-safe metadata.

**Rationale**: Answer files are student-submitted private content. Auditing both outcomes matches existing teacher-content download audit behavior and gives schools traceability for sensitive file access without storing file contents or private paths.

**Alternatives considered**:

- Audit successful downloads only: rejected because denied attempts can indicate probing or authorization issues.
- Audit denied downloads only: rejected because successful access also needs accountability.
- Do not audit downloads separately: rejected because file access has higher sensitivity than ordinary metadata reads.

## Decision: Clean answer files are downloadable by assigned teachers and school administrators

**Decision**: Owning or assigned teachers and school administrators with same-school assessment review authority may download clean answer files for review.

**Rationale**: This matches the grading authority model already approved for advanced responses and avoids creating a file-review role that differs from grading responsibility.

**Alternatives considered**:

- Teacher-only download: rejected because school administrators may need same-school assessment review and grading authority.
- School-administrator-only download: rejected because teachers need to review submitted files for grading.
- Broader staff download: rejected because file responses are private student submissions and require narrow access.

## Decision: Reports expose summary fields only

**Decision**: Report catalog entries and generated reports may expose only assessment counts, completion status, grading status, and score summaries for advanced assessments.

**Rationale**: Reports should support operational oversight without leaking private answer text, answer files, file links, answer keys, private grading notes, storage metadata, or cross-tenant details.

**Alternatives considered**:

- Include long-text answers in reports: rejected because raw student responses increase privacy risk and retention scope.
- Include clean file links for administrators: rejected because report outputs are not approved as secure file packaging or attachment delivery.
- Include private grading notes: rejected because those notes are not student/report-safe data.

## Decision: Backend implementation structure

**Decision**: Implement through Laravel controllers, Form Requests, Policies, API Resources, Services, DTOs, and repositories/query objects only for complex response aggregation, report catalog summary exposure, answer-file lookup, or audit-safe reads.

**Rationale**: This follows the constitution and prior backend slices while keeping controllers thin, authorization explicit, tenant checks central, file access reviewable, and report exposure constrained.

**Alternatives considered**:

- Controller-level assessment logic: rejected by constitution and because submission/grading/file access rules need dedicated services and policies.
- Repository abstraction for every model: rejected because repositories are reserved for genuinely complex data access patterns.
- Store answer schemas as unvalidated free-form payloads: rejected because contract, validation, and reporting behavior require explicit schemas.

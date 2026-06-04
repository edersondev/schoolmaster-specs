# Feature Specification: Backend Teacher Workflow Lifecycle and Corrections

**Feature Branch**: `010-teacher-workflow-lifecycle`  
**Created**: 2026-06-01  
**Status**: Ready for Implementation  
**Input**: User description: "Run speckit-specify for backend roadmap item 5: Teacher Workflow Lifecycle and Corrections. Add detail, update, deactivate/activate, delete/restore, download, bulk import, and correction workflows for teacher content, questionnaires, learning sets, grades, and attendance. Define immutable versus editable fields, closed-period correction authority, audit history, soft-delete rules, student visibility impact, tenant isolation, school authorization, contract-first API-only behavior, and explicit exclusions for guardian self-service, report lifecycle expansion, platform-wide support access, frontend implementation, billing, messaging, and undocumented APIs."

## Clarifications

### Session 2026-06-01

- Q: Who may manage existing teacher workflow records within the same school? -> A: Creating/owning teachers may manage their own records; school administrators may manage same-school records.
- Q: How should content or questionnaire edits that change historical student-facing meaning behave after use? -> A: Reject edits that change historical student-facing meaning once content or questionnaires are used.
- Q: Who may run grade and attendance imports in v1? -> A: Only school administrators may run grade and attendance imports.
- Q: Which lifecycle states apply to teacher workflow resources in v1? -> A: Teacher content, questionnaires, learning sets, grades, and attendance use `active`, `inactive`, and `deleted`; restore returns records to `inactive`.
- Q: Which teacher content download attempts should be audited? -> A: Audit both successful and denied teacher content download attempts.
- Q: Which import submission format is approved for v1 grade and attendance imports? -> A: JSON payload only.
- Q: May v1 grade and attendance imports correct existing records? -> A: No; imports create new records only.
- Q: What correction reason format is required in v1? -> A: Required free-text reason, 10-500 characters.
- Q: How do lifecycle states affect current student self-view? -> A: Current student views show active records only; inactive and deleted records are hidden except documented history.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Maintain Teacher Materials (Priority: P1)

An authorized teacher or school administrator can view details, update allowed metadata, manage lifecycle state, recover soft-deleted records, and download clean instructional materials for teacher content and questionnaires within an active school context. Teachers may manage records they created or own; school administrators may manage same-school records.

**Why this priority**: Teacher content and questionnaires are prerequisites for learning sets. Operational maintenance must preserve private file handling, scan gating, tenant boundaries, and history before downstream teaching workflows depend on edited materials.

**Independent Test**: Authenticate with an active permitted school context, create clean content and a questionnaire, retrieve details, update editable fields, deactivate active records, reactivate inactive records, soft-delete eligible records, restore deleted records to `inactive`, download only clean authorized content, and verify cross-tenant or unclean content remains inaccessible.

**Acceptance Scenarios**:

1. **Given** a teacher owns clean same-school content, **When** they retrieve the content detail or download the file through a documented teacher operation, **Then** only authorized metadata and the clean file are returned for that school.
2. **Given** uploaded content has scan status `pending` or `failed`, **When** a teacher or administrator requests a download or attaches it to a workflow, **Then** the request is rejected without exposing the file.
3. **Given** a creating or owning teacher or school administrator updates content or questionnaire fields that are documented as editable, **When** the request passes tenant, permission, lifecycle, and validation checks, **Then** the record is updated and prior audit history remains available.
4. **Given** a content item or questionnaire is already used by a published or assigned learning set, **When** an actor attempts to mutate a field that would change historical student-facing meaning, **Then** the request is rejected rather than versioned or rewritten in v1.
5. **Given** a soft-deleted content item or questionnaire is restored, **When** the original tenant, owner, dependency, scan, and lifecycle rules still allow restoration, **Then** the record returns to `inactive` in the same school context and requires a separate activation before student-facing use.

---

### User Story 2 - Maintain Roster-Aware Learning Sets (Priority: P2)

An authorized teacher can retrieve, update, deactivate, reactivate, soft-delete, and restore learning sets they created or own when scoped to an academic period and assigned through approved roster foundations rather than new direct student assignments. School administrators may manage same-school learning sets. Restored learning sets return to `inactive` and require a separate activation before student-facing use.

**Why this priority**: The completed classroom roster foundation makes roster-aware teacher workflows possible. Learning sets must now move forward without expanding legacy direct student assignment writes.

**Independent Test**: Create a class section with active roster memberships and an active teacher assignment, create or maintain a learning set for that roster, update editable learning-set fields, deactivate an active learning set, restore a deleted learning set to `inactive`, and confirm existing direct student assignments remain readable only as legacy records.

**Acceptance Scenarios**:

1. **Given** a teacher has an active teacher assignment to a ClassSection/Roster, **When** they create or update a learning set assignment in this slice, **Then** the assignment uses active roster membership and does not create new direct selected-student assignments.
2. **Given** a legacy learning set uses direct selected-student assignments from the teacher workflow foundation, **When** it is read through an approved operation, **Then** the legacy assignment remains visible without allowing new direct-assignment writes.
3. **Given** a learning set is published or visible to students, **When** a teacher attempts to change ordered entries, assignment scope, academic period, or student-visible due information, **Then** the backend applies the documented editability rule and preserves audit history.
4. **Given** a learning set references inactive, unclean, deleted, cross-tenant, or unauthorized content, questionnaires, rosters, memberships, teacher assignments, or academic periods, **When** update, restore, or reactivation is attempted, **Then** the request is rejected without partial state changes.

---

### User Story 3 - Correct Grades and Attendance (Priority: P3)

An authorized teacher or school administrator can retrieve grade and attendance details, update allowed fields, apply corrections with required reasons, and preserve original values, student visibility, and audit history. Teachers may manage grade and attendance records they originally recorded or own while the academic period remains open; school administrators may manage same-school records according to open-period and closed-period rules.

**Why this priority**: Grades and attendance are student-visible academic records. Corrections need explicit authority, traceability, and closed-period rules before operational maintenance is exposed.

**Independent Test**: Record a grade and attendance entry, retrieve details, correct each record with a 10-500 character free-text reason, verify the current value changes according to visibility rules while original values remain in history, and verify unauthorized, cross-tenant, closed-period, blank-reason, too-short-reason, too-long-reason, or missing-reason attempts are rejected.

**Acceptance Scenarios**:

1. **Given** a teacher recorded a same-school grade or attendance entry in an open academic period, **When** they submit a documented correction with a 10-500 character free-text reason, **Then** the current student-visible value is updated and the prior value remains in correction history.
2. **Given** a school administrator corrects a same-school academic record, **When** the correction passes authority and validation checks, **Then** the correction records the administrator as actor and keeps the original recorder immutable.
3. **Given** an academic period is closed, **When** a school administrator requests a grade or attendance correction with a 10-500 character free-text reason, **Then** the correction may be accepted if all tenant, validation, student visibility, and audit rules pass; teacher-submitted closed-period corrections are rejected in v1.
4. **Given** a correction affects a student-visible record, **When** the correction is accepted, **Then** current student self-view shows the documented current value only while the record is `active`, hides `inactive` and `deleted` records except where OpenAPI explicitly documents historical labels, and does not expose private correction notes, unauthorized actor metadata, or cross-tenant details.

---

### User Story 4 - Import Teacher Workflow Records (Priority: P4)

An authorized school administrator can submit bulk imports for approved teacher workflow records, receive all-or-nothing validation results, and avoid partial state changes when any imported row is invalid.

**Why this priority**: Operational schools need efficient maintenance after lifecycle and correction rules exist, but import behavior must not bypass validation, tenant isolation, roster eligibility, audit, or student visibility controls.

**Independent Test**: Submit a valid same-school JSON import payload for the approved import scope, verify records are created atomically, then submit an import with one invalid, duplicate existing, cross-tenant, unauthorized, or closed-period row and verify no row changes state.

**Acceptance Scenarios**:

1. **Given** an import contains only valid same-school records in the approved import scope, **When** an authorized school administrator submits it, **Then** the backend validates every row before committing any state and records audit history for each accepted row.
2. **Given** one row references a cross-tenant student, inactive roster membership, unsupported status, missing required field, invalid academic period, or unauthorized target, **When** the import is submitted, **Then** the entire import is rejected without partial creation.
3. **Given** the feature scope includes bulk import, **When** OpenAPI defines the operation, **Then** imports are limited to create-only JSON payloads for grades and attendance, capped at 500 rows per import, and rejected as a whole if any row is invalid or attempts to correct an existing record.

### Edge Cases

- How does the backend reject missing, inactive, mismatched, or unauthorized `X-School-Id` tenant context before teacher workflow lifecycle or correction data access?
- What happens when a teacher tries to manage content, questionnaires, learning sets, grades, or attendance belonging to another teacher in the same school?
- How does roster-aware assignment avoid creating new legacy direct selected-student assignments while preserving read compatibility for existing records?
- What happens when a teacher assignment, roster, roster membership, student enrollment, teacher-compatible role, or academic period becomes inactive after a record was created?
- Which fields are immutable after creation, after publication, after student visibility, or after academic-period closure?
- What happens when a correction omits a reason, submits a blank reason, submits a reason shorter than 10 characters or longer than 500 characters, or attempts to hide audit history?
- How does the backend keep private correction notes, full import payloads, file metadata, and unauthorized cross-tenant identifiers out of student-visible responses?
- What happens when a soft-deleted record is still referenced by active learning sets, student-visible records, reports, or correction history?
- How does restore behavior handle dependency conflicts, duplicate names or codes, inactive owners, unclean content, and closed academic periods?
- How does download behavior reject unclean, deleted, inactive, cross-tenant, or unauthorized content without revealing private storage paths?
- How does bulk import report validation errors without exposing another tenant's student, teacher, content, roster, or academic data?
- What happens when concurrent update, correction, delete, restore, or import requests target the same record?
- How does this slice avoid introducing guardian self-service, report lifecycle expansion, platform-wide support access, frontend behavior, billing, messaging, or undocumented APIs?

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Implement backend-only teacher workflow lifecycle behavior for teacher content, questionnaires, and learning sets, plus grade and attendance correction behavior, downloads, import operations, editability rules, correction history, soft-delete/restore, authorization, tenant context, audit events, response resources, and regression tests.
- **Frontend repository impact**: No frontend implementation is included. Future teacher, student, or administrator screens must consume only the published contract after backend behavior is specified and implemented.
- **Specification or contract repository impact**: Expand `api/openapi.yaml` and the platform feature contract before backend routes expose detail, update, lifecycle, delete/restore, download, import, or correction operations for teacher workflow resources.
- **Delivery ownership and sequencing**: `schoolmaster-specs` leads with specification and OpenAPI, `schoolmaster-backend` implements only approved contract behavior next, and `schoolmaster-frontend` remains deferred until a later frontend slice.

### API Contract Impact

- **OpenAPI update required**: Yes. The existing teacher workflow foundation published list/create operations; this feature must define all new detail, update, lifecycle, delete/restore, download, import, and correction operations before backend exposure.
- **Versioned endpoints affected**: Existing `/api/v1/teacher-content`, `/api/v1/questionnaires`, `/api/v1/learning-sets`, `/api/v1/grades`, and `/api/v1/attendance` resources, plus only explicitly documented subresources for downloads, lifecycle transitions, restoration, imports, and corrections.
- **JSON response impact**: Responses must use documented success, paginated, validation, unauthorized, forbidden, tenant-mismatch, conflict, not-found, import-validation, correction-history, and download response semantics. No backend-local envelope or undocumented status code is approved.
- **Authentication/authorization impact**: All operations require authenticated access and an active permitted school context. Teacher, school-administrator, and student self-view visibility remain separate. Platform administration does not imply school-scoped teacher workflow authority.
- **Compatibility impact**: Additive API expansion. Existing list/create operations and legacy direct learning-set assignments remain compatible; any breaking response or assignment-shape change requires explicit contract versioning or migration notes.

### Data & Tenancy Impact

- **Tenant scoping impact**: Teacher content, questionnaires, learning sets, learning-set entries, roster-aware learning-set assignments, grades, attendance, imports, corrections, lifecycle history, soft-delete state, and audit events are school-owned records governed by `school_id`.
- **Cross-tenant or platform access impact**: Cross-tenant references must be rejected before data exposure or persistence. Platform users receive no implicit support access, correction authority, import authority, download authority, or student-visible bypass.
- **Soft delete impact**: Public lifecycle must be recoverable where specified. Teacher content, questionnaires, learning sets, grades, and attendance use `active`, `inactive`, and `deleted`; restore returns records to `inactive`. Soft deletion must preserve audit, correction history, student-visible historical references, and report compatibility without permanent purge in this slice.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The backend MUST NOT expose teacher workflow detail, update, lifecycle, delete/restore, download, import, or correction behavior until OpenAPI defines operation IDs, request schemas, response schemas, errors, authorization rules, and tenant behavior for each approved operation.
- **FR-002**: Every operation in this slice MUST resolve an active permitted school context before authorization, validation, file access, roster lookup, student lookup, teacher lookup, academic-period lookup, persistence, lifecycle transition, correction, import processing, audit recording, or response shaping.
- **FR-003**: Requests with missing, mismatched, inactive, or unauthorized school context MUST fail before teacher content, questionnaire, learning set, grade, attendance, roster, teacher assignment, student profile, academic period, import, correction, audit, or private file data is accessed.
- **FR-004**: Detail operations MUST return only records visible to the authenticated actor in the resolved school context and MUST avoid exposing private file paths, full import payloads, private correction notes, credentials, malware-scan internals, or unauthorized cross-tenant identifiers.
- **FR-004a**: Teacher management operations MUST be limited to records the teacher created or owns inside the resolved school context. School administrators may manage same-school teacher workflow records. Same-school teachers who are not the creator or owner MUST NOT manage another teacher's content, questionnaires, learning sets, grades, or attendance unless a future specification grants explicit delegated authority.
- **FR-005**: Teacher content lifecycle operations MUST support only documented editable fields, lifecycle states, soft-delete/restore behavior, and teacher download behavior. Downloads MUST require clean scan status, active allowed lifecycle state, same-school ownership or permission, and private tenant-safe file delivery.
- **FR-005a**: Teacher content download audit behavior MUST record both successful and denied download attempts with actor, outcome, target content identifier when resolved, school identifier when resolved, denial reason category where safe, and tenant-safe summary metadata. Audit records MUST NOT store private file contents, private storage paths, credentials, full request payloads, or unauthorized cross-tenant details.
- **FR-006**: Questionnaire lifecycle operations MUST preserve submitted question sequence, supported v1 question types, historical meaning for questionnaires already attached to published or assigned learning sets, and documented activation, deactivation, soft-delete, and restore rules. V1 MUST reject edits that change historical student-facing meaning for used content or questionnaires rather than automatically creating versions.
- **FR-007**: Learning-set lifecycle operations MUST use roster-aware assignment rules for new writes after the classroom roster foundation. Existing direct selected-student assignments MUST remain readable as legacy records, but this slice MUST NOT introduce new direct-assignment write behavior.
- **FR-007a**: Teacher content, questionnaires, learning sets, grades, and attendance MUST use the v1 lifecycle states `active`, `inactive`, and `deleted`. Delete operations MUST move records to `deleted`; restore operations MUST move records to `inactive`; activation from `inactive` MUST be a separate documented lifecycle transition and MUST pass current tenant, dependency, scan, roster, academic-period, and student visibility rules before the record can become active or student-facing.
- **FR-008**: The specification and OpenAPI MUST identify immutable versus editable fields for teacher content, questionnaires, learning sets, grades, and attendance using the Field Editability Matrix below. At minimum, tenant ownership, original creator or recorder, original creation timestamp, original student or roster target, original academic period, original file scan history, original correction actor history, and audit history MUST remain immutable after creation unless a future migration specification explicitly approves a controlled change.
- **FR-009**: Grade and attendance corrections MUST preserve original values, current values, correction reason, actor, timestamp, affected student profile, academic period, and tenant-safe audit history. Corrections MUST require a free-text reason from 10 to 500 characters inclusive and MUST reject missing, blank, shorter, or longer reasons. Corrections MUST NOT overwrite original recorder identity or erase prior corrections.
- **FR-010**: Closed-period grade and attendance corrections MUST be limited to authorized school administrators in v1. Teachers MUST NOT directly apply or submit closed-period corrections in this slice. School-administrator closed-period corrections MUST require a free-text reason from 10 to 500 characters inclusive and MUST preserve original values, current values, actor, timestamp, affected student profile, academic period, student visibility impact, and tenant-safe audit history.
- **FR-011**: Soft deletion MUST be reversible only through documented restore operations and MUST reject deletes or restores that would break active learning sets, student-visible records, correction history, audit history, roster eligibility, or tenant isolation. Restores MUST return records to `inactive`, not directly to `active`. Permanent purge is outside this slice.
- **FR-012**: Bulk import operations MUST be limited to create-only JSON payload imports for new grades and attendance records in v1, MUST accept no more than 500 rows per import, MUST be all-or-nothing within a submitted import, and MUST reject the entire import when any row is invalid, cross-tenant, unauthorized, malformed, duplicate, references an existing record for correction/update, closed-period-restricted, or dependency-conflicting. CSV, spreadsheet, archive, file-upload imports, and import-based corrections are outside v1.
- **FR-012a**: Grade and attendance import authority MUST be limited to authorized school administrators in v1. Teachers MUST NOT run imports, including imports limited to students in their active roster assignments.
- **FR-013**: Student visibility impact MUST be explicit for every update, lifecycle transition, delete, restore, download, import, or correction that affects student-facing learning sets, content availability, grades, or attendance. Current student self-view MUST show `active` teacher workflow records only. `Inactive` and `deleted` records MUST be hidden from current student views except where OpenAPI explicitly documents historical labels. Student self-view MUST expose only current approved values and permitted historical labels, never private correction notes or unauthorized actor metadata.
- **FR-014**: Backend audit events MUST record detail-sensitive lifecycle changes, updates, successful and denied teacher content downloads, imports, import rejection outcomes, corrections, correction rejections, delete/restore, and blocked cross-tenant attempts with actor user ID when available, action, outcome, lifecycle or correction reason when present, canonical school and target identifiers when resolved, and tenant-safe summary metadata only.
- **FR-015**: Backend authorization MUST keep platform administration, school administration, teacher workflow access, student self-view, guardian access, reporting, and support access permissions separate. Teacher workflow permissions MUST NOT grant access to manage another teacher's records, roster administration, school administration, guardian access, report lifecycle management, or platform support access.
- **FR-016**: Backend validation MUST reject undocumented request fields, unsupported lifecycle states, unsupported filters, unsupported include expansion, unsupported sort values, invalid edit attempts against immutable fields, inactive references, unclean content downloads, unauthorized owner changes, invalid correction values, missing, blank, too-short, or too-long correction reasons, unsupported import rows, cross-tenant references, closed-period violations, and dependency conflicts.
- **FR-017**: Backend responses MUST match the published OpenAPI success and error semantics declared for each approved operation. The backend MUST NOT emit undocumented status semantics, fields, filters, lifecycle actions, correction states, import result shapes, or download behaviors unless OpenAPI first documents them.
- **FR-018**: Backend implementation MUST NOT expose guardian self-service, report lifecycle expansion, platform-wide support access, frontend behavior, billing, messaging, notification-center behavior, advanced assessment types, permanent purge, legal hold, data anonymization, or undocumented APIs in this slice.
- **FR-019**: Backend tests MUST cover successful flows, validation failures, authorization failures, tenant isolation failures, inactive school/user/student/teacher/period handling, roster assignment dependency failures, immutable-field rejection, correction history, closed-period correction authority, soft-delete/restore conflicts, import all-or-nothing behavior, download authorization and scan gating, student visibility impact, audit event coverage, concurrent conflict handling, and response shape for every operation in this slice.

### Field Editability Matrix

OpenAPI MUST encode these editable and immutable field rules before backend implementation exposes matching routes. Lifecycle status changes use documented status, delete, and restore operations rather than generic update payloads.

| Resource | Editable in v1 | Immutable after creation | Additional lock rules |
|----------|----------------|--------------------------|-----------------------|
| TeacherContentItem | Title, description, optional folder reference where already supported, lifecycle status through documented operations | `school_id`, original creator/owner, creation timestamp, private storage reference, file size, detected media metadata, scan history, audit history | File replacement and any metadata change that changes historical student-facing meaning are rejected after the content is used by a published or assigned learning set |
| Questionnaire | Title, description, supported v1 question prompts/options/answers/sequence while unused, lifecycle status through documented operations | `school_id`, original creator/owner, creation timestamp, audit history | Question type, prompt, options, answer metadata, or sequence changes that alter historical student-facing meaning are rejected after the questionnaire is used by a published or assigned learning set |
| LearningSet | Title, description, ordered entries, roster-aware assignment scope, student-visible due information, and lifecycle status while current visibility and dependency rules allow | `school_id`, original creator/owner, creation timestamp, original academic period, legacy direct selected-student assignment origin, audit history | Once published or student-visible, ordered entries, assignment scope, academic period, and student-visible due information are immutable unless a future specification approves a learning-set correction or versioning workflow |
| GradeRecord | Current grade value only through the documented correction endpoint; lifecycle status through documented operations | `school_id`, original recorder, original creation timestamp, student profile, original academic period, original grade value, correction actor history, audit history | Closed-period corrections are school-administrator-only; imports cannot update or correct existing grades |
| AttendanceRecord | Current attendance status only through the documented correction endpoint; lifecycle status through documented operations | `school_id`, original recorder, original creation timestamp, student profile, original academic period, attendance date, original attendance status, correction actor history, audit history | Closed-period corrections are school-administrator-only; imports cannot update or correct existing attendance records |

### Key Entities *(include if feature involves data)*

- **TeacherContentItem**: Existing school-owned instructional content metadata and private file reference with `active`, `inactive`, or `deleted` lifecycle state, scan status, ownership, download eligibility, restore-to-inactive behavior, and audit history.
- **Questionnaire**: Existing school-owned teacher-authored assessment or activity with ordered v1-supported questions, `active`, `inactive`, or `deleted` lifecycle state, usage dependencies, restore-to-inactive behavior, and rejection rules for edits that would change historical student-facing meaning after use.
- **LearningSet**: Existing school-owned instructional sequence for an academic period with `active`, `inactive`, or `deleted` lifecycle state, restore-to-inactive behavior, and roster-aware assignment rules for new writes while retaining legacy direct assignment read compatibility.
- **LearningSetAssignment**: Relationship between a learning set and its student audience; new writes use approved roster membership context, while existing direct selected-student assignments remain readable as legacy records.
- **GradeRecord**: Existing school-owned academic record for one student profile and academic period with immutable original recorder context, current value, correction history, `active`, `inactive`, or `deleted` lifecycle state, restore-to-inactive behavior, and student visibility rules where current student self-view shows active records only.
- **AttendanceRecord**: Existing school-owned attendance record for one student profile, academic period, date, and attendance status with immutable original recorder context, current value, correction history, `active`, `inactive`, or `deleted` lifecycle state, restore-to-inactive behavior, and student visibility rules where current student self-view shows active records only.
- **CorrectionRecord**: Tenant-safe history entry for an accepted grade or attendance correction, including original value reference, new value, actor, 10-500 character free-text reason, timestamp, and student visibility impact.
- **ImportRun**: Tenant-safe record of a submitted create-only JSON bulk import, validation outcome, actor, target resource type, accepted row count, rejected row count, all-or-nothing result, and tenant-safe error summary.
- **TeacherAssignment**: Existing roster foundation relationship that determines teacher access to roster-aware learning-set, grade, and attendance maintenance where the actor is not a school administrator.
- **ClassSection/Roster**: Existing school-owned teaching structure whose active memberships define the approved student audience for new roster-aware teacher workflow writes.
- **AuditEvent**: Tenant-safe record of lifecycle, correction, download, import, authorization, validation, and conflict outcomes without full private payloads or unauthorized cross-tenant details.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Backend reviewers can map 100% of implemented routes in this slice to approved OpenAPI operation IDs without finding undocumented product endpoints.
- **SC-002**: Tenant-isolation acceptance tests reject 100% of missing, mismatched, inactive, and unauthorized school-context attempts for teacher workflow detail, update, lifecycle, delete/restore, download, import, and correction operations.
- **SC-002a**: Authorization tests reject 100% of same-school attempts by teachers to manage teacher workflow records they did not create or own, while allowing school administrators to manage same-school records covered by this specification.
- **SC-003**: Editability tests reject 100% of attempts to change documented immutable fields for teacher content, questionnaires, learning sets, grades, and attendance.
- **SC-003a**: Historical-meaning tests reject 100% of edits that would change student-facing content or questionnaire meaning after the record is used by a published or assigned learning set.
- **SC-003b**: Lifecycle tests confirm 100% of teacher content, questionnaire, learning-set, grade, and attendance restores return records to `inactive` and require a separate valid activation before active or student-facing use.
- **SC-004**: Download tests reject 100% of pending-scan, failed-scan, inactive, deleted, cross-tenant, unauthorized, or unsupported teacher content download attempts covered by this specification.
- **SC-004a**: Download audit tests confirm 100% of successful and denied teacher content download attempts create tenant-safe audit events without private file contents, storage paths, credentials, full payloads, or unauthorized cross-tenant details.
- **SC-005**: Roster-aware learning-set tests confirm 100% of new assignment writes use approved roster membership context and create no new direct selected-student assignments.
- **SC-006**: Correction tests confirm 100% of accepted grade and attendance corrections preserve original value, current value, actor, 10-500 character free-text reason, timestamp, and audit history without erasing prior corrections, and reject 100% of missing, blank, too-short, or too-long correction reasons.
- **SC-007**: Closed-period tests enforce the approved correction authority model for 100% of closed academic-period correction attempts.
- **SC-008**: Import tests confirm valid create-only JSON payload imports commit only after all rows pass validation and invalid or correction-attempt imports reject 100% of rows without partial state changes.
- **SC-008a**: Import authorization tests reject 100% of teacher, student, guardian, platform-without-school-context, and non-administrator attempts to run grade or attendance imports.
- **SC-009**: Student visibility tests confirm corrected grades, attendance, learning-set availability, and content availability show only documented current values for `active` records in current student views, hide `inactive` and `deleted` records except where OpenAPI explicitly documents historical labels, and hide private correction notes and unauthorized actor details.
- **SC-010**: Audit verification confirms 100% of update, lifecycle, delete/restore, download where required, import, correction, conflict, and blocked cross-tenant outcomes are recorded with tenant-safe actor, action, outcome, reason where applicable, target, and summary metadata.
- **SC-011**: Compatibility review confirms existing list/create teacher workflow operations, student self-view, reporting, and legacy direct learning-set assignment reads remain compatible unless an approved contract change explicitly governs a difference.

## Assumptions

- `004-backend-teacher-workflows` established list/create behavior for teacher content, questionnaires, learning sets, grades, and attendance.
- `009-classroom-roster-foundation` established ClassSection/Roster, roster membership, teacher assignment, academic-period scoping, and read-only legacy direct-assignment compatibility.
- Teacher-owned records are managed by their creating or owning teacher unless a school administrator acts under same-school authority.
- Used content and questionnaires cannot receive edits that change historical student-facing meaning in v1; future versioning requires a separate specification and contract update.
- Teacher content, questionnaires, learning sets, grades, and attendance share v1 lifecycle states `active`, `inactive`, and `deleted`; restore returns records to `inactive`.
- New learning-set assignment writes in this slice should use roster-aware assignment rules rather than the direct selected-student assignment model from the initial teacher workflow foundation.
- Teacher content remains private tenant-scoped content and is downloadable only through authorized documented operations after clean scan status.
- Successful and denied teacher content download attempts are audited with tenant-safe metadata only.
- Correction history must be retained for academic accountability and audit review, but student-visible responses should show only approved current values and permitted historical labels.
- Current student self-view shows active teacher workflow records only; inactive and deleted records are hidden except where OpenAPI explicitly documents historical labels.
- V1 correction reasons are required free text from 10 to 500 characters inclusive.
- Soft delete is recoverable and tenant-safe; permanent purge, anonymization, and legal hold are outside this slice.
- Bulk imports are administrative operations that must not bypass validation, authorization, roster eligibility, or correction rules.
- V1 bulk imports are limited to grades and attendance with a maximum of 500 rows per import.
- V1 grade and attendance imports accept JSON payloads only; CSV, spreadsheet, archive, and file-upload imports are outside this slice.
- V1 grade and attendance imports create new records only; import-based correction or update of existing records is outside this slice.
- V1 grade and attendance imports are school-administrator-only.
- V1 closed-period grade and attendance corrections are school-administrator-only and require a correction reason.
- Frontend implementation, guardian self-service, report lifecycle expansion, platform-wide support access, billing, messaging, notifications, advanced assessment content types, and undocumented APIs remain outside this slice.

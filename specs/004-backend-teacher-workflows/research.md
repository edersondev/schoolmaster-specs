# Research: Backend Teacher Workflow Foundation

## Decision: Implement the P2 teacher workflow slice after school administration

**Rationale**: The platform foundation identifies teacher content, questionnaires, learning sets, grades, and attendance as the next value-producing workflow after tenant users, roles, permissions, academic periods, guardians, and student profile references exist. `003-backend-school-admin` provides the required tenant and academic foundation for teacher workflows.

**Alternatives considered**:

- Start student self-service next: rejected because student timelines depend on teacher-created learning sets, grades, attendance, and clean content.
- Start reporting next: rejected because reports depend on operational teacher workflow data.

## Decision: Use the published OpenAPI operation IDs as the hard implementation boundary

**Rationale**: The aggregate and platform contracts already publish list/create operations for teacher content, questionnaires, learning sets, grades, and attendance. Contract-first governance requires backend implementation to stop at these documented operations and avoid local endpoints not yet present in OpenAPI.

**Alternatives considered**:

- Implement common CRUD operations by convention: rejected because detail, update, deactivate, delete, restore, download, correction, folder management, student views, and reports are not all documented for this slice.
- Treat broad product wording about folders as permission for folder CRUD: rejected because the current contract only documents `folder_id` on teacher content creation and does not publish folder operations.

## Decision: Keep teacher-content folder management as a blocked contract gap

**Rationale**: The product specification mentions folder organization, but the current OpenAPI surface does not define public folder list/create/update/delete operations. This slice may validate a submitted `folder_id` where folder persistence already exists, but it must not expose public folder CRUD until the contract is expanded.

**Alternatives considered**:

- Add backend-only folder endpoints now: rejected because frontend and backend contracts would drift.
- Remove folder references entirely: rejected because `TeacherContentCreateRequest` already includes `folder_id`, and backend validation should not ignore same-school ownership when a folder reference is submitted.

## Decision: Resolve tenant context before module-specific work

**Rationale**: ADR 004 and multi-tenant guidance require `School` as the v1 tenant root and `school_id` as the concrete tenant column. Teacher workflow requests must reject missing, mismatched, inactive, or unauthorized tenant context before validation that depends on school-owned records, authorization decisions, storage paths, services, persistence, or response shaping.

**Alternatives considered**:

- Let each service infer tenant scope from submitted identifiers: rejected because it increases cross-tenant leakage risk and duplicates authorization logic.
- Allow platform administrators to bypass school scope: rejected because platform access is not an implicit bypass for school-scoped teacher actions.

## Decision: Treat uploaded content as private and unavailable until clean

**Rationale**: The platform rules require declared and detected type validation, size validation, tenant ownership checks, private tenant-scoped storage, and malware scan gating before content becomes available. Creating content with scan status `pending` preserves asynchronous scan workflows without exposing unverified files.

**Alternatives considered**:

- Make uploads available immediately and revoke later if scanning fails: rejected because it exposes potentially unsafe files.
- Store files publicly and rely on obscured URLs: rejected because tenant-scoped private storage and authorized API access are required.
- Skip detected content-type validation: rejected because declared type alone is not enough to block disguised files.

## Decision: Use direct selected-student assignments for v1 learning sets

**Rationale**: The platform clarification states that v1 learning sets are assigned directly to selected `StudentProfile` records. Class, course, section, group, and roster workflows are outside v1, so backend validation should focus on active same-school student profiles aligned to the selected academic period.

**Alternatives considered**:

- Create class or group models as part of this slice: rejected because those entities are not specified for v1.
- Assign learning sets to all school students by default: rejected because the product decision requires selected student profiles.

## Decision: Enforce active academic period for learning sets, grades, and attendance

**Rationale**: Teacher operations are period-scoped. Active periods allow normal recording, while closed periods remain read-only unless a future correction workflow is documented. This preserves academic history and avoids accidental changes to completed periods.

**Alternatives considered**:

- Allow writes to planned or closed periods: rejected because the platform rules define closed periods as read-only and teacher operations as active-period workflows.
- Let clients decide period validity: rejected because backend is the authority for tenant and academic integrity.

## Decision: Verify the slice with operation-level feature tests and contract validation

**Rationale**: The slice changes critical teacher workflows, file handling, tenant isolation, academic validation, and response envelopes. PHPUnit feature and unit tests should cover success, validation, authorization, tenant isolation, inactive statuses, upload validation, scan gating, learning-set reference integrity, grade and attendance rules, and response shape. Redocly validation ensures consumed contracts remain valid.

**Alternatives considered**:

- Rely on unit tests only: rejected because tenant, upload, response-envelope, and route behavior are observable at the API boundary.
- Defer contract validation to frontend work: rejected because backend implementation must not drift from the published contract.

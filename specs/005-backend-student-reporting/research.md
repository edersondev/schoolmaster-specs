# Research: Backend Student and Reporting Foundation

## Decision: Implement the P3 student and reporting backend slice after teacher workflows

**Rationale**: The platform foundation places student progress views and school reporting after teacher-created learning sets, content, grades, and attendance exist. `004-backend-teacher-workflows` provides the operational records that student self-view and reports consume.

**Alternatives considered**:

- Start frontend student screens next: rejected because backend behavior and report output semantics must be implemented and verified before the SPA can safely consume them.
- Expand teacher workflows next: rejected because the next approved platform priority is P3 visibility and reporting, and new teacher write behavior would require additional contract expansion.

## Decision: Use the published OpenAPI operation IDs as the hard implementation boundary

**Rationale**: The aggregate and platform contracts already publish the student timeline, student content download, student grade/attendance listing, report listing, report request, and report download operations. Contract-first governance requires backend implementation to stop at these documented operations and avoid local endpoints not present in OpenAPI.

**Alternatives considered**:

- Implement extra student dashboard or report-management endpoints by convention: rejected because frontend and backend contracts would drift.
- Add report retry, deletion, or custom report definition operations now: rejected because those workflows are not published in the v1 contract.

## Decision: Resolve tenant context before student or report data access

**Rationale**: ADR 004 and multi-tenant guidance require `School` as the v1 tenant root and `school_id` as the concrete tenant column. Student and report requests must reject missing, mismatched, inactive, or unauthorized tenant context before validation that depends on school-owned records, authorization decisions, file storage, report filters, persistence, or response shaping.

**Alternatives considered**:

- Let student endpoints infer school context only from the authenticated user: rejected because users may have multiple school relationships and the contract includes tenant context behavior.
- Allow platform administrators to bypass school scope for reports: rejected because platform-wide reporting is outside v1 and would require explicit contract and authorization rules.

## Decision: Require active same-school student ownership for student self-view

**Rationale**: Student self-view endpoints expose personal learning sets, grades, attendance, and assigned content. The authenticated user must be linked to an active `StudentProfile` in the resolved school, and records must be filtered to that profile only.

**Alternatives considered**:

- Accept a submitted student identifier for self-view endpoints: rejected because it creates unnecessary risk of other-student access and is not documented on the student endpoints.
- Let guardian or school-admin users consume student endpoints: rejected because guardian/admin views require separate actor-specific contracts.

## Decision: Gate student content download by assignment and clean scan status

**Rationale**: Students may download teacher content only when the file is same-school, private, linked through a learning set assigned to their active profile, operationally available, and malware-scan clean. This preserves the upload safety model established in the teacher workflow slice.

**Alternatives considered**:

- Permit download for any same-school clean content: rejected because assignment is the student authorization boundary.
- Allow pending-scan downloads with a warning: rejected because the platform requires scan-gated availability.

## Decision: Keep reporting asynchronous and launch-scope only

**Rationale**: The platform specification and OpenAPI define report requests as asynchronous `ReportRun` records for attendance, grades, academic structure, and school activity. Report requests return status and availability metadata rather than blocking until output files exist.

**Alternatives considered**:

- Generate reports synchronously during the request: rejected because the contract returns an accepted report run and later download availability.
- Add custom report definitions: rejected because v1 only defines launch-scope report types.

## Decision: Retain report output files for 90 days and preserve metadata after expiry

**Rationale**: The approved product clarification requires generated PDF and CSV output files to expire after 90 days while `ReportRun` metadata remains for audit and history. Output availability must derive from generation and expiry metadata, not client assumptions.

**Alternatives considered**:

- Retain generated files indefinitely: rejected because it conflicts with the approved retention rule.
- Delete the whole report run after output expiry: rejected because metadata retention is required.

## Decision: Require explicit regeneration through a new ReportRun

**Rationale**: Expired output downloads must return the documented expired-output response and must not trigger regeneration. Users regenerate by requesting a new `ReportRun` with the same filters, preserving history and making compute/storage behavior explicit.

**Alternatives considered**:

- Regenerate expired files automatically during download: rejected because the platform explicitly disallows download-time regeneration.
- Reopen the expired run and attach new files to it: rejected because a new report run preserves a clearer audit history.

## Decision: Verify the slice with operation-level feature tests and contract validation

**Rationale**: The slice changes critical student visibility, private file access, tenant isolation, report filters, asynchronous generation, retention, and response/binary behavior. PHPUnit feature and unit tests should cover success, validation, authorization, tenant isolation, inactive statuses, student ownership, content scan gating, report filter scoping, output expiry, no download-time regeneration, and response shape. Redocly validation ensures consumed contracts remain valid.

**Alternatives considered**:

- Rely on service unit tests only: rejected because tenant context, authorization, binary downloads, and response semantics are observable at the API boundary.
- Defer contract validation to frontend work: rejected because backend implementation must not drift from the published contract.

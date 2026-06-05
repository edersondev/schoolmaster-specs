# Backend Feature Roadmap

This roadmap lists backend feature areas that remain outside the completed
backend slices and should be specified before implementation. Each item needs
contract review before backend code changes expose new behavior.

## Current Backend Baseline

Completed or specified backend slices:

- `002-backend-api-foundation`: authentication, tenant context, response
  foundations, and baseline school management.
- `003-backend-school-admin`: user, role, permission, academic year, academic
  period, and guardian list/create foundation.
- `004-backend-teacher-workflows`: teacher content, questionnaires, learning
  sets, grades, and attendance list/create foundation.
- `005-backend-student-reporting`: student self-view, authorized student
  content download, report request/list/download, report retention, and
  explicit regeneration through new `ReportRun`.
- `006-backend-student-enrollment`: draft specification for student profile
  and enrollment management.
- `009-classroom-roster-foundation`: class-section/roster foundation,
  roster membership, teacher assignment, academic-period scoping, and
  read-only legacy direct-assignment compatibility.

## Recommended Sequence

### 1. Student Profile and Enrollment Management

**Status**: Spec started in `specs/006-backend-student-enrollment/spec.md`.

**Purpose**: Provide contract-governed student profile creation, listing,
detail, lifecycle status, transfer behavior, guardian-compatible associations,
and enrollment history.

**Why next**: Future classroom, roster, guardian, reporting, and correction
features depend on explicit student lifecycle rules instead of implicit
`StudentProfile` records.

**Contract gate**: Add OpenAPI operations for student profile list/create,
detail, status update, and transfer before backend implementation.

### 2. Administration Lifecycle Management

**Status**: Spec started in `specs/007-administration-lifecycle/spec.md`.

**Purpose**: Add detail, update, activate/deactivate, delete/restore, and
selected bulk operations for users, roles, academic years, academic periods,
guardians, and schools where allowed.

**Why next**: The current backend only covers list/create for most school-admin
resources. Real administration requires lifecycle maintenance with tenant-safe
history.

**Contract gate**: Define operation IDs, allowed transitions, conflict rules,
soft-delete behavior, and response envelopes for each affected resource.

### 3. Account Lifecycle Workflows

**Status**: Spec started in `specs/008-account-lifecycle-workflows/spec.md`.

**Purpose**: Define invitations, password setup, password reset, account
reactivation, and account lock or recovery behavior.

**Why next**: User creation exists, but operational onboarding and recovery
flows remain blocked. These rules affect security, audit events, and frontend
auth behavior.

**Contract gate**: Define token lifetime, delivery assumptions, audit events,
inactive-user behavior, and allowed actor roles before implementation.

### 4. Classroom, Course, Section, and Roster Foundation

**Status**: Completed and implemented from
`specs/009-classroom-roster-foundation/spec.md`.

**Purpose**: Introduce class/course/section/group models, roster membership,
teacher assignment, and academic-period scoping.

**Why next**: Teacher workflows currently assign learning sets, grades, and
attendance directly to selected student profiles. Group-based teaching needs a
formal roster model before any workflow can depend on it.

**Contract gate**: Define tenant ownership, enrollment links, teacher
assignment rules, active/inactive behavior, and migration impact on existing
direct student assignments.

### 5. Teacher Workflow Lifecycle and Corrections

**Status**: Completed and implemented from
`specs/010-teacher-workflow-lifecycle/spec.md`.

**Purpose**: Add detail, update, deactivate/activate, delete/restore, download,
bulk import, and correction workflows for teacher content, questionnaires,
learning sets, grades, and attendance.

**Why next**: The current P2 backend supports initial list/create behavior but
does not support operational maintenance or corrections after records exist.

**Contract gate**: Define immutable versus editable fields, closed-period
correction authority, audit history, soft-delete rules, and student visibility
impact.

### 6. Guardian Self-Service

**Status**: Completed and implemented from
`specs/011-guardian-self-service/spec.md`.

**Purpose**: Allow guardians to view permitted student profile, academic, and
contact information without gaining school-admin or student self-view powers.

**Why next**: Guardian records exist, but guardian access is explicitly outside
the current backend scope. This requires a separate authorization model.

**Contract gate**: Define guardian authentication, student association proof,
data visibility limits, consent or school approval rules, and cross-tenant
behavior.

### 7. Report Lifecycle Expansion

**Status**: Ready for implementation from
`specs/012-report-lifecycle-expansion/spec.md`.

**Purpose**: Add report retry, cancellation, soft delete/restore, report
catalog discovery, custom report definitions, custom report request snapshots,
and approved XLSX/filter expansion.

**Why next**: Current reports are limited to launch-scope asynchronous runs,
PDF/CSV output, 90-day retention, and explicit regeneration through a new
`ReportRun`.

**Contract gate**: Define report lifecycle state transitions, permissions,
retention impact, retry semantics, output compatibility, custom-definition
ownership, and tenant visibility.

### 8. Platform-Wide Reporting and Support Access

**Status**: Not specified.

**Purpose**: Define platform administrator or support-user visibility across
schools for operational oversight.

**Why next**: Existing rules intentionally prevent platform administrators from
implicitly bypassing school-scoped student, teacher, and report permissions.

**Contract gate**: Define allowed platform views, audit obligations,
redaction/minimization rules, school opt-in if needed, and explicit
authorization exceptions.

### 9. Advanced Assessment and Content Types

**Status**: Not specified.

**Purpose**: Add questionnaire types beyond multiple choice, true/false, and
short text; support long-text or file-response answers; expand content
processing where needed.

**Why next**: The current v1 assessment model is intentionally narrow. Any new
question or answer type affects validation, storage, student experience, and
reporting.

**Contract gate**: Define answer schemas, grading rules, file handling,
malware scanning, storage visibility, and student/reporting impact.

## Scope Still Outside Backend Roadmap

These are not recommended backend feature candidates until product scope
changes:

- Payroll, billing, accounting, and financial management.
- Messaging, notifications, live classroom, video conferencing, and parent
portal surfaces beyond explicitly approved guardian self-service.
- Frontend implementation work.
- Undocumented API behavior in any repository.

## Usage Rules

- Start each roadmap item with `speckit-specify` before planning or tasks.
- Expand OpenAPI before backend implementation exposes new behavior.
- Keep `school_id` as the v1 tenant column for school-owned records.
- Preserve platform versus school authorization separation.
- Keep backend and frontend repositories pinned to an approved
  `schoolmaster-specs` revision.

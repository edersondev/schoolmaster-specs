# Research: Backend Teacher Workflow Lifecycle and Corrections

## Decision: Expand OpenAPI before backend exposure

**Decision**: Add or update OpenAPI operations, schemas, response envelopes, error cases, authorization notes, and lifecycle/correction/import semantics before any backend route exposes this behavior.

**Rationale**: The current teacher workflow foundation only approved list/create behavior. This feature adds detail, update, lifecycle, download, import, and correction workflows that are externally visible and must be contract-governed.

**Alternatives considered**:
- Implement backend routes first and document later: rejected because it violates API-first governance.
- Reuse existing list/create operations for lifecycle behavior: rejected because it would hide new state transitions and errors from clients.

## Decision: Owner-scoped teacher management with school-administrator override

**Decision**: Creating/owning teachers may manage their own teacher workflow records within an active permitted school context; school administrators may manage same-school records.

**Rationale**: This preserves teacher ownership while allowing operational administration. It avoids same-school lateral access where one teacher can mutate another teacher's materials or records.

**Alternatives considered**:
- Any same-school teacher can manage records: rejected for excessive lateral authority.
- School administrators only: rejected because teachers need operational maintenance for their own records.
- Active roster teachers can manage same-roster records: rejected because it adds delegated authority rules not yet specified.

## Decision: Reject historical-meaning edits after use

**Decision**: Content and questionnaire edits that would change historical student-facing meaning are rejected once the item is used by a published or assigned learning set.

**Rationale**: Rejecting these edits is simpler and safer for v1. It preserves student-visible history without introducing version trees, migration rules, or separate version visibility semantics.

**Alternatives considered**:
- Auto-create a new version: rejected for v1 because it adds data model and visibility complexity.
- Allow edits with before/after correction history: rejected because it still changes historical learning context for students.

## Decision: Roster-aware learning-set writes only

**Decision**: New learning-set assignment writes in this slice use approved ClassSection/Roster membership context. Existing direct selected-student assignments remain readable as legacy records only.

**Rationale**: The completed roster foundation exists specifically to replace new direct student assignment writes for group-based teaching. Keeping legacy reads avoids breaking existing student self-view and reporting behavior.

**Alternatives considered**:
- Continue direct selected-student writes: rejected because it bypasses the roster foundation.
- Migrate legacy reads immediately: rejected because migration/backfill needs a separate specification and contract update.

## Decision: Shared lifecycle states and restore behavior

**Decision**: Teacher content, questionnaires, learning sets, grades, and attendance use `active`, `inactive`, and `deleted`. Delete moves records to `deleted`; restore moves records to `inactive`; activation is a separate transition.

**Rationale**: A shared model keeps contract and tests consistent across teacher workflow resources. Restore-to-inactive prevents accidentally making restored records student-facing before current tenant, dependency, scan, roster, academic-period, and visibility rules are rechecked.

**Alternatives considered**:
- Restore to previous state: rejected because prior active state may no longer be valid.
- Resource-specific lifecycle states: rejected for v1 because it increases contract complexity without clear product value.
- Correction-only history for grades and attendance: rejected because the feature explicitly includes deactivate/activate and delete/restore behavior for all teacher workflow resources.

## Decision: School-administrator-only closed-period corrections

**Decision**: Closed-period grade and attendance corrections are limited to authorized school administrators and require a free-text reason from 10 to 500 characters inclusive.

**Rationale**: Closed periods are academically sensitive. School-administrator-only authority provides a clear v1 control point while still allowing operational correction. Free-text reasons keep the contract light and auditable without prematurely defining a reason taxonomy that schools may need to customize later.

**Alternatives considered**:
- Teachers submit corrections requiring administrator approval: rejected for v1 because it introduces approval states and queues.
- Prohibit closed-period corrections: rejected because operational schools need controlled correction capability.
- Fixed reason-code catalog: rejected because a useful taxonomy needs product discovery and school customization rules.
- Reason code plus optional note: rejected for v1 because it adds taxonomy maintenance without immediate need.

## Decision: Correction reasons use bounded free text

**Decision**: Grade and attendance correction reasons are required free text from 10 to 500 characters inclusive.

**Rationale**: The lower bound prevents empty or non-informative audit notes, while the upper bound keeps request payloads and audit summaries manageable. This supports accountability without defining a rigid catalog too early.

**Alternatives considered**:
- Optional free text: rejected because corrections require accountability.
- Fixed reason code catalog: rejected because the domain taxonomy is not yet approved.
- Longer unrestricted note: rejected because it increases privacy and payload risk.

## Decision: Create-only JSON grade and attendance imports, school-administrator-only, capped at 500 rows

**Decision**: Bulk imports are limited to create-only JSON payload imports for new grades and attendance records, may be run only by authorized school administrators, accept no more than 500 rows per import, and are all-or-nothing.

**Rationale**: Grades and attendance are high-volume records where import is most valuable. JSON payloads keep validation, schema documentation, error reporting, and all-or-nothing behavior precise without adding file parsing and upload-security scope. Create-only import keeps bulk creation separate from correction workflows, which require explicit reasons and correction history. Administrator-only authority reduces risk, and the 500-row cap bounds validation, transaction, and error reporting complexity.

**Alternatives considered**:
- Include content, questionnaires, and learning sets: rejected because they have richer dependency and file/sequence semantics.
- Allow teachers to import roster-limited records: rejected because imports can affect many students and should begin with stronger administrative control.
- CSV or spreadsheet file upload: rejected for v1 because file parsing and upload-security behavior would expand this slice.
- Import-based correction of existing records: rejected because corrections already have explicit reason and audit workflows.
- Partial success imports: rejected because partial writes complicate reconciliation and audit.

## Decision: Audit successful and denied teacher content downloads

**Decision**: Record tenant-safe audit events for both successful and denied teacher content download attempts.

**Rationale**: Downloads access private tenant-scoped files. Auditing both outcomes supports security review without exposing file contents, storage paths, credentials, full request payloads, or unauthorized cross-tenant details.

**Alternatives considered**:
- Audit only successful downloads: rejected because denied attempts are useful security signals.
- Audit only denied downloads: rejected because successful private file access also needs traceability.
- Do not audit downloads: rejected because content download is a sensitive file-access event.

## Decision: Current student views show active records only

**Decision**: Current student self-view shows `active` teacher workflow records only. `Inactive` and `deleted` records are hidden from current student views except where OpenAPI explicitly documents historical labels.

**Rationale**: Active-only current views keep lifecycle behavior predictable and prevent soft-deleted or inactive instructional and academic records from remaining visible as current student work. Historical labels remain possible where the contract explicitly defines a safe student-facing history.

**Alternatives considered**:
- Show active and inactive records in current views: rejected because inactive is an operational non-current state.
- Show all lifecycle states with status labels: rejected because deleted records should not remain visible as current student-facing records.

## Decision: Use existing Laravel service and policy boundaries

**Decision**: Implement lifecycle, correction, import, and download rules through Laravel services, Form Requests, Policies, API Resources, DTOs, and focused repositories/query objects where needed.

**Rationale**: This follows the project constitution and existing backend slices. The workflows include multi-field validation, tenant checks, ownership checks, dependency checks, and audit writes that should not live in controllers.

**Alternatives considered**:
- Controller-local business logic: rejected by constitution and maintainability requirements.
- Broad repositories for all teacher workflow access: rejected unless query complexity justifies them.

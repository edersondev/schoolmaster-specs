# Quickstart: Platform-Wide Reporting and Support Access

## Purpose

Use this walkthrough to verify that the platform-wide reporting and support access plan, contract boundary, and backend implementation remain aligned before implementation proceeds.

## Prerequisites

- Active feature branch: `013-platform-support-access`
- Active spec: `specs/013-platform-support-access/spec.md`
- Active plan: `specs/013-platform-support-access/plan.md`
- OpenAPI changes must be completed before backend routes expose platform/support behavior.
- Backend implementation must remain API-only and scoped to the backend repository.
- Frontend implementation is out of scope for this slice.

## Contract Validation

After OpenAPI is expanded for this feature, validate the aggregate contract:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

If validating individual files directly, use:

```bash
npx @redocly/cli lint api/openapi.yaml
npx @redocly/cli lint specs/001-schoolmaster-platform/contracts/openapi.yaml
```

### Validation Results

- Pending: Run after OpenAPI operations and schemas are added.

Contract review must confirm:

- operation IDs exist for every approved platform/support operation
- all routes remain under `/api/v1`
- platform school summaries expose only minimized approved fields
- cross-school reporting overview exposes only approved aggregate report health, lifecycle, retention, output availability, and failure-summary metrics
- protected counts below 5 are suppressed
- target-school opt-in is required for support drill-down
- target-school opt-in uses dedicated school-scoped support opt-in operations
- `requestSupportAccess` can create or return a requested or pending decision before internal platform approval exists
- internal platform approval is required for support drill-down
- support access approval and target-school opt-in expire after 24 hours
- stale, revoked, expired, mismatched, concurrently changed, and older-than-24-hour approvals or opt-ins return documented denial or conflict responses before diagnostics are returned
- generated report downloads, raw report outputs, private file metadata, emergency access, unrestricted record search, support writes, and unrestricted impersonation are explicitly rejected
- platform/support audit events include actor, action, outcome, target school where applicable, correlation ID, reason code, and minimized metadata
- audit metadata excludes credentials, bearer tokens, private file paths, raw report outputs, private content, full student/guardian records, and unauthorized cross-tenant details

## Backend Verification

After backend implementation, run the backend test suite from the backend repository:

```bash
docker exec schoolmaster-backend-app-1 php artisan test
```

### Backend Validation Results

- Pending: Run after backend implementation.

Focused backend coverage must include:

- successful platform school summary retrieval by actor with explicit platform-wide operational oversight permission
- platform administrator without explicit platform-wide reporting/support permission denied
- cross-school reporting overview returns only approved aggregate health and lifecycle summaries
- protected counts below 5 are suppressed in platform summary and reporting overview responses
- unauthorized actors receive documented envelopes without cross-school data disclosure
- support drill-down succeeds only when support actor permission, target-school opt-in, internal platform approval, target-school match, reason code, and correlation ID are valid
- support access decision creation succeeds in a requested or pending state when target-school opt-in and request metadata are valid but internal platform approval has not yet been recorded
- missing target-school opt-in rejected before diagnostics are returned
- school-scoped support opt-in create and revoke require explicit same-school support opt-in permission
- missing internal platform approval rejected before diagnostics are returned
- support access approval older than 24 hours rejected before diagnostics are returned
- target-school opt-in older than 24 hours rejected before diagnostics are returned
- revoked, stale, expired, mismatched, and concurrently changed support access decisions or opt-ins rejected before diagnostics are returned
- support access decision for one school cannot be reused for another school
- support diagnostics remain read-only and redacted
- support users cannot download generated reports, receive raw report outputs, see private file metadata, use emergency access, perform unrestricted record search, impersonate school users, or write school-owned operational records
- support audit summary visible only to actors with explicit support audit review permission
- audit events written for allowed access, denied access, validation rejection, cross-school lookup, platform reporting summary access, support drill-down access, support escalation, support approval, support revocation, support expiration, and conflict outcomes
- audit metadata redacts credentials, tokens, private paths, raw outputs, private content, full student/guardian records, full request/response payloads, and unauthorized cross-tenant details
- existing school-scoped student, teacher, guardian, administration, and reporting endpoints remain inaccessible to platform/support actors without operation-specific future contract approval

## Backend Architecture Checklist

- Controllers are orchestration-only.
- Business rules live in `App\Services\PlatformSupport`.
- Request validation uses Form Requests.
- Authorization uses Policies.
- Responses use API Resources and documented envelopes.
- Multi-field inputs use DTOs where they improve clarity.
- Repositories/query objects are used only for complex cross-school aggregation, small-count suppression, support access decision lookup, and audit-safe reads.
- Public identifiers are UUIDs.
- School-owned reads remain scoped by `school_id`.
- Platform users do not receive implicit school-scoped endpoint access.
- Support drill-down is read-only, target-school-bound, and approval-gated.
- No frontend behavior is implemented in this slice.

## Out-of-Scope Guardrail

Reject implementation or contract additions for:

- frontend platform/support UI implementation
- support write actions against school-owned records
- account reactivation, report retry/cancel, school status changes, or other operational mutations through support access
- generated report downloads for support users
- raw report output or private file metadata visibility
- emergency or break-glass access
- unrestricted impersonation of school users
- unrestricted raw database or record search
- advanced assessment and content types
- billing, payroll, accounting, messaging, notifications, live classroom, or video conferencing behavior
- permanent purge, legal hold, or anonymization workflows

Any needed behavior from this list requires a separate specification and OpenAPI update.

# Data Model: Platform-Wide Reporting and Support Access

## Modeling Principles

- `School` remains the v1 tenant root.
- School-owned records continue to use `school_id`.
- Public identifiers crossing API boundaries use UUIDs.
- Platform-wide views read school-owned data only through documented platform operations.
- Platform users do not receive implicit access to existing school-scoped endpoints.
- Support drill-down is read-only, target-school-bound, and requires both school-scoped target-school opt-in and internal platform approval.
- Support drill-down approvals and target-school opt-ins expire after 24 hours.
- Protected platform-visible aggregate counts below 5 are suppressed.
- Generated report downloads, raw report outputs, private file metadata, emergency access, unrestricted impersonation, support writes, and unrestricted record search are not exposed in this slice.
- Platform support audit events store tenant-safe minimized metadata only.

## Entities

### PlatformOperationalSummary

- **Purpose**: Minimized cross-school operational view for platform oversight.
- **Core fields**:
  - `school_id` or public school UUID
  - school display name
  - school status
  - aggregate user/student/guardian/teacher/report counts where approved
  - report lifecycle state summary
  - report output availability summary
  - support diagnostic status indicators
  - suppressed-count indicators where protected counts are below 5
  - last activity timestamps where approved
- **Relationships**:
  - summarizes one `School`
  - may aggregate approved data from users, student profiles, guardians, teacher workflow records, report runs, report outputs, and support access events
- **Validation rules**:
  - requires explicit platform-wide operational oversight permission
  - exposes only OpenAPI-approved fields
  - suppresses protected counts below 5
  - must not expose raw report outputs, private file paths, full student/guardian records, private content, or unauthorized cross-tenant details
  - school-owned detail records remain hidden unless an operation-specific contract explicitly allows a redacted detail view

### PlatformReportingOverview

- **Purpose**: Aggregate cross-school reporting health view for platform administrators.
- **Core fields**:
  - report lifecycle status counts by school or filtered platform scope
  - failure-summary counts and tenant-safe reason codes where approved
  - output availability counts by format where approved
  - retention/expiry summary counts
  - suppressed-count indicators for protected counts below 5
  - filter and pagination metadata
- **Relationships**:
  - summarizes `ReportRun`, `ReportOutput`, `ReportDefinition`, and school status data through platform-approved views
- **Validation rules**:
  - requires explicit platform-wide reporting permission
  - supports only documented filters, sorting, and pagination
  - suppresses protected counts below 5
  - must not expose generated files, raw report contents, private storage metadata, custom report private payloads, full filter payloads, or school-owned detail records
  - does not grant report lifecycle action permission

### SupportAccessDecision

- **Purpose**: Target-school-specific authorization decision that allows read-only support drill-down when all gates are satisfied.
- **Core fields**:
  - `id` (UUID)
  - support actor user ID
  - target `school_id`
  - tenant-safe reason code
  - correlation ID
  - target-school opt-in state
  - internal platform approval state
  - lifecycle state (`requested`, `approved`, `denied`, `expired`, `revoked`)
  - approved timestamp
  - expires timestamp, maximum 24 hours after approval
  - revoked timestamp and revocation reason code where applicable
  - timestamps
- **Relationships**:
  - belongs to target `School`
  - belongs to support actor `User`
  - references one `TargetSchoolOptIn`
  - references one `InternalPlatformApproval`
  - has many `PlatformSupportAuditEvent`
- **Validation rules**:
  - support drill-down requires active `approved` state
  - support drill-down is denied when approval is missing, stale, expired, revoked, mismatched, older than 24 hours, or concurrently changed
  - one target-school decision cannot be reused for another school
  - approval does not grant write actions, report downloads, emergency access, raw record search, or impersonation

### TargetSchoolOptIn

- **Purpose**: School-side approval gate for support drill-down access.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - opt-in state (`pending`, `approved`, `denied`, `revoked`, `expired`)
  - requested_by_user_id where available
  - approved_by_user_id where available
  - reason code
  - correlation ID
  - approved timestamp
  - expires timestamp, maximum 24 hours after approval
  - timestamps
- **Relationships**:
  - belongs to `School`
  - may be created, denied, or revoked through dedicated school-scoped support opt-in operations
  - may be referenced by `SupportAccessDecision`
- **Validation rules**:
  - requires explicit same-school support opt-in permission
  - opt-in must apply to the same target school as the support access decision
  - revoked, denied, expired, older-than-24-hour, or mismatched opt-in blocks support drill-down
  - opt-in decisions are audited with minimized metadata

### InternalPlatformApproval

- **Purpose**: Platform-side approval gate for support drill-down access.
- **Core fields**:
  - `id` (UUID)
  - approver user ID
  - support actor user ID
  - target `school_id`
  - approval state (`pending`, `approved`, `denied`, `revoked`, `expired`)
  - reason code
  - correlation ID
  - approved timestamp
  - expires timestamp
  - timestamps
- **Relationships**:
  - belongs to approver `User`
  - belongs to support actor `User`
  - references target `School`
  - may be referenced by `SupportAccessDecision`
- **Validation rules**:
  - approval must target the same support actor and same school as the support access decision
  - approval expires no later than 24 hours after approval
  - revoked, denied, expired, or mismatched approval blocks support drill-down
  - approval decisions are audited with minimized metadata

### SupportDiagnosticView

- **Purpose**: Redacted read-only diagnostic payload for one target school.
- **Core fields**:
  - target school identity and status
  - approved operational diagnostic indicators
  - approved report health summaries
  - approved lifecycle state summaries
  - approved support troubleshooting metadata
  - suppressed-count indicators where counts are below 5
  - response correlation ID
- **Relationships**:
  - resolved from one active `SupportAccessDecision`
  - summarizes one target `School`
- **Validation rules**:
  - requires explicit support drill-down permission
  - requires target-school opt-in and internal platform approval
  - requires approval age of 24 hours or less
  - response is read-only and redacted
  - must not expose student detail, guardian detail, teacher detail, report-run detail, custom-report detail, school-admin detail, generated report downloads, raw report outputs, private file metadata, emergency access, unrestricted record search, or write actions unless a future contract explicitly adds a separate view

### PlatformSupportAuditEvent

- **Purpose**: Minimized audit event for platform overview, support access, support decisions, denials, validation failures, expiration, revocation, and conflicts.
- **Core fields**:
  - `id` (UUID)
  - actor user ID where available
  - action
  - outcome
  - target school ID where applicable and safe
  - target type
  - target ID where safe
  - correlation ID
  - tenant-safe reason code
  - timestamp
  - minimized metadata
- **Relationships**:
  - may reference actor `User`
  - may reference target `School`
  - may reference `SupportAccessDecision`
  - may reference `TargetSchoolOptIn`
  - may reference `InternalPlatformApproval`
- **Validation rules**:
  - required for allowed access, denied access, validation rejection, cross-school lookup, platform reporting summary access, support drill-down, support escalation, support approval, support revocation, support expiration, and conflict outcomes
  - must not store credentials, bearer tokens, private file paths, raw report outputs, private content, full student/guardian records, full request/response payloads, or unauthorized cross-tenant details
  - denied cross-tenant attempts are audited without storing unauthorized protected target details

### User

- **Purpose**: Existing actor identity for platform administrators, support users, platform approvers, and school opt-in actors.
- **Validation rules**:
  - inactive users cannot perform platform/support operations
  - platform overview requires explicit platform-wide operational oversight permission
  - cross-school reporting overview requires explicit platform-wide reporting permission
  - support drill-down requires explicit support drill-down permission plus approved target-school opt-in and internal platform approval
  - audit review requires explicit support audit review permission
  - platform role membership alone does not grant school-scoped endpoint access

### School

- **Purpose**: V1 tenant root and target of platform summaries and support drill-down.
- **Validation rules**:
  - platform summary may include approved minimized school identity/status fields
  - school-owned operational details stay behind school-scoped authorization or explicitly contracted redacted support diagnostics
  - inactive, suspended, or restricted school state must be represented only through approved minimized fields

## State Transitions

### SupportAccessDecision

```text
requested -> approved
requested -> denied
approved -> expired
approved -> revoked
approved -> denied (when stale, mismatched, or concurrently changed before access)
```

- `approved` requires matching target-school opt-in and internal platform approval, each no older than 24 hours.
- `approved` decisions expire after 24 hours.
- `expired`, `revoked`, `denied`, stale, mismatched, and concurrently changed decisions cannot return school-owned diagnostics.

### TargetSchoolOptIn

```text
pending -> approved
pending -> denied
approved -> revoked
approved -> expired
```

- `approved` target-school opt-ins expire after 24 hours.

### InternalPlatformApproval

```text
pending -> approved
pending -> denied
approved -> revoked
approved -> expired
```

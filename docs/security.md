# Security Guidelines

## Baseline

Security requirements must be reflected in specifications, OpenAPI contracts,
and implementation repositories.

## Core Expectations

- Validate backend input
- Protect routes with appropriate middleware
- Enforce authorization on tenant-scoped resources
- Keep secrets in environment configuration
- Sanitize uploaded content paths and metadata when applicable

## Contract Role

Security-sensitive behavior exposed to clients should be documented in OpenAPI
before backend and frontend implementation diverge.

## Authentication and Session Rules

- Authentication establishes the user identity and, for school-scoped users,
  the resolved school context.
- First-slice bearer tokens expire 8 hours after issuance.
- Logout revokes continued access for the current bearer token.
- Inactive users cannot authenticate or continue protected workflows.
- Inactive schools reject school-scoped operational workflows before
  module-specific logic runs.
- Expired, revoked, inactive-user, and inactive-school tokens must return the
  documented token rejection response envelope.
- Failed login attempts are counted by both submitted email and source IP.
  Login locks after 5 failed attempts for either key within 15 minutes and
  returns the documented lockout envelope with retry metadata.
- Token rotation and refresh behavior remain out of scope until specified in
  OpenAPI.

## Authorization Rules

- Permissions are assigned to roles; direct per-user permission assignment is
  outside v1 scope.
- Platform-scoped roles may grant platform capabilities only.
- School-scoped roles may grant school capabilities only and are effective only
  within the resolved active school tenant.
- System administrator access does not bypass school-scoped authorization unless
  a platform override is explicitly documented and tested.

## Upload Security

- Uploaded instructional files must be validated for declared type, detected
  content type, size, tenant ownership, and authorization before persistence.
- Teacher content is stored in private tenant-scoped storage and served only
  through authorized API access.
- Allowed file types, maximum size, malware scanning expectations, and
  rejected-file behavior must be defined before teacher content implementation.

## Teacher Workflow Security

- Teacher content downloads must require authenticated same-school authority,
  clean malware-scan status, and an allowed active lifecycle state before any
  file delivery metadata is issued.
- Denied teacher content downloads must not expose private storage paths,
  bucket keys, raw file metadata, scan internals, or cross-tenant record
  existence.
- Correction reasons for grades and attendance are operationally sensitive and
  must be stored and returned only through approved teacher workflow
  management responses, never through student self-view payloads.
- Teacher workflow imports must accept JSON payloads only in v1 and must return
  tenant-safe row error summaries that avoid exposing unauthorized student,
  teacher, roster, content, or academic data.
- Import requests that contain any invalid, duplicate, unauthorized,
  cross-tenant, or closed-period-restricted row must reject the entire import
  without partial writes.

## Audit and Monitoring

- Tenant-sensitive workflows should record enough context to investigate
  access, authorization, upload, report generation, and administrative changes.
- The first backend slice records audit events for login success, login
  failure, logout, token rejection, school create, school update, school
  activation, and school deactivation.
- Authentication and school lifecycle audit events must not store plaintext
  credentials or bearer token values.
- Audit event schemas and retention rules remain module-level decisions and
  must be specified before implementation where they affect product behavior.

## Teacher Workflow Audit Requirements

- Audit successful and denied teacher content downloads, lifecycle transitions,
  delete and restore actions, grade and attendance corrections, import
  submissions, import rejections, validation rejections, conflict outcomes, and
  blocked cross-tenant attempts.
- Teacher workflow audit entries must include actor, action, outcome, target,
  school, and tenant-safe summary metadata only.
- Teacher workflow audit entries must not store private file contents, private
  storage paths, credentials, full import payloads, or unauthorized
  cross-tenant details.

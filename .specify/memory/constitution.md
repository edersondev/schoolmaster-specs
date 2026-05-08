<!--
Sync Impact Report
Version change: 1.0.0 -> 1.0.0
Modified principles:
- I. Contract-First Split Architecture -> I. API-First Contract Governance
- II. Laravel Backend Boundaries -> II. Laravel Backend Architecture and Coding Standards
- III. Vue Frontend Module Boundaries -> III. Vue Frontend Architecture and Coding Standards
- IV. MySQL and Tenant-Safe Data Design -> IV. Multi-Tenant Data and Access Boundaries
- V. Verified Critical Flows -> V. Verification and Release Gates
Added sections:
- API Standards
- Repository Standards
- Delivery Workflow
Removed sections:
- Delivery Constraints
- Engineering Workflow
Templates requiring updates:
- ✅ updated: .specify/templates/plan-template.md
- ✅ updated: .specify/templates/spec-template.md
- ✅ updated: .specify/templates/tasks-template.md
- ✅ reviewed: .specify/templates/commands/*.md (directory absent; no updates required)
- ✅ reviewed: AGENTS.md (no conflicting runtime guidance found in the tracked file)
Follow-up TODOs:
- None
-->
# SchoolMaster Constitution

## Core Principles

### I. API-First Contract Governance
SchoolMaster MUST keep the backend API, frontend application, and shared
specification artifacts in separate repositories. Every externally visible API
change MUST begin in the OpenAPI contract before implementation starts, and
public endpoints MUST remain versioned under `/api/v1` until a later amendment
defines a new versioning policy. Backend and frontend teams MUST treat the
OpenAPI document as the source of truth for payloads, status codes,
authentication expectations, and error semantics. Rationale: explicit
contract governance keeps independently deployed repositories aligned and
prevents either side from redefining system behavior by convention.

### II. Laravel Backend Architecture and Coding Standards
Backend services MUST use Laravel and organize application code by feature.
Controllers MUST remain orchestration-only and MUST NOT contain business
rules. Business logic MUST live in `App\Services`, multi-field or workflow
inputs MUST use DTOs when they improve clarity, and Repositories MUST be
introduced only for genuinely complex data access patterns. Validation MUST be
enforced with Form Requests, authorization MUST be enforced with Policies,
public API output MUST be normalized with API Resources, and domain failures
MUST use exceptions instead of return-code branching. New or materially
modified PHP classes SHOULD declare strict types when practical, except where
Laravel framework constraints make that unreasonable. Public API identifiers
MUST use UUIDs. Rationale: these rules keep transport, policy, persistence, and
domain logic isolated, reviewable, and stable as the API surface expands.

### III. Vue Frontend Architecture and Coding Standards
Frontend applications MUST use Vue 3 Composition API with `<script setup>`,
Pinia, Vue Router, Axios, and Tailwind CSS. Frontend code MUST be organized by
feature module. Components MUST remain focused on presentation and interaction;
they MUST NOT contain business rules or direct HTTP calls. API access MUST be
isolated in services, and stores MAY coordinate state but MUST NOT become ad
hoc transport layers that bypass service contracts. Rationale: feature-local
composition and service isolation keep UI code maintainable while preserving a
clear boundary with the versioned backend API.

### IV. Multi-Tenant Data and Access Boundaries
MySQL MUST be the authoritative relational store. Every tenant-owned record,
query, policy, background process, and service workflow MUST make tenant scope
explicit, and the default implementation stance MUST be deny-by-default for
cross-tenant access. Any platform-wide or support-user override MUST be
documented in the spec, enforced intentionally in authorization, and covered by
tests. Recoverable business records MUST use soft deletes unless the
implementation plan documents and justifies a permanent deletion path. File
uploads and imported data MUST be validated and sanitized before persistence or
further processing. Rationale: explicit tenancy and data handling rules protect
institutional boundaries and reduce future migration or isolation failures.

### V. Verification and Release Gates
Critical business flows MUST have automated coverage before merge. Changed REST
contracts MUST have OpenAPI and response-shape verification, backend behavior
MUST have PHPUnit feature or unit coverage, and frontend behavior MUST have
Vitest coverage for the affected services, stores, or composables. No
repository MAY merge a change that knowingly breaks the published contract or
its dependent repository without an approved migration or rollout plan.
Rationale: SchoolMaster ships across separate repositories, so contract,
backend, and frontend verification must move together to keep releases safe.

## API Standards

- REST APIs MUST return consistent JSON structures for success and failure.
- Error responses MUST expose machine-readable codes, human-readable messages,
  and field-level validation details when validation fails.
- Additive changes MAY ship within the active API version, but breaking changes
  MUST introduce an explicit versioning or migration plan before implementation.
- Authentication, authorization, pagination, filtering, sorting, and tenancy
  semantics MUST be documented in OpenAPI whenever a feature affects them.

## Repository Standards

- The backend API, frontend application, and specification repository MUST
  remain independently releasable and MUST NOT depend on direct source imports
  across repositories.
- Every feature spec, plan, and task list MUST identify which repositories
  change, which repository leads the delivery, and how related work is linked
  across repositories.
- Cross-repository work MUST share a common feature identifier, issue linkage,
  or equivalent traceability so reviewers can reconstruct the full delivery.

## Delivery Workflow

- Specifications MUST identify backend repository impact, frontend repository
  impact, specification or contract repository impact, OpenAPI impact, tenant
  impact, and required critical-flow tests.
- Implementation plans MUST fail Constitution Check when they omit Service
  Layer usage, Form Requests, Policies, API Resources, DTO or Repository
  decisions where applicable, UUID boundaries, service-isolated frontend API
  access, repository coordination, or tenant-safety considerations.
- Task lists and reviews MUST show how PHPUnit, Vitest, and contract
  verification cover each changed critical business flow across the affected
  repositories.
- Deviations from this constitution MUST be documented in Complexity Tracking
  or equivalent review notes and approved before implementation.

## Governance

This constitution supersedes conflicting local workflow guidance for
SchoolMaster. Amendments MUST be proposed in pull requests that update this
document and any affected templates or guidance files in the same change.
Versioning follows semantic versioning for governance: MAJOR for incompatible
principle changes or removals, MINOR for new principles or materially expanded
requirements, and PATCH for clarifications that do not alter obligations.
Compliance review is mandatory for every specification, implementation plan,
task list, and pull request; any non-compliant item MUST either be corrected or
carry an explicit, approved exception with rationale.

**Version**: 1.0.0 | **Ratified**: 2026-05-08 | **Last Amended**: 2026-05-08

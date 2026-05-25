# Quickstart: Backend API Foundation

## Purpose

Use this guide to validate backend readiness before implementation begins for authentication and school management.

## Prerequisites

- Current branch: `002-backend-api-foundation`
- Active spec: `specs/specs/002-backend-api-foundation/spec.md`
- Active plan: `specs/specs/002-backend-api-foundation/plan.md`
- Source-of-truth specs mounted at `specs/`

## Source-of-Truth Review

Read these files before coding:

```bash
sed -n '1,220p' specs/AGENTS.md
sed -n '1,260p' specs/specs/001-schoolmaster-platform/spec.md
sed -n '1,220p' specs/docs/backend-guidelines.md
sed -n '1,220p' specs/docs/multi-tenant.md
sed -n '1,220p' specs/docs/security.md
sed -n '1,220p' specs/decisions/002-use-laravel-native-auth.md
sed -n '1,220p' specs/decisions/003-use-mysql.md
sed -n '1,220p' specs/decisions/004-use-tenant-by-column.md
```

## Contract Validation

Validate the aggregate and active feature contracts before backend implementation merges:

```bash
npx @redocly/cli lint specs/api/openapi.yaml
npx @redocly/cli lint specs/specs/001-schoolmaster-platform/contracts/openapi.yaml
```

Expected result: both contracts validate. Existing metadata warnings, such as a missing `info.license`, should be recorded but do not by themselves define backend behavior.

Latest result, 2026-05-16: both `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml` validated successfully with Redocly after adding logout revocation, 8-hour token expiry, token rejection, login lockout, and audit-relevant auth semantics.

Latest backend implementation result, 2026-05-16: both aggregate and active
feature OpenAPI contracts validated successfully with Redocly. Each contract
still reports the existing `info.license` warning only.

## Required Contract Sync Before Auth Coding

Before implementing `login` or `getCurrentUser`, update `/specs` and OpenAPI to document:

- 8-hour bearer-token expiry
- logout operation and response semantics
- logout revocation behavior
- inactive-user and inactive-school token rejection
- failed-login lockout after 5 attempts per email or IP within 15 minutes
- lockout response envelope and retry metadata
- token rejection response envelope for expired, revoked, inactive-user, and inactive-school cases
- audit event expectations for auth and school lifecycle changes

Run contract validation again after those updates.

## Backend Readiness Checks

Confirm the repository is prepared as an API backend before product endpoints are implemented:

```bash
php artisan route:list
docker exec schoolmaster-backend-app-1 php artisan test
```

Readiness expectations:

- Product API routes are versioned under `/api/v1`.
- No product endpoint exists outside the approved OpenAPI scope.
- Product Blade views are not used for SchoolMaster features.
- Backend setup documentation identifies MySQL, `/specs`, test commands, and contract validation commands.
- `routes/api.php` is the product API route entry point.
- The first product slice is limited to authentication and school management operations documented in OpenAPI.

Latest route result, 2026-05-16: `php artisan route:list` shows the approved
first-slice product routes under `/api/v1`:

- `POST /api/v1/auth/login`
- `GET /api/v1/auth/me`
- `POST /api/v1/auth/logout`
- `GET /api/v1/schools`
- `POST /api/v1/schools`
- `GET /api/v1/schools/{schoolId}`
- `PATCH /api/v1/schools/{schoolId}`

Current backend test command: `docker exec schoolmaster-backend-app-1 php
artisan test`.

Latest Docker/MySQL test result, 2026-05-16: the suite passed with 17 tests and
88 assertions against the `schoolmaster_testing` MySQL database. The host PHP
runtime still has no PDO drivers, so database-backed validation must run inside
the backend Docker container.

## First Slice Contract Boundary

Only these operations are in the first backend implementation boundary:

- `login`
- `getCurrentUser`
- `logout`
- `listSchools`
- `createSchool`
- `getSchool`
- `updateSchool`

Token rotation and refresh behavior remain out of scope unless `/specs` and OpenAPI are updated first.

## Security Validation

Backend design and tests must verify:

- bearer tokens expire after 8 hours
- logout revokes continued access
- inactive users cannot authenticate or continue protected workflows
- users tied to inactive schools cannot authenticate or continue school-scoped workflows
- failed login attempts are counted by submitted email and source IP
- lockout occurs after 5 failed attempts within 15 minutes for either key
- login success, login failure, logout, token rejection, and school lifecycle changes create audit events
- audit events do not store plaintext passwords or bearer token values

## Tenancy Validation

Backend design and tests must verify:

- `School` is the v1 tenant root.
- `school_id` is the v1 concrete tenant column for school-owned records.
- `tenant_id` is only a generic architecture term unless a future ADR changes the tenant root.
- Missing, mismatched, inactive, or unauthorized tenant context fails before tenant-owned data access.
- System administrators can perform platform-scope school provisioning only through explicit authorization.

## Exit Criteria

- This plan, research, data model, contracts boundary, and quickstart are present.
- `AGENTS.md` points to this active plan between the Speckit markers.
- OpenAPI validation can be run and results recorded.
- OpenAPI is updated for token expiry, revocation, lockout, token rejection, and audit behavior before authentication coding.
- Generated implementation tasks remain bounded to this plan and must preserve the contract-first guardrails before backend product coding.

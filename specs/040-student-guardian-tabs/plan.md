# Implementation Plan: Student Guardian Tabs

**Branch**: `040-student-guardian-tabs` | **Date**: 2026-09-01 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/040-student-guardian-tabs/spec.md`

## Summary

Move admin guardian capture into Create Student. Backend contract work updates
student creation so a single student create request can create new guardians or
link existing same-school guardians, with zero to two guardian entries and no
partial success. Frontend work follows after backend readiness: remove the
standalone Guardians sidebar item, redirect direct guardian admin routes to
Create Student, and convert Create Student into Student and Guardians tabs with
permission-aware guardian capture. The same feature converts the Student
Profile detail page into Student and Guardians tabs that render the documented
`guardian_associations` response read-only.

## Technical Context

**Language/Version**: PHP 8.3/Laravel API; JavaScript with Vue 3 Composition API and `<script setup>`
**Primary Dependencies**: OpenAPI/Redocly, Laravel validation/policies/services/resources/PHPUnit, Vue Router/Pinia/Axios/Element Plus/Tailwind CSS/Vitest
**Storage**: MySQL with `school_id` tenant ownership; no planned schema change because existing student, guardian, and guardian association records are reused
**Testing**: Redocly contract validation, PHPUnit feature/unit coverage, Vitest frontend coverage, production build, focused browser and responsive review, timed usability check, focused frontend performance timing
**Target Platform**: Versioned `/api/v1` Laravel API and responsive Vue SPA administration shell
**Project Type**: Cross-repository OpenAPI, Laravel API, and Vue SPA change
**Performance Goals**: Create Student route shows stable form, blocked state, or recoverable error within 2 seconds after app bootstrap; existing-guardian search returns a usable page within 2 seconds under normal admin list latency; valid student-with-guardians submission settles with success or actionable feedback without duplicate submits
**Constraints**: OpenAPI before backend, backend gate before frontend, no standalone Guardians sidebar entry, direct standalone guardian admin routes redirect to Create Student, zero to two guardians allowed, third guardian blocked, guardian capture requires guardian management authority, no partial student/guardian success, no direct Axios outside services, no undocumented request fields or endpoints
**Scale/Scope**: One student-create contract expansion, one backend validation/service transaction update, one guardian lookup use from existing list contract, one Create Student tabbed workflow, one navigation removal, one redirect route behavior, one Student Profile detail tabbed view, and focused backend/frontend verification

## Constitution Check

*GATE: PASS before Phase 0; re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. The `createStudentProfile` request must
  publish zero to two guardian entries and support both new guardian data and
  existing same-school guardian references before implementation.
- PASS: Repository impacts are separated. `schoolmaster-specs` owns spec,
  OpenAPI, and planning artifacts; `schoolmaster-backend` owns contract-backed
  validation, authorization, and atomic persistence; `schoolmaster-frontend`
  owns navigation, redirect, tabs, service mapping, and UI tests.
- PASS: Backend contract, implementation, and verification are sequenced before
  frontend implementation. Frontend begins only after backend readiness is
  recorded.
- PASS: Affected implementation repositories use
  `040-student-guardian-tabs`. No unaffected repository branch is planned.
- PASS: Backend design uses Laravel Form Request validation, Policy
  authorization, Service Layer transaction orchestration, API Resource output,
  UUID public identifiers, and no Repository unless implementation discovers
  complex reusable data access.
- PASS: Frontend design uses Vue 3 Composition API, Vue Router, existing Pinia
  session/shell stores, Axios service modules, Tailwind CSS, Element Plus, and
  route-local form state.
- PASS: MySQL, `school_id`, tenant scoping, cross-tenant denial, and soft-delete
  expectations are documented. No cross-tenant access path is approved.
- PASS: API compatibility, auth/permission behavior, success shape, validation,
  forbidden, tenant-mismatch, not-found, conflict, and temporary failure
  expectations are documented.
- PASS: Redocly, PHPUnit, Vitest, build, route, responsive, timed usability,
  performance timing, and privacy verification cover changed critical flows.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/040-student-guardian-tabs/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── student-guardian-tabs-contract.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Phase 2 output (/speckit-tasks command)
```

### Source Code (target repositories)

```text
# Specification repository
schoolmaster-specs/
├── api/
│   ├── openapi.yaml
│   ├── paths/student-profiles/
│   └── components/schemas/student-profiles/
├── specs/040-student-guardian-tabs/
└── AGENTS.md

# Backend repository
schoolmaster-backend/
├── app/
│   ├── Http/
│   │   ├── Controllers/Api/V1/
│   │   ├── Requests/
│   │   └── Resources/
│   ├── Models/
│   ├── Policies/
│   └── Services/
├── routes/api.php
└── tests/
    ├── Feature/
    └── Unit/

# Frontend repository
schoolmaster-frontend/
├── src/
│   ├── components/admin-system/
│   │   ├── students/
│   │   └── guardians/
│   ├── composables/admin-system/
│   ├── contracts/admin-system/
│   ├── pages/admin-system/
│   │   ├── students/
│   │   └── guardians/
│   ├── router/modules/
│   └── services/admin-system/
└── tests/unit/admin-system/
```

**Structure Decision**: Keep source changes inside existing student and
guardian administration boundaries. Backend updates the existing student
profile creation boundary rather than adding a new public workflow endpoint.
Frontend keeps Create Student as the owner of the tabbed workflow, uses the
existing guardians service only for same-school lookup, and removes sidebar
exposure from guardian route metadata.

## Implementation Approach

### Phase 0: Contract and Research

Confirm the published behavior for existing student, guardian, and guardian
association models. Publish the student-create request expansion in OpenAPI:
zero to two guardian entries, each either a new guardian payload or an existing
same-school guardian reference, and one atomic success or failure.

### Phase 1: Backend Gate

Implement request validation, policy checks, tenant-first lookup, active
same-school existing-guardian validation, new guardian creation, association
creation, duplicate detection, two-guardian maximum, transaction atomicity, and
response shaping. Verify no partial student or guardian result is committed
when any guardian entry fails.

### Phase 2: Frontend After Backend Verification

Remove the standalone Guardians sidebar item, redirect direct guardian admin
routes to Create Student, and convert Create Student into a route-local tabbed
form. The Student tab owns student fields; the Guardians tab supports zero to
two entries, new guardian data, existing same-school guardian selection,
guardian permission gating, tab-level validation badges or summaries, stale
request handling, and safe feedback.

### Phase 3: Student Profile Detail Tabs (Frontend Only)

Convert the Student Profile detail page into Student and Guardians tabs. The
Student tab keeps the existing student summary and enrollment status panels;
the Guardians tab renders the zero to two `guardian_associations` records
already returned and documented by `getStudentProfile` as a read-only panel.
No backend or OpenAPI change is required for this phase.

## Post-Design Constitution Check

- PASS: OpenAPI contract work leads backend and frontend changes.
- PASS: Backend readiness remains an explicit gate before frontend delivery.
- PASS: Service and policy boundaries are preserved in both implementation
  repositories.
- PASS: Tenant isolation and no-sensitive-data diagnostics remain testable.
- PASS: No unresolved clarification or constitution deviation remains.

## Complexity Tracking

No constitution deviations.

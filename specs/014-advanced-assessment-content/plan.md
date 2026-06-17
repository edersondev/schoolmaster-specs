# Implementation Plan: Advanced Assessment and Content Types

**Branch**: `014-advanced-assessment-content` | **Date**: 2026-06-11 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/014-advanced-assessment-content/spec.md`

**Note**: This plan defines the backend implementation and contract boundary for advanced assessment and content types. It does not decompose tasks and does not authorize behavior outside the specification and OpenAPI contract.

## Summary

Implement a backend-only advanced assessment slice for SchoolMaster. The backend must expand OpenAPI before exposing `long_text` and `file_response` questionnaire behavior, student response submission, private answer-file upload and scan gating, teacher/school-administrator review, manual 0-100 grading, single-attempt enforcement, student-safe assessment summaries, report-safe aggregate assessment fields, and tenant-safe audit events.

The design preserves existing `multiple_choice`, `true_false`, and `short_text` behavior; preserves questionnaire and learning-set lifecycle rules; keeps `School` as the v1 tenant root with `school_id` on school-owned records; rejects cross-tenant access before data or file exposure; excludes frontend implementation; excludes guardian advanced-assessment visibility; excludes report output file packaging; and keeps platform/support visibility limited to separately approved minimized diagnostics.

## Technical Context

**Language/Version**: PHP 8.3+ with Laravel API conventions already established by backend foundation, teacher workflow, student/reporting, teacher lifecycle, report lifecycle, and platform support slices  
**Primary Dependencies**: Laravel routing, middleware, Eloquent ORM, Form Requests, Policies, API Resources, Services, DTOs for questionnaire schema/response/grading/file inputs; existing tenant context resolver, authentication/account lifecycle behavior, role/permission authority, `School`, `User`, `StudentProfile`, `Questionnaire`, `LearningSet`, `TeacherAssignment`, report catalog/custom report definition behavior, private file storage, malware-scan status workflow, audit-event patterns, and response envelope components; repositories/query objects only for complex response aggregation, report catalog summary exposure, answer-file lookup, or audit-safe reads  
**Storage**: MySQL for questionnaires, advanced question schemas, response attempts, answers, file attachment metadata, grading outcomes, audit events, report catalog metadata, and relationships to school, student profile, learning set, academic period, and teacher assignment; private tenant-scoped file storage for student answer files; no public file storage or generated report packaging changes in this slice  
**Testing**: PHPUnit feature and unit tests in `schoolmaster-backend`; OpenAPI validation through Redocly from `schoolmaster-specs`; backend tests should run with `docker exec schoolmaster-backend-app-1 php artisan test` after implementation  
**Target Platform**: API-only Laravel backend consumed by future clients through the published `/api/v1` REST contract  
**Project Type**: Laravel API backend slice governed by the shared specification and OpenAPI repository  
**Performance Goals**: Contract validation maps 100% of new operations and schemas before backend implementation; tenant and authorization checks complete before protected data access for 100% of operations; file-response validation rejects unsupported categories, unsafe names, executables/archives, files over 25 MB, multiple files per question, type mismatches, and unsafe scan states; reports expose only assessment counts, completion status, grading status, and score summaries  
**Constraints**: OpenAPI expansion required before backend exposure; approved advanced question types are only `long_text` and `file_response`; long-text answers allow 1-10,000 characters and reject blank or whitespace-only answers; file responses allow PDF, image, text, and office files up to 25 MB with one file per question; file attachments stay unavailable until scan status is `clean`; clean answer-file downloads are limited to owning or assigned teachers and school administrators with same-school assessment review authority; failed-scan file-response answers may be graded only as zero or exempt; one submission attempt per student per assigned questionnaire; response submission closes at the learning-set due date; manual grading uses 0-100 points for advanced answers; students may see score/status and teacher feedback summary but not private grading notes; successful and denied answer-file downloads are audited; existing school-scoped, report, support, guardian, and teacher lifecycle behavior remains compatible; public identifiers are UUIDs; no undocumented endpoints, fields, filters, sort values, states, report fields, file access paths, or authorization exceptions  
**Scale/Scope**: Proposed boundary covers questionnaire schema expansion, student response submission, answer-file upload metadata, scan gating, teacher/admin response review, manual grading, student-safe response summaries, report-safe aggregate assessment fields, audit events, OpenAPI expansion, and backend regression coverage. No frontend implementation is included.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **OpenAPI impact**: PASS. The plan requires contract expansion before backend implementation exposes advanced question types, response submission, file upload, review, grading, report catalog, or assessment-summary behavior.
- **Repository impact**: PASS. `schoolmaster-specs` leads specification and OpenAPI work, `schoolmaster-backend` implements the backend slice, and `schoolmaster-frontend` has no implementation change in this feature.
- **Backend architecture**: PASS. The plan requires feature-organized controllers, Form Requests, Policies, API Resources, Services, DTOs for multi-field schema/response/grading/file inputs, UUID public identifiers, and repositories/query objects only where response aggregation, report catalog summary exposure, answer-file lookup, or audit-safe reads become complex.
- **Frontend architecture**: PASS. No frontend code changes are included; future frontend consumption must remain service-isolated and contract-based.
- **Tenancy and data boundaries**: PASS. School-owned records remain scoped by `school_id`; response attempts, answers, file attachments, grading outcomes, report summary fields, and audit events are tenant-owned; cross-tenant access is deny-by-default.
- **API compatibility and auth**: PASS. The plan is additive, preserves existing question types and school-scoped endpoints, requires explicit teacher/student/school-administrator authority, and documents tenant-safe validation, authorization, not-found, conflict, scan-pending, scan-failed, and unavailable-file responses before backend exposure.
- **Verification**: PASS. PHPUnit feature/unit coverage and Redocly contract validation are required before backend implementation merges. No frontend implementation is planned, so Vitest is deferred until frontend consumption begins.
- **Constitution deviations**: PASS. No deviations are introduced.

## Project Structure

### Documentation (this feature)

```text
specs/014-advanced-assessment-content/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── backend-advanced-assessment-content.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Created later by /speckit-tasks
```

### Source Code (target repositories)

```text
# Backend repository (Laravel API)
schoolmaster-backend/
├── app/
│   ├── DTOs/
│   │   └── Assessment/
│   ├── Http/
│   │   ├── Controllers/Api/V1/Assessment/
│   │   ├── Controllers/Api/V1/Student/
│   │   ├── Requests/Api/V1/Assessment/
│   │   └── Resources/Assessment/
│   ├── Models/
│   ├── Policies/
│   ├── Services/
│   │   └── Assessment/
│   └── Repositories/
├── database/
│   ├── factories/
│   ├── migrations/
│   └── seeders/
├── routes/
│   └── api.php
└── tests/
    ├── Feature/Assessment/
    └── Unit/Assessment/

# Frontend repository (Vue 3)
schoolmaster-frontend/
└── No implementation changes in this feature.

# Specification and contract repository
schoolmaster-specs/
├── api/openapi.yaml
├── specs/001-schoolmaster-platform/contracts/openapi.yaml
└── specs/014-advanced-assessment-content/
```

**Structure Decision**: Keep this as a backend-only advanced assessment API slice with specification artifacts under `specs/014-advanced-assessment-content/`. Backend implementation should follow existing Laravel API structure used by prior slices, with domain rules in services, request validation in Form Requests, authorization in Policies, API Resources for response shaping, DTOs for questionnaire schema/response/grading/file inputs, and repositories/query objects only for complex response aggregation, report catalog summary exposure, answer-file lookup, or audit-safe reads. Frontend work is explicitly deferred until contract-compliant backend behavior exists.

## Phase 0 Research Summary

Research is captured in [research.md](./research.md). Key decisions:

- OpenAPI must be expanded before backend implementation exposes any advanced assessment operation or schema.
- Approved advanced question types are limited to `long_text` and `file_response`.
- `long_text` answers allow 1-10,000 characters and reject blank or whitespace-only answers.
- `file_response` answers reuse teacher-content file categories: PDF, image, text, and office files up to 25 MB, one file per question.
- File-response attachments use private tenant storage and are unavailable until malware scan status is `clean`.
- Failed-scan file-response answers remain unavailable and may receive only zero or exempt grading outcomes.
- Students get one submission attempt per assigned questionnaire; no resubmission after submit.
- Response submission closes at the learning-set due date; no separate assessment window is introduced.
- `long_text` and `file_response` answers are manually graded on a 0-100 point scale; legacy auto-gradable behavior remains unchanged.
- Students see score/status and teacher feedback summary; private grading notes remain hidden from students, guardians, reports, platform, support, and audit payloads.
- Successful and denied answer-file download attempts are audited with tenant-safe metadata.
- Clean answer-file downloads are limited to owning or assigned teachers and school administrators with same-school assessment review authority.
- Reports expose only assessment counts, completion status, grading status, and score summaries; no answer text, uploaded files, file links, answer keys, private file metadata, or grading notes.
- Guardian advanced assessment visibility, frontend implementation, report output file packaging, public file URLs, permanent purge, legal hold, anonymization, messaging, notifications, and emergency support access are excluded.

## Phase 1 Design Summary

Design artifacts are captured in:

- [data-model.md](./data-model.md): advanced question schemas, response attempts, answers, file attachments, grading outcomes, report catalog boundaries, audit events, validation rules, state transitions, and tenant boundaries.
- [contracts/backend-advanced-assessment-content.md](./contracts/backend-advanced-assessment-content.md): proposed operation boundary, required OpenAPI expansion, request/response expectations, tenant and authorization rules, blocked behavior, and verification requirements.
- [quickstart.md](./quickstart.md): validation walkthrough and commands for contract and backend readiness.

Implementation must update `api/openapi.yaml` before routes, controllers, requests, services, or tests are added in the backend repository.

## Post-Design Constitution Check

- **OpenAPI impact**: PASS. Design artifacts require contract expansion before implementation and do not approve backend-local behavior outside OpenAPI.
- **Repository impact**: PASS. `schoolmaster-specs` remains the source of truth, `schoolmaster-backend` is the only implementation target for this slice, and frontend work remains deferred.
- **Backend architecture**: PASS. Design notes preserve Service Layer, Form Request, Policy, Resource, DTO, UUID, audit-event, tenant-scope, scan-gating, manual grading, report summary, and explicit authorization expectations.
- **Frontend architecture**: PASS. No frontend source change is planned.
- **Tenancy and data boundaries**: PASS. School-owned records remain scoped by `school_id`; response attempts, answers, files, grading outcomes, and report summary fields are exposed only through documented school-scoped operations.
- **API compatibility and auth**: PASS. The plan is additive, requires contract revision before exposure, preserves existing v1 question types and endpoint behavior, and separates student, teacher, school-administrator, report, platform, support, and guardian permissions.
- **Verification**: PASS. Quickstart defines PHPUnit and OpenAPI validation expectations for all affected critical flows.
- **Constitution deviations**: PASS. No deviations are introduced.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations are introduced by this plan |

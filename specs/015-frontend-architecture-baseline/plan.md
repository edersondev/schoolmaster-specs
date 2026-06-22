# Implementation Plan: Frontend Architecture Baseline

**Branch**: `015-frontend-architecture-baseline` | **Date**: 2026-06-21 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/015-frontend-architecture-baseline/spec.md`

## Summary

Define the durable frontend architecture baseline for the SchoolMaster Vue 3 SPA before concrete layout, authentication, or business module implementation begins. The plan establishes JavaScript-only Vue 3 Composition API conventions, Vue Router, Pinia, Axios service isolation, Element Plus, Element Plus Icons, Vue I18n, Tailwind CSS, clean frontend boundaries, reusable CRUD-oriented admin patterns, and API-first consumption of approved `/api/v1` OpenAPI contracts.

## Technical Context

**Language/Version**: JavaScript with Vue 3 SFCs using Composition API and `<script setup>`; TypeScript is intentionally out of scope for this baseline.  
**Primary Dependencies**: Vue 3, Vue Router, Pinia, Axios, Element Plus, `@element-plus/icons-vue`, Vue I18n, Tailwind CSS.  
**Storage**: N/A for this planning slice. Future session or tenant-context persistence may use only approved authenticated backend responses or approved persisted session metadata.  
**Testing**: Specification review in `schoolmaster-specs`; future frontend repository work should use Vitest for services, stores, composables, and route-level workflow logic where risk justifies coverage.  
**Target Platform**: `schoolmaster-frontend` Vue 3 SPA consuming contracts from `schoolmaster-specs`.
**Project Type**: Frontend architecture baseline and cross-repository specification artifact.  
**Performance Goals**: Establish reusable frontend boundaries that support responsive admin SaaS workflows as architecture guidance only. This slice does not define runtime performance benchmarks; future frontend implementation specs must define measurable performance targets when they approve concrete shell, dashboard, or business module behavior.
**Constraints**: No TypeScript requirement; no backend implementation; no OpenAPI changes; no concrete business module behavior; only approved OpenAPI-backed `/api/v1` contract semantics may be consumed; Element Plus tags use PascalCase; Tailwind handles layout and visual refinement; shared UI text is centralized through Vue I18n; reusable UI must target WCAG 2.1 AA; icons default to Element Plus Icons.  
**Scale/Scope**: Baseline applies to all future SchoolMaster frontend features, especially admin CRUD workflows, application shell composition, service/store/composable boundaries, JSDoc-based JavaScript contract definitions, UI primitives, localization readiness, and diagnostic boundaries.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. This slice introduces no API changes and requires future frontend work to consume only approved `/api/v1` OpenAPI contract behavior.
- PASS: Repository impacts are separated. `schoolmaster-specs` leads this baseline, `schoolmaster-frontend` consumes it later, and `schoolmaster-backend` remains unchanged.
- PASS: Backend architecture requirements are N/A because this feature does not change Laravel backend code, models, policies, services, requests, resources, or public identifiers.
- PASS: Frontend design uses Vue 3 Composition API with `<script setup>`, Pinia, Vue Router, Tailwind CSS, Axios service modules, feature-aligned organization, and service-isolated API access.
- PASS: Tenant and data impact are documented as no new tenant-owned data, no new cross-tenant access, and no new soft-delete behavior. Future tenant context must come from authenticated API responses or approved persisted session metadata.
- PASS: API compatibility and authentication/authorization impact are documented as unchanged. Client-side permission checks are visibility and navigation aids only.
- PASS: Verification is scoped to specification review now, with future Vitest coverage required for affected frontend services, stores, composables, and route-level workflow logic when implementation begins.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/015-frontend-architecture-baseline/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
│   └── frontend-architecture-contract.md
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (target repositories)

```text
# Specification repository
schoolmaster-specs/
├── docs/
│   ├── frontend-architecture.md
│   ├── frontend-admin-system-architecture.md
│   ├── frontend-guidelines.md
│   ├── frontend-feature-roadmap.md
│   └── naming-conventions.md
├── specs/
│   └── 015-frontend-architecture-baseline/
│       ├── spec.md
│       ├── plan.md
│       ├── research.md
│       ├── data-model.md
│       ├── quickstart.md
│       └── contracts/
│           └── frontend-architecture-contract.md
└── AGENTS.md

# Frontend repository target shape consumed later by schoolmaster-frontend
schoolmaster-frontend/
├── src/
│   ├── app/
│   ├── assets/
│   ├── components/
│   ├── composables/
│   ├── constants/
│   ├── contracts/
│   ├── layouts/
│   ├── pages/
│   ├── router/
│   ├── services/
│   ├── stores/
│   └── utils/
│
│   # Feature modules are organized consistently across these boundaries.
│   # Shared folders contain only cross-feature infrastructure and primitives.
└── tests/
```

**Structure Decision**: This planning slice changes only specification artifacts in `schoolmaster-specs`. It defines the target frontend structure that later work in `schoolmaster-frontend` must follow, but it does not implement frontend runtime files yet. Future frontend implementation must organize product code by feature module across the approved responsibility boundaries; shared layer folders are reserved for cross-feature infrastructure, reusable primitives, and framework boundaries. Backend source layout is intentionally omitted because this feature has no backend implementation impact.

## Phase 0: Research

Research output is captured in [research.md](research.md). All technical context items are resolved.

## Phase 1: Design and Contracts

Design outputs are captured in:

- [data-model.md](data-model.md)
- [contracts/frontend-architecture-contract.md](contracts/frontend-architecture-contract.md)
- [quickstart.md](quickstart.md)

## Post-Design Constitution Check

- PASS: The design artifacts preserve the no-OpenAPI-change boundary and require future features to link consumed operation IDs or block implementation until contracts are approved.
- PASS: The frontend contract defines service isolation, Pinia state boundaries, Vue Router route modules, Element Plus PascalCase usage, Element Plus Icons, Vue I18n text centralization, Tailwind layout responsibilities, JavaScript file conventions, and JSDoc typedef plus service mapping helper contract definitions.
- PASS: Data and tenancy impact remain documentation-only with no backend schema, tenant policy, or soft-delete changes.
- PASS: Future verification expectations are captured without creating implementation tasks in this slice.
- PASS: No complexity exception is introduced.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No constitution violations.

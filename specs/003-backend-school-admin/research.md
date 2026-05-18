# Research: Backend School Administration Foundation

## Decision: Implement the remaining P1 school administration slice before P2 workflows

**Rationale**: The platform foundation spec identifies tenant onboarding, school administration, academic structure, users, roles, permissions, and guardians as prerequisites for teacher workflows, student self-service, and reporting. After `002-backend-api-foundation`, the remaining P1 backend surface is users, roles, permission listing, academic years, academic periods, and guardians.

**Alternatives considered**:

- Start P2 teacher content next: rejected because teacher workflows depend on tenant users, roles, permissions, academic periods, and student/guardian foundations.
- Expand authentication and school management first: rejected for this slice because the user stated the 002 backend work is implemented and the next step should follow the same approach.

## Decision: Use the published OpenAPI operation IDs as the hard implementation boundary

**Rationale**: The aggregate contract already publishes list/create operations for users, roles, academic years, academic periods, guardians, and permission listing. Contract-first governance requires backend implementation to stop at these documented operations and avoid exposing local behavior not yet present in OpenAPI.

**Alternatives considered**:

- Implement common CRUD operations by convention: rejected because update, get, deactivate, delete, restore, invitation, and password-management flows are not fully documented for these resources.
- Treat the platform spec's broad "maintain" wording as permission for full CRUD: rejected because OpenAPI is the formal frontend/backend contract and must be expanded first.

## Decision: Resolve tenant context before module-specific work

**Rationale**: ADR 004 and multi-tenant guidance require `School` as the v1 tenant root and `school_id` as the concrete tenant column. School-scoped requests must reject missing, mismatched, inactive, or unauthorized tenant context before validation that depends on school-owned records, authorization decisions, services, persistence, or response shaping.

**Alternatives considered**:

- Let each service infer tenant scope from request fields: rejected because it increases cross-tenant leakage risk and duplicates authorization logic.
- Allow system administrators to bypass school scope: rejected because platform access is not an implicit bypass for school-scoped module actions.

## Decision: Keep role and permission scope separation explicit

**Rationale**: Security guidance says permissions are assigned through roles, not directly to users. Platform-scoped roles may grant platform capabilities only, and school-scoped roles may grant school capabilities only within the resolved school tenant. This prevents accidental privilege elevation and keeps authorization reviewable.

**Alternatives considered**:

- Allow mixed-scope roles for convenience: rejected because it blurs platform and school boundaries.
- Allow per-user permission overrides: rejected because direct per-user permission assignment is outside v1 scope.

## Decision: Treat academic period validation as same-school parent containment

**Rationale**: Academic periods are school-owned subdivisions of academic years. A valid period must reference an academic year in the same resolved school, fit within the parent academic year date range, and use a sequence unique within that academic year.

**Alternatives considered**:

- Allow periods outside the academic year with warnings: rejected because later grades, attendance, learning sets, and reports depend on coherent academic boundaries.
- Enforce global sequence uniqueness across all years: rejected because sequence has business meaning inside the academic year, not across the full school history.

## Decision: Make guardian-student associations atomic

**Rationale**: Guardian creation may include student profile references. To preserve tenant integrity, every referenced student profile must be active and belong to the same resolved school. If any reference fails, no partial guardian association should be created.

**Alternatives considered**:

- Create the guardian and skip invalid student references: rejected because it hides data-quality and cross-tenant authorization failures.
- Allow cross-tenant guardian links for shared family cases: rejected because no cross-school guardian model is specified for v1.

## Decision: Verify the slice with operation-level feature tests and contract validation

**Rationale**: The slice changes critical school administration flows. PHPUnit feature tests should cover success, validation, authorization, tenant isolation, inactive status, response shape, role/permission compatibility, academic date/sequence rules, and guardian association failures. Redocly validation ensures the consumed contracts remain valid.

**Alternatives considered**:

- Rely on unit tests only: rejected because tenant and response-envelope behavior is observable at the API boundary.
- Defer contract validation to frontend work: rejected because backend implementation must not drift from the published contract.

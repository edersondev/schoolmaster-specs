# Research: Backend Guardian Self-Service

## Decision: Expand OpenAPI before backend exposure

**Decision**: Add OpenAPI operations, schemas, response envelopes, field visibility rules, authorization notes, tenant behavior, and non-enumerating denial semantics before any backend route exposes guardian self-service behavior.

**Rationale**: Guardian self-service creates a new externally visible authorization surface. The existing contracts publish school-admin guardian management and student self-view behavior, but not guardian-facing profile, academic, or contact views.

**Alternatives considered**:
- Implement backend routes first and document later: rejected because it violates API-first governance.
- Reuse student self-view endpoints for guardians: rejected because guardian visibility is deliberately narrower than student self-view.
- Reuse school-admin guardian endpoints for guardians: rejected because guardian users must not receive school-admin powers or field visibility.

## Decision: Explicit school-admin guardian-user link proof

**Decision**: Guardian self-service requires an authenticated active user account explicitly linked by a school administrator to an active same-school guardian record.

**Rationale**: An explicit school-admin-created link is auditable and avoids risky identity matching from email or phone values alone. It also keeps account onboarding separate from guardian association proof.

**Alternatives considered**:
- Automatic contact matching: rejected because contact values may be shared, stale, duplicated, or controlled by someone else.
- Invitation completion creates the guardian-user link: rejected because account activation alone should not prove school-approved guardian authority.
- Guardian self-claiming: rejected for v1 because it needs verification, dispute, and approval workflows not yet specified.

## Decision: Active guardian-student association is required

**Decision**: Guardian access to a student requires an active school-approved guardian-student association in the resolved school in addition to the explicit guardian-user link.

**Rationale**: The authenticated guardian identity and the student relationship are separate proofs. Both must be active, same-school, and school-approved before any student information is returned.

**Alternatives considered**:
- Guardian record link alone grants access to all associated students including inactive associations: rejected because association status is the school-controlled visibility gate.
- Student profile guardian references alone grant access without a user link: rejected because a guardian record must be tied to the authenticated account.

## Decision: Read-only guardian self-service in v1

**Decision**: Guardian self-service is read-only for permitted student profile summaries, academic summaries, and limited contact views.

**Rationale**: The roadmap item is specifically about guardian view access. Writes introduce higher-risk workflows for identity proof, contact maintenance, consent, legal restrictions, and school review.

**Alternatives considered**:
- Guardian profile updates: rejected because school-owned contact data maintenance needs school approval and conflict handling.
- Guardian association requests: rejected because self-claiming and approval queues are outside v1.
- Student activity submission or teacher content download: rejected because that would duplicate or bypass student self-view and teacher content authorization.

## Decision: Academic summaries only

**Decision**: Guardian academic visibility is limited to current grade summary, attendance totals or status, and learning-set progress or status where OpenAPI documents each summary.

**Rationale**: Summary-only access gives guardians useful academic context without turning guardian self-service into a parallel student self-view surface. It also minimizes exposure of detailed academic rows, correction history, internal actor metadata, and private teacher content.

**Alternatives considered**:
- Detailed grade and attendance rows: rejected for privacy and duplication of student self-view.
- Detailed grades but summary attendance: rejected because it creates inconsistent visibility rules.
- Deferring academic information entirely: rejected because academic visibility is part of the roadmap purpose.

## Decision: Explicit same-school academic period is required

**Decision**: Guardian academic summary requests must include an explicit same-school academic period.

**Rationale**: Explicit period scoping avoids ambiguous current-period rules and aligns with existing academic-period validation patterns in student reporting and teacher workflows.

**Alternatives considered**:
- Default to current active period: rejected because schools may have overlapping, transitional, or differently selected active periods.
- Return current academic year across all periods: rejected because it broadens scope and response size without a clear v1 need.
- No period filter: rejected because it weakens validation and may expose more academic history than intended.

## Decision: Limited contact visibility

**Decision**: Guardian contact views expose only the authenticated guardian's own contact fields, school-approved relationship labels, and the student's primary school-approved contact details.

**Rationale**: Contact information is privacy-sensitive. This boundary gives guardians useful operational contact context while hiding other guardians, non-primary student contact details, restricted emergency handling details, and school-only notes.

**Alternatives considered**:
- Authenticated guardian contact fields only: rejected because the roadmap includes contact information and student primary contact details are useful.
- Emergency contact names: rejected because emergency handling can expose sensitive relationship or custody information.
- All contact views deferred: rejected because it removes an approved part of guardian self-service scope.

## Decision: Uniform target-specific not-found response

**Decision**: Target-specific requests for missing, unassociated, inactive, or cross-tenant students return the same not-found envelope.

**Rationale**: A uniform not-found response prevents protected student enumeration. It keeps guardian APIs from revealing whether a student exists outside the guardian's permitted view.

**Alternatives considered**:
- Use forbidden for unassociated or inactive students: rejected because it confirms target existence or relationship state.
- Use forbidden for all denied target-specific requests: rejected because it still distinguishes protected targets from missing records after authentication.
- Split cross-tenant and inactive cases: rejected because it makes denial semantics harder to test and easier to leak.

## Decision: Use existing Laravel service and policy boundaries

**Decision**: Implement guardian self-service rules through Laravel services, Form Requests, Policies, API Resources, DTOs, and focused repositories/query objects where needed.

**Rationale**: The workflows include tenant resolution, guardian-user link proof, association checks, summary aggregation, field-level visibility, non-enumerating denial, and audit writes that should not live in controllers.

**Alternatives considered**:
- Controller-local business logic: rejected by constitution and maintainability requirements.
- Broad repositories for all guardian access: rejected unless query complexity justifies them.

## Decision: Tenant-safe audit for reads and denials

**Decision**: Record tenant-safe audit events for guardian self-service access grants, denied attempts, blocked cross-tenant attempts, and visibility-sensitive reads.

**Rationale**: Guardian self-service exposes sensitive student and contact information. Audit events support investigation and compliance review without exposing private payloads, school-only notes, credentials, file paths, or unauthorized cross-tenant details.

**Alternatives considered**:
- Audit only denied attempts: rejected because successful access to sensitive student data also needs traceability.
- Audit only writes: rejected because this slice is read-only and still privacy-sensitive.
- Full payload audit: rejected because it would store sensitive student/contact data unnecessarily.

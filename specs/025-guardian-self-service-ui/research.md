# Research: Guardian Self-Service UI

## Decision: Consume only approved guardian self-service OpenAPI contracts

**Rationale**: The aggregate contract already includes guardian student list,
guardian student detail, guardian academic summary, and guardian contact view
operations. This UI slice can deliver the roadmap item without new backend
behavior.

**Alternatives considered**:

- Add frontend-only routes to inferred backend behavior: rejected because
  OpenAPI is the contract source of truth.
- Reuse student self-service routes for guardian views: rejected because
  guardian visibility is narrower and has different association proof.
- Reuse school-admin guardian routes for guardian self-service: rejected
  because guardian users must not receive admin authority or admin field
  visibility.

## Decision: Keep guardian workspace separate from student self-service

**Rationale**: Guardian access requires guardian-user link proof, active
guardian-student association, and narrower field visibility. A distinct
workspace, route family, service family, and UI text prevents guardians from
appearing to have student-only powers.

**Alternatives considered**:

- Add guardian mode inside student self-service screens: rejected because it
  risks mixing actor authority, data visibility, and route guards.
- Reuse student components without guardian-specific contracts: rejected
  because student details, downloads, questionnaire behavior, grades, and
  attendance records differ from guardian summary-only behavior.

## Decision: Use current active academic period only

**Rationale**: The clarified scope requires the UI to use the current active
same-school academic period when calling guardian academic summary. This avoids
inventing a period picker, arbitrary query parameter, or alternate period
source not approved for guardians.

**Alternatives considered**:

- Manual period picker: rejected until a guardian-approved period source and UX
  contract exists.
- Free URL/query `academic_period_id`: rejected because it encourages
  undocumented period selection and extra tenant-validation edge cases.
- Hide academic summary entirely: rejected because academic summary is an
  approved guardian self-service view when current active period context
  exists.

## Decision: Treat target-specific denial as safe not-found

**Rationale**: Backend guardian self-service requires uniform not-found for
missing, unassociated, inactive, or cross-tenant target students. The frontend
must preserve that non-enumerating behavior instead of translating some cases
into more revealing messages.

**Alternatives considered**:

- Show forbidden for unassociated or inactive students: rejected because it
  can confirm target existence or relationship state.
- Show different copy for cross-tenant targets: rejected because it leaks
  tenant boundary details.
- Preserve raw backend denial details: rejected because UI state and
  diagnostics must remain tenant-safe.

## Decision: Model guardian-specific empty and missing-context states

**Rationale**: Guardian workflows need states distinct from generic denial:
no-active-school, no-guardian-link, no-linked-students, no-academic-period,
unavailable-summary, not-found, and stale-response. Clear states improve
testing and prevent false empty states from masking authorization or context
problems.

**Alternatives considered**:

- Use one generic empty state: rejected because no linked students differs from
  missing school, safely identified missing link, no academic period, and
  denied target.
- Treat missing guardian link as forbidden only: rejected where approved
  session or access behavior can safely identify the missing link, but the UI
  must not infer no-guardian-link from generic denials.
- Treat unavailable academic summary as not-found: rejected because a
  successful response with no summary values is not a target-denial event.

## Decision: Use route-local composables and minimal Pinia coordination

**Rationale**: Guardian screens need stale-response protection and selected
student coordination without persisting sensitive student, contact, or guardian
records globally. Route-local composables keep state minimal, with Pinia used
only for shared session, active school, shell, and navigation state.

**Alternatives considered**:

- Store all guardian student/contact/academic data in Pinia: rejected because
  it persists sensitive guardian-visible data across routes more than needed.
- Direct service calls in route components: rejected by service isolation and
  component-boundary rules.
- Put business visibility rules in components: rejected because visibility
  rules must come from specs and approved backend contracts.

## Decision: Verify safe diagnostics as part of feature acceptance

**Rationale**: Guardian views expose student identity, academic summaries,
contact values, relationship labels, and tenant-sensitive denials. Diagnostics
may include safe operation IDs and generic state kinds, but must not persist
unassociated identifiers, other guardian data, non-primary contacts, school-only
notes, correction details, teacher-private data, report data, token values, role
internals, or cross-tenant details.

**Alternatives considered**:

- Rely on existing logging conventions only: rejected because this feature
  introduces guardian-specific student and contact privacy risk.
- Check only denied states: rejected because successful and empty states also
  carry guardian-visible student context.

## Decision: Keep Vue component boundaries explicit

**Rationale**: Vue 3 Composition API, `<script setup>`, props-down/events-up,
service-isolated HTTP access, focused components, and composables are the
approved frontend baseline. Guardian route views should compose smaller
components instead of owning service calls, mapping logic, and multiple UI
sections directly.

**Alternatives considered**:

- One large route component for the whole guardian workspace: rejected because
  it mixes orchestration, list, detail, academic, contact, and feedback
  responsibilities.
- Direct Axios in components: rejected by frontend architecture rules.
- Add TypeScript-only contracts: rejected because the current frontend baseline
  uses JavaScript with JSDoc typedefs and mapping helpers.

# Feature Specification: School Context Selection UI

**Feature Branch**: `033-activate-school-ui`
**Created**: 2026-07-29
**Status**: Draft
**Input**: User description: "I want to activate a school through UI, check the School-selection flow."

## Clarifications

### Session 2026-07-29

- Q: Where should school selection be presented? → A: Use a dedicated
  selection page when school context is required but missing, plus a persistent
  authenticated-shell control for later switching.
- Q: After a school is confirmed, where should the administrator go? → A:
  Resume the requested route when still valid; shell switches keep a compatible
  current route; otherwise open the school dashboard.
- Q: How long should the last-confirmed school preference remain available? →
  A: Retain it across reloads while the same session identity remains valid;
  clear it on logout, expiry, token rejection, or identity change.
- Q: Which fields should identify each school choice? → A: Show school name,
  INEP code, city, and state.
- Q: How should the frontend detect that the current school became invalid? →
  A: Invalidate context after an authoritative tenant-mismatch or inactive
  response, bootstrap failure, or successful local lifecycle action; do not
  poll.
- Q: When may the persistent shell control initiate a school switch? → A: US1
  presents confirmed context read-only; US2 enables selector navigation only
  after prior-tenant cleanup, stale-response rejection, and pending-work guards
  are active.
- Q: Which identifiers may appear in a selector choice? → A: INEP is the only
  approved identifier; do not expose CNPJ or any other identifier.
- Q: What makes a requested route "released" after confirmation? → A: It is a
  registered named route that the existing release/feature-gate mechanism
  reports enabled at the post-confirmation check; missing or disabled routes
  fall back to the school dashboard.
- Q: Which response shape may authoritative invalidation inspect? → A: A single
  frontend normalization boundary reads only canonical
  `response.data.error.code` or documented legacy `response.data.code`, returns
  a lowercase code or `null`, and leaves all other shapes to local error handling.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Select an Active School (Priority: P1)

A signed-in System Administrator can open a school selector from the dedicated
selection page when required context is missing, find an active school, and
select it as the current school context before entering school-owned
workspaces. The authenticated shell presents the confirmed school without
enabling deliberate switching until User Story 2 protections are available.

**Why this priority**: System Administrator master access cannot be used for
school-owned work until the operator has a visible, safe way to establish the
required active school context.

**Independent Test**: Sign in as a System Administrator without a resolved
school, open the selector, search a set containing at least 100 schools, choose
one active school, and verify that the confirmed school is shown as current
before its school-owned workspace loads.

**Acceptance Scenarios**:

1. **Given** a signed-in System Administrator has no resolved school, **When**
   they open the school selector, **Then** they can search and choose only
   active schools available through their platform access.
2. **Given** multiple active schools are available, **When** the administrator
   chooses one school, **Then** the application confirms that exact school as
   the current context before showing school-owned content.
3. **Given** the chosen school has not yet been confirmed, **When** selection
   is pending, **Then** school-owned content remains blocked and the selector
   communicates progress without accepting duplicate selection submissions.
4. **Given** no school was previously confirmed, **When** the selector opens,
   **Then** the application does not automatically choose the first visible
   school.
5. **Given** schools share the same or similar names, **When** choices are
   displayed, **Then** each option shows school name, INEP code, city, and
   state so the administrator can distinguish the intended school without
   exposing CNPJ.
6. **Given** a System Administrator opens a school-owned route without active
   school context, **When** route requirements are evaluated, **Then** a
   dedicated selection page blocks the route until context is confirmed.
7. **Given** a System Administrator already has confirmed school context,
   **When** they use the authenticated application shell, **Then** a persistent
   read-only indicator shows that exact school without initiating a switch.
8. **Given** school selection was required while opening a school-owned route,
   **When** the selected context is confirmed and the requested route remains
   released and authorized, **Then** the administrator resumes that route;
   otherwise they enter the school dashboard.

---

### User Story 2 - Restore or Switch School Context (Priority: P2)

A returning System Administrator can restore the last confirmed active school
when it remains valid, or deliberately switch to another active school without
seeing data retained from the previous school.

**Why this priority**: Frequent school switching must remain efficient while
preserving strict tenant isolation and preventing stale cross-school content.

**Independent Test**: Confirm School A, load school-owned content, switch to
School B, and verify School A data clears before School B data appears. Repeat
after reload with a valid and then invalid last-confirmed school.

**Acceptance Scenarios**:

1. **Given** the administrator has a last-confirmed school that remains active
   and available, **When** the authenticated session is restored, **Then** that
   school becomes current only after fresh confirmation.
2. **Given** School A is current, **When** the administrator selects School B,
   **Then** visible and shared School A data is cleared before School B data is
   presented.
3. **Given** a prior selection request completes after a newer selection,
   **When** its result arrives, **Then** it cannot replace the newer current
   school or repopulate prior-school data.
4. **Given** switching school would discard unsaved work or an open lifecycle
   confirmation, **When** the administrator uses the persistent shell control
   to request the switch, **Then** they must confirm or cancel the discard
   before selector navigation or context changes.
5. **Given** confirmation of a selected school fails, **When** the failure is
   shown, **Then** the proposed school is not made current, prior school-owned
   data remains cleared, and the administrator can retry or choose another
   active school.
6. **Given** the administrator is on a platform-wide screen, **When** they
   change the current school, **Then** the selected-school indicator updates
   without incorrectly filtering the platform-wide screen.
7. **Given** the administrator switches schools from the shell, **When** the
   current route is a generic school-owned workspace or list valid for the new
   school, **Then** that route remains open and reloads in the new context.
8. **Given** the administrator switches schools from a school-owned detail,
   edit, action, or subject-bound route, **When** the prior route identifiers
   are not safe for reuse, **Then** navigation falls back to the school
   dashboard instead of carrying those identifiers into the new context.
9. **Given** the same authenticated session identity remains valid across an
   application reload, **When** restoration begins, **Then** the last-confirmed
   school may be requested for fresh confirmation.
10. **Given** the administrator logs out or the session expires, is rejected,
    or changes identity, **When** session state is cleared, **Then** the stored
    school preference is also cleared and cannot affect a later identity.

---

### User Story 3 - Activate Then Select a School (Priority: P3)

A System Administrator who needs an inactive school can use the existing
school lifecycle UI to activate it, then return to the selector and establish
it as the current school context. Lifecycle activation and context selection
remain visibly distinct actions.

**Why this priority**: An inactive tenant cannot safely become operational
merely through selection, but administrators still need a complete UI path
from lifecycle activation to school-owned work.

**Independent Test**: Locate an inactive school in School administration,
complete the existing activation confirmation, refresh or reopen the selector,
select the now-active school, and verify that school-owned loading begins only
after context confirmation.

**Acceptance Scenarios**:

1. **Given** a school is inactive, **When** the administrator opens the school
   selector, **Then** that school is not offered as a selectable context.
2. **Given** an inactive school is needed for operational work, **When** the
   administrator follows the School administration path, **Then** activation
   uses the existing lifecycle confirmation, reason, effective-date,
   authorization, conflict, and audit rules.
3. **Given** lifecycle activation succeeds, **When** the selector is refreshed
   or reopened, **Then** the school becomes eligible to appear and can be
   selected through the normal confirmation flow.
4. **Given** lifecycle activation fails or conflicts, **When** the result is
   shown, **Then** the school remains unavailable for selection and no school
   context is inferred from the attempted activation.
5. **Given** the currently selected school is deactivated, deleted, or becomes
   unavailable, **When** that state is confirmed, **Then** current school
   context and school-owned data are cleared and another active school must be
   selected before school-owned work resumes.

### Edge Cases

- The last-confirmed school becomes inactive, deleted, or unauthorized between
  sessions; restoration fails safely and the selector requires a new choice.
- A school is active when search results load but becomes inactive before
  selection confirmation; it is not committed as current context.
- No active schools are available; the selector shows a clear empty state and,
  when authorized, a path to School administration without fabricating a
  tenant context.
- Search, pagination, or selection results arrive out of order; stale results
  do not replace newer search results or confirmed context.
- Session, role, or permission state changes while the selector is open; the
  selector closes or returns to a safe state before protected content loads.
- A browser is shared by multiple administrators; logout, expiry, token
  rejection, or identity change clears the prior administrator's school
  preference before another identity can restore context.
- Two schools have the same display name; approved secondary identity details
  let the administrator distinguish them without exposing sensitive data.
- Lifecycle activation succeeds while a cached active-school search is still
  visible; the option appears only after a deliberate refresh or new search.
- A school switch is cancelled because of unsaved work; the current confirmed
  school and its data remain unchanged.
- The current school is deactivated, deleted, or made unavailable outside the
  current browser; the next authoritative tenant-mismatch or inactive response
  clears context and school-owned data without waiting for polling.
- A non-System Administrator with one resolved school sees the existing current
  school presentation, not the platform-wide selector.
- A non-System Administrator who may need multi-school selection remains in the
  existing safe blocked state until an approved user-authorized school source
  is documented for that actor type.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Correct current-session contract conformance so
  the tenant context already resolved from `X-School-Id` is serialized as the
  authenticated session's `resolved_school`. Existing school listing, school
  lifecycle, authorization, validation, conflict, and audit behavior otherwise
  remain authoritative; no new endpoint or persistence is required.
- **Frontend repository impact**: Add the dedicated missing-context selection
  page and authenticated-shell school control, active-school search and empty
  states, selection confirmation orchestration, current-school presentation,
  last-confirmed restoration, lifecycle-to-selector refresh behavior,
  active-context invalidation, and focused regression tests.
- **Specification or contract repository impact**: This specification approves
  `listSchools` as the selector source only for System Administrator, records
  the boundary between lifecycle activation and context selection, and makes
  the existing nullable `AuthSession.resolved_school` property required so
  successful and unresolved current-session responses are unambiguous.
- **Delivery ownership and sequencing**: This specification and OpenAPI
  tightening lead. The backend repository then corrects the verified
  current-session serialization gap and adds regression coverage. The frontend
  repository implements the selector only after that contract behavior is
  available.

### API Contract Impact

- **OpenAPI update required**: Yes, contract tightening only. The existing
  nullable `resolved_school` property becomes required in `AuthSession`; no
  route, field, status, envelope, or version is added.
- **Versioned endpoints affected**: Existing `GET /api/v1/schools` supplies
  active searchable choices, `GET /api/v1/auth/me` confirms selected context
  through the documented school-context header, and
  `POST /api/v1/schools/{schoolId}/activate` remains the existing lifecycle
  activation path.
- **JSON response impact**: No new fields or envelopes are required. School
  choices use the existing paginated School collection, confirmed context uses
  the existing always-present nullable authenticated-session `resolved_school`,
  and lifecycle actions use the existing lifecycle outcome.
- **Authentication/authorization impact**: The selector is approved only for
  a resolved active platform-scoped `System Administrator`, which may access
  every active school under the completed master-access contract. Selection
  does not bypass authentication, active-school validation, tenant resolution,
  identity ownership, lifecycle, or business prerequisites.
- **Compatibility impact**: Additive frontend behavior for System
  Administrator. Existing single-school restoration and non-System
  Administrator behavior remain unchanged; no endpoint version change is
  required.

### Data & Tenancy Impact

- **Tenant scoping impact**: One backend-confirmed active school is the current
  tenant context for school-owned work. School-owned data is cleared before a
  context switch and remains limited to the confirmed school.
- **Cross-tenant or platform access impact**: System Administrator may discover
  active tenant roots for selection through the existing platform-wide school
  list. Selecting one school does not turn platform-wide screens into
  school-owned screens or permit combined cross-school school-owned output.
- **Soft delete impact**: No soft-delete behavior changes. Deleted schools are
  not valid choices; deactivation or deletion of the current school invalidates
  its selected context.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The application MUST expose the interactive school selector only
  after the authenticated session confirms an active platform-scoped role
  named exactly `System Administrator`.
- **FR-001a**: The application MUST use a dedicated selection page when a
  school-owned route requires school context and none is confirmed, and MUST
  retain persistent authenticated-shell current-school presentation. The shell
  switch action MUST be enabled only with the prior-context cleanup and
  pending-work protections required by FR-011 and FR-019.
- **FR-002**: The selector MUST source System Administrator choices only from
  the approved school list behavior and MUST request only active schools.
- **FR-003**: The selector MUST support server-confirmed search by school name
  and INEP code and MUST support navigating result pages without treating a
  partial page as the complete set of choices.
- **FR-004**: Each choice MUST display school name, INEP code, city, and state,
  and MUST NOT display CNPJ or any identifier other than the approved INEP code
  in the choice summary.
- **FR-005**: The application MUST NOT automatically select the first listed
  school when no last-confirmed valid context exists.
- **FR-006**: Choosing a school MUST request authenticated confirmation of that
  exact school before committing it as current context.
- **FR-007**: A selection MUST be committed only when the confirmed resolved
  school identifier matches the chosen school and the confirmed school is
  active.
- **FR-008**: School-owned routes and content MUST remain blocked while school
  selection or restoration is unresolved.
- **FR-009**: The application MUST store a last-confirmed school preference only
  after successful authenticated confirmation, MUST retain it only while the
  same session identity remains valid, and MUST treat it as a restoration
  request rather than proof of authorization.
- **FR-010**: Returning sessions MUST restore the last-confirmed school only
  after fresh authenticated confirmation that it remains active and available.
- **FR-010a**: Explicit logout, session expiry, token rejection, or authenticated
  identity change MUST clear the last-confirmed school preference before a
  later identity can request context restoration.
- **FR-011**: Before switching from one confirmed school to another, the
  application MUST invalidate visible and shared data associated with the prior
  school and MUST prevent stale prior-context responses from repopulating it.
- **FR-012**: If selection confirmation fails, the application MUST NOT commit
  the proposed school, MUST keep school-owned content blocked, and MUST offer a
  retry or new-selection path using contract-safe feedback.
- **FR-012a**: After successful selection from the dedicated selection page,
  the application MUST resume the requested route only when it remains
  registered and enabled by the existing release/feature-gate mechanism,
  authorized, and compatible with the confirmed school context; otherwise it
  MUST open the school dashboard.
- **FR-012b**: After a shell context switch, the application MUST retain a
  platform-wide route or a generic school-owned workspace/list route that can
  safely reload under the new school, and MUST open the school dashboard rather
  than reuse prior-school detail, edit, action, or subject identifiers.
- **FR-013**: Platform-wide screens MUST retain their published platform scope
  even when a current school is selected.
- **FR-014**: Inactive, soft-deleted, unavailable, or unconfirmed schools MUST
  NOT be selectable as current school context.
- **FR-015**: Lifecycle activation MUST remain a separate School administration
  action and MUST continue to require the existing confirmation, effective
  date, reason, authorization, conflict, and audit behavior.
- **FR-016**: After a school is successfully activated, the selector MUST allow
  a deliberate refresh or new search that can return the newly active school;
  activation alone MUST NOT select it automatically.
- **FR-017**: When the current school is confirmed inactive, deleted, or no
  longer available through bootstrap, a successful local lifecycle action, or
  an authoritative tenant-mismatch or inactive response, the application MUST
  clear its current context and school-owned data before requiring another
  selection and MUST NOT poll for lifecycle changes.
  Authoritative response classification MUST consume only the lowercase code
  or `null` returned by the documented frontend error normalization boundary;
  it MUST NOT infer context invalidity from another raw response shape.
- **FR-018**: Selector labels and feedback MUST distinguish "select school" as
  choosing tenant context from "activate school" as changing lifecycle status.
- **FR-019**: School switching MUST honor existing unsaved-change and open
  lifecycle-confirmation guards before discarding pending work.
- **FR-020**: The application MUST NOT use `listSchools` as a selection source
  for non-System Administrator actors until a specification confirms that the
  operation returns only schools authorized for each intended actor type.
- **FR-021**: Selector and context feedback MUST cover loading, no results, no
  active schools, validation, unauthorized, forbidden, inactive school,
  tenant mismatch, expired session, conflict, and temporary unavailable states
  without exposing hidden school or permission details. "No results" MUST mean
  the current filters matched no active school; "no active schools" MUST mean
  an unfiltered active-school request returned an empty collection.
- **FR-022**: The selector, current-school control, confirmation, empty states,
  and feedback MUST be keyboard operable, visibly focused, programmatically
  labeled, screen-reader announced, and usable at 390px, 768px, and 1440px
  viewport widths.
- **FR-023**: This feature MUST consume only the published school list,
  authenticated current-session, and existing school lifecycle contracts and
  MUST NOT add undocumented routes, fields, statuses, or client-side tenant
  authorization.
- **FR-024**: Frontend verification MUST cover role gating, active-only choices,
  search, pagination, duplicate-name distinction, explicit selection,
  restoration, switch cleanup, stale-response rejection, selection failures,
  platform-wide scope, lifecycle activation integration, current-school
  invalidation, non-System Administrator blocking, responsive behavior, and
  accessibility.

### Key Entities *(include if feature involves data)*

- **School**: Platform-managed tenant root with identity and lifecycle status;
  only an active, available school may become current context.
- **School Selection Choice**: Search result representing an active school the
  System Administrator may request as context, identified in the selector by
  school name, INEP code, city, and state; it is not authorization proof until
  authenticated confirmation succeeds.
- **Active School Context**: One confirmed school governing all subsequent
  school-owned navigation and data until cleared or replaced.
- **Last-Confirmed School Preference**: Non-authoritative identifier used to
  request context restoration across reloads while the same authenticated
  session identity remains valid; it is cleared when that identity is lost or
  replaced.
- **School Lifecycle State**: Active, inactive, or deleted availability that
  determines whether a school may appear in and be confirmed through the
  selector.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 9 of 10 representative System Administrators can find
  and select an intended active school from a set of at least 100 schools in
  under 30 seconds without facilitator help under the usability protocol below.
- **SC-002**: In 100% of tested school switches, no visible or shared data from
  the prior school appears after the new school is confirmed.
- **SC-003**: In 100% of tested inactive, deleted, unavailable, mismatched, and
  stale-result cases, an invalid school is not committed as current context and
  school-owned content remains blocked.
- **SC-004**: At least 95 of 100 selection attempts and 95 of 100 restoration
  attempts MUST show a confirmed school or a recoverable safe state within 2
  seconds under the performance protocol below.
- **SC-005**: At least 9 of 10 representative System Administrators can
  correctly distinguish school selection from lifecycle activation and
  complete the activate-then-select journey in under 2 minutes without
  facilitator help under the usability protocol below.
- **SC-006**: In all tested flows, no school-owned content appears before active
  context confirmation.
- **SC-007**: In all authorization tests, non-System Administrator actors gain
  no new school choices or tenant access from this feature.
- **SC-008**: All critical selector, switch, empty, and failure flows pass
  keyboard and screen-reader review at the three supported viewport widths.

### Verification Protocols

- **Performance protocol for SC-004**: Use the production frontend build in
  Chromium against deterministic API fixtures containing at least 100 active
  schools. Apply a 300 ms response delay to each school-list and current-session
  response with no injected server failure. Measure 100 explicit selections
  from selection submission and 100 restorations from the authenticated
  bootstrap request. Stop timing when exact context is confirmed or when a
  recoverable state renders with school-owned content blocked and a retry or
  new-selection action. Selection and restoration are scored separately.
- **Usability protocol for SC-001 and SC-005**: Use one cohort of at least 10
  intended System Administrator users or approved role-representative proxies
  who did not implement the feature and receive no feature-specific coaching.
  Give each participant only the task goal. Use the same seeded dataset with at
  least 100 active schools and one inactive activation target. Record completion
  time, assistance, outcome, and whether the participant correctly states that
  activation changes lifecycle status while selection chooses tenant context.

## Assumptions

- Initial selector scope is System Administrator only. Its existing master
  access authorizes discovery and selection of any active school.
- Existing `listSchools` authorization plus the active status filter is an
  approved school-choice source for System Administrator, but not automatically
  for other actor types.
- Existing `getCurrentUser` school-context behavior is the authoritative
  confirmation step. Planning verification found that backend serialization
  currently discards the resolved platform context, so contract-conformance
  correction is a delivery prerequisite.
- Feature 020 already provides School lifecycle activation and deactivation UI;
  this feature integrates selection with that completed flow instead of
  duplicating lifecycle controls.
- Feature 032 already provides master-access recognition and safe
  school-context switching foundations; this feature adds the user-facing
  selector and closes the intentionally blocked selection-source gap.
- Last-confirmed school persistence follows existing approved session metadata
  practices, survives reloads only while the same session identity remains
  valid, and contains no authority beyond requesting fresh confirmation.
- Active school selection does not change platform-wide page scope and does not
  create identity-owned student or guardian subject context.
- No new backend endpoint, persistence, or OpenAPI field is expected; the
  verified current-session conformance gap and existing-property requirement
  are corrected before frontend delivery.

# Research: Administration Foundation UI

## Decision: Use URL query as list-state source of truth

**Rationale**: Approved filters, sort, page, and page size must survive refresh,
browser back/forward, and return from create routes. A shared query composable
normalizes route values, removes unsupported keys, and triggers list reloads.

**Alternatives considered**: Page-local state was rejected because refresh and
back navigation lose context. Pinia persistence was rejected because it
duplicates router state and risks leaking tenant-specific list state.

## Decision: Use dedicated create routes

**Rationale**: Six create workflows have distinct fields, validation, remote
lookups, responsive needs, and unsaved-change rules. Dedicated routes provide
stable navigation, accessible page structure, refresh behavior, and consistent
route-leave guards.

**Alternatives considered**: Dialogs and drawers were rejected because they
complicate deep linking, mobile layouts, validation summaries, and browser
navigation. A resource-specific mix was rejected because it weakens shared
behavior.

## Decision: Guard every create-route exit with unsaved changes

**Rationale**: Vue Router route-leave guards can block browser and in-app
navigation. One composable compares current form values with the initial model,
suppresses prompts after successful submit, and uses the centralized
confirmation UI.

**Alternatives considered**: Guarding only school switches was rejected because
sidebar, browser-back, and list-return navigation could still discard data.
Automatic draft persistence was rejected because no storage/privacy behavior is
approved.

## Decision: Keep list and form orchestration in composables, not resource stores

**Rationale**: List state is represented in URL query and form state belongs to
one create route. `useAdminList`, `useAdminCreateForm`, and focused lookup/guard
composables provide predictable state without creating global state. Existing
Pinia session and shell stores supply permissions and active school context.

**Alternatives considered**: One Pinia store per resource was rejected because
state is not shared across unrelated routes and would duplicate query/form
state. Putting orchestration in route pages was rejected because seven modules
would repeat cancellation, errors, pagination, and tenant-reset behavior.

## Decision: Use shared CRUD presentation with resource-specific composition

**Rationale**: Shared components own page frame, filters, remote table,
pagination, feedback, and form shell. Resource components own fields, columns,
and options. Props flow down; events flow up. Route pages remain composition
surfaces.

**Alternatives considered**: One schema-driven mega-component was rejected
because resource validation and lookup behavior would become opaque. Fully
separate pages were rejected because they duplicate common accessibility and
state behavior.

## Decision: Map and validate OpenAPI data in service modules

**Rationale**: Each service accepts only approved request parameters, adds
tenant context through the shared API client, maps snake_case envelopes to
documented frontend contracts, and returns normalized errors. Components never
call Axios or inspect transport responses.

**Alternatives considered**: Direct API use in components or composables was
rejected by constitution and architecture. A single generic CRUD service was
rejected because operation-specific parameters and request fields need explicit
review.

## Decision: Cancel or ignore stale requests

**Rationale**: Query, page, route, permission, and school changes can overlap.
Each list/lookup load gets cancellation or request-generation protection. Only
the latest request for the current route and tenant may update visible state.

**Alternatives considered**: Last-response-wins behavior was rejected because a
slow old-school or old-filter response could overwrite current data.

## Decision: Use remote student-profile lookup for guardian associations

**Rationale**: `listStudentProfiles` publishes school context, pagination,
status, search, and sort parameters. Guardian form uses an Element Plus remote
multi-select backed by this operation, always requests `status=active`, and
submits UUIDs only.

**Alternatives considered**: Manual UUID entry was rejected as unsafe and
unusable. Loading every student was rejected for scale. Adding student CRUD was
rejected as outside this feature.

## Decision: Use server-driven sorting and pagination only where approved

**Rationale**: Element Plus tables emit sort intent, but service calls send it
only for operations publishing `sort`. Pagination always follows returned
`page`, `per_page`, and `total`. No client-side sort suggests unsupported
backend semantics. School sort remains hidden because the current backend
implementation ignores the published sort parameter and always orders by name.

**Alternatives considered**: Sorting only the visible page was rejected because
it misrepresents collection order. Generic sort controls on every resource were
rejected because contracts differ.

## Decision: Bind frontend surfaces to implemented permission codes

**Rationale**: Backend permission seeds and services define separate view and
manage permissions. Lookup-dependent create routes require both mutation
permission and lookup read permission. Route metadata, navigation, actions, and
tests use the exact matrix documented in the spec and frontend contract.
Guardian student association is optional, so `student_profiles.view` gates only
that lookup control; guardian creation remains available with guardian view and
manage permissions.

**Alternatives considered**: Generic role-name checks were rejected because
permissions, not role names, are the approved session capability contract.
Single-permission create routes were rejected where required lookup operations
have separate authorization.

## Decision: Limit role creation to school scope

**Rationale**: This story is an active-school administration workflow. The
frontend omits scope selection and always maps the required API field to
`scope=school`, while selecting only school-scope permissions. Platform-role
creation requires a separate platform workflow and is not approved here.

**Alternatives considered**: Exposing both scopes in one form was rejected
because it conflicts with the story actor/context and creates ambiguous
permission and tenant behavior.

## Decision: Normalize errors into shared administration feedback

**Rationale**: Service errors map to validation, unauthorized, forbidden,
tenant-mismatch, inactive-context, not-found, conflict/unavailable, and unknown
states. Field errors remain keyed to contract fields. Messages use Vue I18n and
must not disclose cross-tenant existence.

**Alternatives considered**: Per-page transport parsing was rejected because it
creates inconsistent privacy and recovery behavior. Raw backend messages were
rejected because they can expose internals and are not reusable UI contracts.

## Decision: Preserve observability without sensitive payload logging

**Rationale**: Failed actions should expose operation ID, normalized error code,
route name, and request/correlation identifier when available to existing
diagnostic hooks. Form values, email addresses, tenant data, bearer tokens, and
permission payloads are excluded.

**Alternatives considered**: No diagnostic signal was rejected because contract
mapping failures become hard to trace. Logging full requests was rejected for
privacy and security.

## Decision: Follow JavaScript project baseline despite TypeScript-preferring guidance

**Rationale**: Project architecture explicitly requires JavaScript and JSDoc
contracts. Vue Composition API, focused SFC boundaries, explicit props/emits,
computed derivation, and composable isolation still apply.

**Alternatives considered**: Introducing TypeScript was rejected because it
would violate the approved frontend baseline and expand feature scope.

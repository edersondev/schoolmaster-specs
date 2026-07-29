# Quickstart: System Administrator Frontend Access

## Preconditions

- Backend feature `031-system-admin-master` is deployed or available locally.
- The existing authenticated-session response contains the active platform role
  collection used by the frontend auth store.
- The frontend checkout is on its matching feature branch and has installed
  dependencies.

## Implementation Checklist

1. Inventory released non-identity-owned protected routes, navigation items,
   and actions, including their feature permissions and school-context needs.
2. Confirm the auth store exposes a resolved active role collection without
   changing the session response contract.
3. Centralize the exact active platform `System Administrator` predicate.
4. Use the predicate only in shared permission evaluation, route guards, and
   navigation/action visibility helpers.
5. Preserve authentication, session, release, school, subject, approval, and
   safety checks around that predicate.
6. Keep student and guardian self-service in their existing actor-owned or
   guardian-link flows; do not add subject selection or impersonation.
7. Clear school-owned state before loading against a changed school context and
   reject stale prior-context responses.

## Released Surface Inventory

| Surface | Released destinations or actions | Permission/context boundary |
|---|---|---|
| Administration shell | Dashboard plus school, user, role, permission, academic-year, academic-period, guardian, student-profile, class-section, and teacher-assignment list/detail/create/edit routes | Route metadata permissions; school context retained where declared; 11 approved sidebar destinations and approved quick actions use authenticated store permission codes. |
| Teacher workflow | Content, questionnaire, learning-set, grade, attendance, administrative observation, academic-record, and import routes/actions | `teacherWorkflowRoutes` metadata and `hasCapability`; active school, release gates, closed-period correction, and other workflow prerequisites remain separate. |
| Advanced assessment | Authoring, student response, review, grading, student result, reporting, and download actions | `useAdvancedAssessmentAccess`; active school, actor/student ownership, scan, due-date, and safety gates remain separate. |
| Reporting | History, catalog, run detail, definitions, definition detail, lifecycle, and download actions | `useReportingAccess`; active school, lifecycle, output availability, and closed/expired output states remain separate. |
| Platform support | Oversight, decisions, decision detail, diagnostics, audit, approval, revocation, and drill-down actions | `usePlatformSupportAccess`; platform-wide scope remains independent of selected school, while support approval/access/revocation states remain separate. |
| Identity-owned self-service | Student workspace and guardian workspace routes | Explicitly excluded from master override. Existing student-profile ownership, guardian-link, academic-period, and subject context remain required. |

Shared review confirmed all permission-bearing route metadata enters
`authGuards`, administration navigation and quick actions receive
`sessionStore.permissionCodes`, and local workspace gates use
`sessionStore.hasPermission` or the exact shared role predicate. No local
`System Administrator` name checks remain outside the auth session contract.

## Focused Verification

Run from `schoolmaster-frontend`:

```bash
npm run test:unit -- --run tests/unit/system-admin-master
npm run build
```

Confirm the focused tests cover:

- System Administrator direct route access and limited-role denial.
- Protected navigation and action visibility.
- Session restoration before master-access evaluation.
- Missing, inactive, and changed school contexts, including stale-data cleanup.
- Existing identity-owned student and guardian self-service restrictions.
- Approval, confirmation, support, file-safety, closed-period, and other
  prerequisite-specific feedback states.

## Manual Scenarios

1. Sign in as a zero-permission user with the active platform `System
   Administrator` role and open each released non-identity-owned protected
   route group directly and from navigation.
2. Select an active school, verify school-owned data is scoped to it, then
   switch schools and verify prior data disappears before new data arrives.
3. Remove the selected school and verify school-owned pages do not load data.
4. Attempt student and guardian self-service without the normal actor-owned
   profile or guardian link and verify those routes remain unavailable.
5. Trigger a representative approval, confirmation, support, file-safety, or
   closed-period block and verify its existing state remains distinct from a
   permission denial.

## Evidence to Record

- Route, navigation, action, context, and feedback test results.
- Production build output.
- Inventory showing all released non-identity-owned protected frontend route
  groups were covered.
- Confirmation that no endpoint, response schema, OpenAPI, or client-side audit
  change was made.

## Implementation Evidence

Implementation completed on 2026-07-14:

- Exact active platform role name is `System Administrator`; prior loose aliases
  are not accepted.
- Role-derived wildcard permission satisfaction is unavailable until session
  status is authenticated. Limited-role behavior remains unchanged.
- School context switching clears active school, student profile, academic
  period, and registered school-owned state before loading. Generation checks
  ignore stale earlier selection responses.
- Student and guardian self-service contexts remain actor/profile/link-bound;
  selected subject context is never inferred from master role.
- Existing auth, administration, reporting, platform-support, assessment, and
  self-service error mappers retain prerequisite-specific states.
- Direct Axios scan of `src/components`, `src/pages`, and `src/router`: PASS,
  no matches.
- Client-side audit boundary: PASS; state-changing actions continue to use only
  existing submitters and backend audit behavior.
- Targeted ESLint and Prettier checks: PASS.
- Focused System Administrator Vitest suite: PASS, 13 files and 20 tests.
- Full frontend Vitest regression suite: PASS, 326 files and 594 tests.
- Production build: PASS, 2,089 modules transformed. Existing dependency pure-
  annotation and large-chunk warnings remain non-blocking.

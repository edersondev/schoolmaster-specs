# Quickstart: System Administrator Master Access

## Preconditions

- Feature spec exists at `specs/031-system-admin-master/spec.md`.
- OpenAPI remains source of truth for protected operation authorization notes.
- Backend implementation uses feature identifier `031-system-admin-master`.
- Frontend implementation is deferred and no frontend repository changes are
  part of this run.

## Implementation Inventory

### Contract operation groups

The protected-operation inventory is the aggregate of `api/openapi.yaml` and
the referenced files under `api/paths/`:

| Scope | Operation groups |
|-------|------------------|
| Platform | schools, platform school summaries, platform reporting overview, support access, support diagnostics, support audit, account lifecycle, platform roles/users/permissions |
| School | users, roles, permissions, academic years, academic periods, guardians, student profiles, classroom rosters, teacher assignments, teacher content, questionnaires, learning sets, grades, attendance, report catalog, report definitions, report runs, assessment responses |
| Identity-owned | student self-service and guardian self-service; existing actor-owned profile and active guardian-link rules remain authoritative |
| Shared lookup | school lookups and address lookups; authentication/account state remains required where the operation is protected |
| Public | login, invitation setup, password-reset request/completion; excluded from the permission override because their operations declare `security: []` |

The implementation and verification matrix must use the concrete operation IDs
from these groups. Inline platform-wide operation IDs are
`listPlatformSchoolSummaries`, `getPlatformReportingOverview`,
`requestSupportAccess`, `getSupportAccessDecision`, `approveSupportAccess`,
`revokeSupportAccess`, `getSupportSchoolDiagnostics`, and
`listSupportAuditEvents`.

### Backend authorization entry points

- Central permission methods: `User::hasPermission()` and
  `User::hasSchoolPermission()`.
- Policy callers: all policies under `app/Policies/` that call either central
  method, with special review for `ScopePolicy`, `AccountLifecyclePolicy`,
  `AcademicRecordPolicy`, `AcademicRecordImportPolicy`, `AssessmentPolicy`,
  `LearningSetPolicy`, `QuestionnairePolicy`, `TeacherContentItemPolicy`, and
  `TeacherWorkflowPolicy` because they also compare `users.school_id`.
- Direct service callers: permission/role/user/school services, authorization
  concerns, report services, classroom roster services, teacher workflow
  services, and assessment authorization services under `app/Services/`.
- Non-permission gates retained: active account/session, resolved active school,
  tenant ownership, actor-owned student profile, active guardian link, resource
  lifecycle, approval, confirmation, support opt-in, file scan, and closed
  academic period.

### Fixed-school precondition updates

- `ScopePolicy` and `AccountLifecyclePolicy` keep scope, active-school, and
  permission checks but accept the platform master role without a fixed
  `users.school_id`.
- `AcademicRecordPolicy`, `AcademicRecordImportPolicy`, `AssessmentPolicy`,
  `LearningSetPolicy`, `QuestionnairePolicy`, `TeacherContentItemPolicy`, and
  `TeacherWorkflowPolicy` retain record-school, ownership, lifecycle, file-scan,
  and administrator gates while removing only the fixed-school assumption for
  System Administrator.
- `AssessmentReviewAuthorizationService` and
  `AssessmentResponseReviewService` retain active-account, selected-school,
  active-period, ownership, and query-scoping gates while delegating permission
  decisions to the central `User` methods.
- `StudentTransferValidator` returns all active schools only for System
  Administrator; destination existence, active state, selected identifiers,
  and tenant-matching validation remain unchanged.
- `AuthorizesStudentSelfView`, `GuardianAccessResolver`, student/guardian
  policies, platform-support approvals and opt-ins, account/session state,
  file-scan checks, confirmations, and closed-period gates are deliberately not
  bypassed.

### Audit pipelines

- Generic: `AuditEventData` -> `AuditEventService` -> `AuditEvent`, including
  account lifecycle, administration lifecycle, classroom roster, guardian, and
  teacher-workflow adapters.
- Assessment: `AssessmentAuditService` -> `AuditEvent`.
- Reporting: `ReportAuditService` -> `ReportLifecycleEvent`.
- Platform support: `PlatformSupportAuditService` ->
  `PlatformSupportAuditEvent`.
- Covered state changes: create, update, delete, restore, activate, deactivate,
  import, retry, cancel, approve, revoke, and every other protected mutation
  found in the contract inventory.

### Test fixture and evidence plan

- `tests/TestCase.php` provides a zero-permission active platform user with an
  active `System Administrator` role, limited users, active/inactive schools,
  actor-owned student and guardian-link fixtures, and bearer tokens.
- Redocly: `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1` from
  the specs repository.
- Focused PHPUnit: `php artisan test --compact tests/Feature/SystemAdminMasterAccess`
  plus the named unit and current-user tests.
- Frontend route/navigation/action work is deferred to a separate feature; this
  implementation changes no frontend repository file.

## Specification and Contract Validation

1. Review `docs/security.md` and replace contradictory System Administrator
   "no implicit bypass" language with the master-access rule.
2. Review `docs/multi-tenant.md` and document that System Administrator may
   select any active school while school-owned output remains selected-school
   scoped.
3. Review affected existing feature specs and contracts for permission matrices
   or route guard rules that deny System Administrator only because
   feature-specific permissions are missing.
4. Update `api/openapi.yaml` and affected path/component descriptions so
   protected operations document System Administrator master access.
5. Confirm any operation returning cross-school output is documented as
   platform-wide before unscoped output is allowed.
6. Run OpenAPI validation:

```bash
npx @redocly/cli lint api/openapi.yaml
```

Expected result: OpenAPI lint passes and protected operation descriptions use
consistent System Administrator master-access language.

Implementation result (2026-07-13): PASS. `aggregate@v1` and
`schoolmaster-platform@v1` are valid. The mirrored platform contract retains
four pre-existing `no-unused-components` warnings for `ReportRunId`,
`ReportRun`, `ReportRequest`, and `OutputExpired`. All 156 protected operations
under `api/paths/`, all 141 protected operations in the platform mirror, and
all eight inline platform-support operations carry
`x-system-administrator-master-access: true`; the four public unauthenticated
operations do not.

## Backend Validation

1. Add or update centralized authorization behavior so System Administrator
   satisfies feature-specific permission checks.
2. Preserve account/session, tenant-context, actor-ownership, guardian-link, school-state,
   release-state, approval workflow, and safety-gate checks.
3. Verify school-scoped operations require selected active school context and
   return only selected-school data.
4. Verify identity-owned self-service operations retain actor-owned student and
   active guardian-link authorization without impersonation.
5. Record master-access audit evidence for System Administrator writes and
   lifecycle actions.
6. Run backend tests:

```bash
php artisan test
```

Expected result: System Administrator allow cases pass, tenant/subject/safety
gate denials still pass, audit-marker assertions pass, and non-System
Administrator denial behavior remains unchanged.

## Deferred Frontend Validation

1. Make no frontend repository changes in this backend implementation run.
2. Verify the existing authenticated-session roles collection exposes the
   active platform `System Administrator` role without a response schema change.
3. Record route guard, navigation, and action visibility adoption as a separate
   frontend feature.

Deferred task record: route-guard coverage, route-guard implementation,
navigation/action-visibility coverage, navigation helper implementation,
frontend error mapping, active-school selection, and stale-data cleanup belong
to that separate frontend feature. No frontend repository files are changed by
feature 031.

## Manual Review Scenarios

- Sign in as System Administrator with no extra feature-specific permissions.
- Open a platform-scoped protected route and confirm access.
- Select any active school, open school-owned administration, and confirm only
  selected-school data appears.
- Switch to another active school and confirm stale school-owned data clears
  before the new school's data loads.
- Call student and guardian identity-owned self-service operations as System
  Administrator and confirm existing actor-owned profile or guardian-link rules
  deny access without leaking another subject's data.
- Attempt a workflow blocked by approval, confirmation, support opt-in, file
  scan, or closed-period safety state and confirm the safety gate still blocks
  the action.
- Perform a System Administrator write or lifecycle action and confirm audit
  evidence marks master access.

## Evidence to Capture

- OpenAPI lint output.
- Backend authorization and audit test output.
- Current-session role-context regression output and deferred frontend handoff.
- Short notes showing non-System Administrator denial behavior was not
  broadened.

## Final Implementation Evidence

Implementation validation completed on 2026-07-13:

- OpenAPI lint: PASS for `aggregate@v1` and
  `schoolmaster-platform@v1`. The platform mirror retains four pre-existing
  unused-component warnings for `ReportRunId`, `ReportRun`, `ReportRequest`,
  and `OutputExpired`.
- Feature 031 PHPUnit: PASS, 27 tests and 259 assertions across
  `tests/Feature/SystemAdminMasterAccess`, `CurrentUserApiTest`, master-role
  unit coverage, and audit-metadata unit coverage.
- Focused unit PHPUnit: PASS, 6 tests and 17 assertions for exact active
  platform-role detection, limited-role behavior, and canonical audit-marker
  serialization.
- Broader authorization/audit regression run: 138 tests passed with 686
  assertions. One pre-existing fixture failure remains in
  `SchoolDetailUpdateTest`: it supplies string `active` to the integer
  `schools.status` column before feature 031 code is reached.
- Pint: PASS after formatting all dirty PHP files with the project agent
  format.

Verified behavior:

- A zero-permission active platform `System Administrator` reaches all released
  protected read operation groups; a limited platform role remains denied.
- The existing authenticated-session roles collection exposes the exact role
  without adding a response field.
- Any active school may be selected, inactive/unknown context is denied, and
  user, academic, and teacher-content queries remain selected-school scoped.
- Student self-service still requires the actor-owned profile, guardian
  self-service still requires an active guardian link, pending files remain
  unavailable, support opt-in ownership and internal approval remain required,
  lifecycle input remains validated, and closed-period imports remain blocked.
- Every protected unsafe route is covered by the master-mutation audit
  middleware. Existing generic, lifecycle-history, classroom, teacher,
  assessment, report, and platform-support audit pipelines serialize the same
  canonical `master_access_used: true` marker for state changes. Read-only
  navigation adds no marker.
- Audit assertions verify actor, action/route, target, outcome, timestamp, and
  selected school without recording another tenant as context.
- A final direct-check review found no remaining feature permission query that
  bypasses `User::hasPermission()` or `User::hasSchoolPermission()` except
  `StudentTransferValidator`, which now explicitly grants System Administrator
  the active-school destination set while preserving destination validation.
- No frontend repository file changed. Route guards, navigation/actions,
  session-store mapping, error mapping, active-school UI, and stale-data cleanup
  remain deferred to the separate frontend feature.

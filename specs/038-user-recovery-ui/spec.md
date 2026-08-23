# Feature Specification: User Recovery UI

**Feature Branch**: `038-user-recovery-ui`  
**Created**: 2026-08-22  
**Status**: Draft  
**Input**: User description: "Add frontend handling for recoverable_user_conflict during school-user creation. Show a safe warning with a Restore existing user action, reuse the lifecycle dialog to collect reason and effective date, restore the returned user_id, then navigate to edit status, roles, and profile. Never auto-restore or expose deleted-account or cross-tenant details; keep all other duplicate cases generic. Include service, component, and workflow tests."

## Clarifications

### Session 2026-08-22

- Q: What happens when the administrator changes the email after receiving recovery guidance? → A: Clear the warning and recovery reference immediately and require a new creation submission.
- Q: Where should the administrator go after a successful restoration? → A: Open the restored user's detail page, which provides permitted lifecycle actions and access to the existing edit workflow.
- Q: What happens to the recovery workflow when restoration fails? → A: Keep the dialog and entered values for validation or temporary failures; clear recovery for forbidden, not-found, tenant, or terminal state conflicts.
- Q: What happens to the failed creation draft after restoration succeeds? → A: Discard it and load the restored user's authoritative current data on the detail page.
- Q: Where does the recoverable warning text come from? → A: Use localized frontend copy selected by the exact recovery code and ignore the backend message text.

### Session 2026-08-23

- Q: Which restore failures count as temporary and preserve the open dialog? → A: Preserve for validation/422, network failures, timeouts/408, rate limiting/429, and 5xx responses; clear recovery for 401, 403, 404, 409, and every other non-allowlisted HTTP response.
- Q: What happens if restore succeeds but the restored user's detail request fails? → A: Stay on the user detail route, show existing safe detail-load feedback with its normal retry or return actions, and never repeat restoration.
- Q: How should the newly displayed recovery warning behave for assistive technology? → A: Announce it through a polite live region/status, keep current focus unchanged, and place the restore action in normal keyboard order.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Recognize a Recoverable Identity (Priority: P1)

An authorized school administrator who attempts to create a user with an email
owned by a recoverable retained identity receives clear, safe guidance that the
existing user can be restored instead of seeing an unrelated generic record
conflict.

**Why this priority**: The published recovery response is useful only when the
administrator can understand the next permitted action. Clear guidance prevents
repeated creation attempts while preserving the single retained identity.

**Independent Test**: Submit a school-user creation request that returns an
eligible recoverable conflict and verify that the creation form shows one safe
warning and one explicit restore action without exposing the returned identifier
or any retained-user details.

**Acceptance Scenarios**:

1. **Given** an authorized administrator is creating a school user and the creation attempt returns an eligible recoverable-user conflict, **When** the response is presented, **Then** localized frontend copy states `An existing user can be restored.` and offers a `Restore existing user` action.
2. **Given** the recoverable response contains the permitted recovery reference, **When** the warning is displayed, **Then** no user identifier, name, email, school, tenant, status, deletion date, role, profile, or audit detail from the retained identity is shown.
3. **Given** the recoverable warning is displayed, **When** the administrator does not choose restore, **Then** no restore, activation, update, or role change occurs and the entered creation data remains available for deliberate review or cancellation.
4. **Given** the recoverable warning is displayed, **When** the administrator changes the email, **Then** the warning and recovery reference are cleared immediately and another creation submission is required before recovery can be offered again.
5. **Given** an eligible recovery warning is added dynamically, **When** assistive technology observes the page update, **Then** the warning is announced through a polite live region or status without moving current focus and the restore action remains in normal keyboard order.

---

### User Story 2 - Restore and Continue Maintenance (Priority: P2)

An administrator who chooses the recovery action confirms the existing restore
workflow with a reason and effective date. After a successful restore, the
administrator continues to the restored user's detail page, where permitted
lifecycle actions are available and the existing edit workflow provides role
and profile maintenance.

**Why this priority**: Restoration must remain explicit and audit-sensitive.
Continuing to the existing user detail page completes the recommended
recovery journey without turning creation into an automatic update operation.

**Independent Test**: From an eligible recovery warning, open the restore
confirmation, validate required inputs, complete restoration, and verify
navigation to the restored user's detail page without any automatic
status, role, or profile changes.

**Acceptance Scenarios**:

1. **Given** an eligible recoverable warning is current, **When** the administrator selects `Restore existing user`, **Then** the existing lifecycle confirmation requests the required reason and effective date before any restore request can be submitted.
2. **Given** required recovery inputs are missing or invalid, **When** restoration is submitted, **Then** actionable validation is shown, the confirmation remains available, and no lifecycle change is presented as successful.
3. **Given** restoration succeeds, **When** the result is received for the same active school context, **Then** the administrator is directed to the restored user's detail page with permitted lifecycle actions and access to the existing role and profile edit workflow.
4. **Given** restoration succeeds, **When** the user detail page opens, **Then** no status, role, profile, activation, or invitation change has been performed merely because recovery was selected.
5. **Given** the failed creation form contained values that differ from the retained user, **When** restoration succeeds, **Then** those creation values are discarded and the detail page presents the restored user's authoritative current data.
6. **Given** restoration returns validation/422, a network failure, timeout/408, rate limiting/429, or a 5xx response, **When** the failure is presented, **Then** the confirmation and entered values remain available for correction or an explicit retry and no replacement identity is created.
7. **Given** restoration returns 401, 403, 404, 409, or another non-allowlisted HTTP status, including session, forbidden, tenant-denial, not-found, terminal state-conflict, or unexpected HTTP outcomes, **When** the failure is presented, **Then** the administrator receives existing safe lifecycle feedback, the recovery warning and reference are cleared, and a new creation submission is required before recovery can be offered again.
8. **Given** restoration succeeded and the administrator reached the restored user's detail route, **When** the authoritative detail request fails, **Then** the existing detail page shows safe load feedback with its normal retry or return actions and does not repeat restoration or reopen recovery.

---

### User Story 3 - Preserve Generic Duplicate Privacy (Priority: P3)

Administrators continue to receive generic validation or conflict feedback for
all creation failures that are not an exact, valid recoverable-user response.
The client does not infer or reveal whether another identity is active,
inactive, invited, deleted, platform-owned, or in another school.

**Why this priority**: Recovery convenience must not weaken the non-disclosure
boundary established for identity email ownership and tenant isolation.

**Independent Test**: Submit active, inaccessible, unauthorized, cross-tenant,
platform-scope, malformed, and concurrent duplicate cases and verify that none
offers recovery or exposes retained identity information.

**Acceptance Scenarios**:

1. **Given** user creation returns generic unavailable-email validation, **When** feedback is shown, **Then** only the documented generic email message is displayed and no restore action is offered.
2. **Given** a conflict code is not the exact recoverable-user code, **When** feedback is shown, **Then** it retains the existing generic conflict treatment and does not consume any recovery-looking details.
3. **Given** the recoverable-user code is present but the recovery reference or recommendation is missing, malformed, or unsupported, **When** feedback is shown, **Then** no restore action is offered and the response is handled as a safe generic failure.
4. **Given** the administrator changes school, loses the required context, signs out, or leaves the creation workflow after receiving recovery guidance, **When** the stale action could otherwise be used, **Then** the recovery reference is discarded and cannot submit into the new context.

### Edge Cases

- The administrator submits the create form repeatedly while the first request is pending; only one active request and one resulting recovery state may control the page.
- The same recoverable response is received more than once; the interface shows one current recovery action and does not queue multiple restores.
- The administrator changes the email after receiving recovery guidance; the prior recovery state is cleared immediately even if the new email later resolves to the same identity.
- The recovery response contains extra fields; the client ignores them and never renders or records them as user-facing diagnostics.
- The recovery reference is not a valid public user identifier or the recommendation is not exactly `restore`; no recovery action is enabled.
- The active school or session changes while the create or restore request is in flight; stale results do not navigate, replace current feedback, or submit into the new context.
- The retained user is restored, deleted again, or otherwise changes state between creation rejection and restore submission; the restore result remains authoritative.
- Restore succeeds but the authoritative detail request fails; the administrator remains on the user detail route, receives the existing safe detail-load feedback and normal retry or return actions, and cannot repeat restoration or reopen recovery from that result.
- Restore validation returns field errors for reason or effective date; entered confirmation values remain available for correction.
- Validation/422, network failures, timeouts/408, rate limiting/429, and 5xx restore responses preserve the confirmation for correction or an explicit retry; 401, 403, 404, 409, and every other non-allowlisted HTTP status clear the stale recovery workflow.
- Failed creation values differ from the retained user's current data; successful restoration discards the draft and loads only authoritative restored-user data.
- A generic duplicate response includes no recovery reference; the client does not attempt an additional lookup to discover the matching identity.
- Browser back, refresh, or copied URLs do not persist or disclose the recovery identifier beyond the active recovery workflow.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No backend behavior change is required. The published recoverable-user conflict, generic duplicate validation, user restore, and user update behavior from Feature 037 remain authoritative.
- **Frontend repository impact**: Extend school-user creation feedback to recognize the exact recoverable outcome, preserve only the minimum recovery reference for the active workflow, offer the existing lifecycle confirmation, and continue to the restored user's detail page. Add service-level, component-level, and complete workflow regression coverage.
- **Specification or contract repository impact**: Add this frontend consumption specification. During planning, document the mapping between the existing creation conflict, lifecycle confirmation, restore result, maintenance navigation, and privacy fallbacks. OpenAPI changes are required only if contract review finds the published Feature 037 responses insufficient.
- **Delivery ownership and sequencing**: `schoolmaster-specs` is the delivery lead and defines and approves the client behavior first. `schoolmaster-frontend` implements it only after specification approval, using the shared Feature 038 identifier and linking its branch or pull request to the approved specification change. No backend delivery is expected unless contract verification finds a defect.

### API Contract Impact

- **OpenAPI update required**: No. This feature consumes the existing documented contracts without changing request or response shapes.
- **Versioned endpoints affected**: Existing school-scoped user creation, single-user restoration, user detail, user update, and user activation or deactivation operations support the journey. No new endpoint is introduced.
- **JSON response impact**: No response shape changes. The client recognizes only the exact recoverable-user conflict with its documented user reference and `restore` recommendation. All other responses retain their current generic mappings.
- **Authentication/authorization impact**: Existing authentication, active-school context, and user view/manage authorization remain authoritative. Client action visibility is advisory; every restore, view, update, and lifecycle request must still be authorized independently.
- **Compatibility impact**: Additive frontend behavior. Clients that do not implement recovery continue to receive a rejected create operation, and existing generic conflict and validation behavior remains unchanged.

### Data & Tenancy Impact

- **Tenant scoping impact**: The recovery reference is valid only in the exact active school context that received it. A school change clears the reference and requires a new authorized workflow.
- **Cross-tenant or platform access impact**: The client performs no discovery lookup and offers no recovery action for generic, cross-tenant, opposite-scope, inaccessible, or platform-user collisions.
- **Soft delete impact**: No deletion or retention semantics change. The client may state only that an existing user can be restored when the authorized recoverable response permits that guidance; it does not display deletion metadata or infer recoverability from any other response.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The user-creation experience MUST recognize the exact `recoverable_user_conflict` outcome separately from generic record conflicts.
- **FR-002**: A valid recoverable outcome MUST present the localized frontend warning `An existing user can be restored.` and MUST provide one `Restore existing user` action. The warning MUST be selected by the exact recovery code.
- **FR-003**: The recoverable warning MUST NOT display the returned user identifier or any retained user's name, email, school, tenant, lifecycle state, deletion date, role, profile, audit, or other account details.
- **FR-004**: The client MUST use the returned user reference only for the explicit recovery workflow and MUST NOT perform a discovery lookup to enrich the warning before restoration.
- **FR-005**: A creation attempt and its feedback MUST NOT automatically restore, activate, deactivate, update, invite, or reassign roles to the retained user.
- **FR-006**: Selecting the recovery action MUST open the existing lifecycle confirmation and MUST require a valid reason and non-future effective date before restore submission.
- **FR-007**: The restore submission MUST use the exact active school context associated with the recoverable creation result.
- **FR-008**: While a create or restore submission is pending, repeated submission controls MUST be disabled or deduplicated so that one user action cannot create concurrent duplicate requests.
- **FR-009**: On successful restoration, the administrator MUST be directed to the restored user's detail page, where permitted lifecycle actions and access to the existing role and profile edit workflow are available. If its authoritative detail request fails, the application MUST remain in the detail workflow, use the page's existing safe retry or return behavior, and MUST NOT repeat restoration or reopen recovery.
- **FR-010**: After restoration, status changes, role assignment, and profile changes MUST remain separate, explicit actions governed by existing authorization and lifecycle rules. Successful restoration MUST discard the failed creation draft and load the restored user's authoritative current data; creation values MUST NOT prefill, overwrite, or be represented as retained-user data.
- **FR-011**: Restore validation/422, network-failure, timeout/408, rate-limit/429, and 5xx outcomes MUST use existing safe lifecycle feedback, preserve the confirmation and entered values, and permit only a deliberate correction or retry. Restore 401, 403, 404, 409, and every other non-allowlisted HTTP outcome MUST clear the recovery warning and reference and require a new creation submission before recovery can be offered again. No failure outcome may trigger an automatic retry or replacement creation.
- **FR-012**: Generic unavailable-email validation MUST retain its documented field feedback and MUST NOT offer restoration.
- **FR-013**: Conflict codes other than `recoverable_user_conflict` MUST retain generic conflict treatment and MUST NOT consume recovery-looking response details.
- **FR-014**: Recovery MUST be offered only when the response contains the exact recoverable code, a valid public user identifier, and the exact supported `restore` recommendation; malformed or unsupported variants MUST fail safely without a recovery action.
- **FR-015**: An email change, school change, session end, permission-context reset, route departure, cancellation, or newer creation result MUST immediately clear any retained recovery warning and reference and prevent stale recovery submission or navigation. After an email change, recovery MUST require a new creation submission.
- **FR-016**: User-facing diagnostics, telemetry, and logs for this workflow MUST NOT contain the submitted email, recovery identifier, lifecycle reason, hidden account data, authentication data, or tenant data.
- **FR-017**: The workflow MUST ignore and not render any response fields beyond those explicitly required to classify and execute the documented recovery action. Backend message text and unexpected fields MUST NOT be shown or combined with localized recovery guidance.
- **FR-018**: Verification MUST cover response classification and safe mapping, warning and action rendering, lifecycle confirmation validation, successful restore and maintenance navigation, restore rejection paths, generic duplicate privacy, malformed recovery details, repeated submission, and school or route changes during in-flight requests.
- **FR-019**: Verification MUST include isolated service behavior, user-surface component behavior, and the complete creation-to-restoration-to-maintenance journey.
- **FR-020**: This feature MUST NOT introduce a new backend endpoint, change email ownership, enable platform-user restoration, add cross-tenant recovery, or change existing restore/update authorization.
- **FR-021**: All consumed response codes, fields, operations, and authorization assumptions MUST be verified against the published contract before frontend implementation begins.
- **FR-022**: `schoolmaster-specs` MUST lead delivery. Implementation spanning specification and frontend repositories MUST follow specification approval first and frontend delivery second, and the frontend branch or pull request MUST use the shared Feature 038 identifier and link to the approved specification change.
- **FR-023**: The recovery warning MUST be announced through a polite live region or status, MUST NOT move the administrator's current focus when it appears, and MUST expose the restore action in normal keyboard order.

### Key Entities

- **Recoverable Creation Feedback**: The safe, temporary client state produced only by an exact eligible recoverable-user conflict. It contains the minimum internal recovery reference and supported recommendation but exposes neither to the administrator.
- **Recovery Reference**: The returned public user identifier held only for the current active-school recovery workflow and cleared whenever that workflow or context becomes stale.
- **Restore Confirmation**: The explicit lifecycle decision containing the administrator's required reason and effective date before restoration.
- **Restored User Detail Journey**: The post-restore destination where permitted lifecycle actions are available and role or profile changes remain in the separate existing edit workflow.
- **Generic Duplicate Feedback**: The non-recovery validation or conflict state used for every duplicate outcome not matching the exact safe recovery contract.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In 100% of tested valid recoverable-user conflicts, administrators see specific safe recovery guidance and one restore action instead of the generic record-state conflict.
- **SC-002**: In 100% of tested generic, malformed, unauthorized, cross-context, or unsupported duplicate outcomes, no restore action or retained-user detail is shown.
- **SC-003**: An authorized administrator can move from recoverable creation feedback to a completed restore confirmation in no more than two deliberate user actions, excluding required reason and date entry.
- **SC-004**: In 100% of successful recovery journeys, the administrator reaches the restored user's detail page without a second identity or any automatic status, role, or profile change being created.
- **SC-005**: In 100% of tested context changes and stale in-flight results, no recovery request or navigation is applied to the wrong school, session, or route.
- **SC-006**: All specified service, component, and complete-workflow verification scenarios pass before release, including every documented success, privacy fallback, validation, conflict, and stale-context path.
- **SC-007**: During moderated acceptance testing with a preselected cohort of at least 10 authorized administrators, at least 90% MUST, without assistance, identify that the existing identity must be restored rather than a new identity created and select `Restore existing user` as the correct next action. A participant passes only when both observations are satisfied.

## Assumptions

- Feature 037 is merged and its recoverable conflict, generic duplicate validation, restore, and update contracts are available in the target environment.
- The existing administration lifecycle confirmation already supports required reason and effective-date validation for user restoration.
- The existing user detail page exposes permitted lifecycle actions and access to the edit workflow, while role and profile edits remain within that edit workflow.
- The recoverable response is issued only after backend authorization and exact-school checks; the frontend still clears stale context and relies on backend authorization for every follow-up request.
- The recovery reference is transient workflow state and is not placed in user-visible URLs, persistent browser storage, analytics, or logs.
- No platform-user recovery experience is included.
- Existing localization, responsive-layout, and safe-feedback conventions apply to the warning, action, confirmation, and failure states. The recovery warning uses a polite live region or status without automatic focus movement, and its action remains in normal keyboard order. Backend message text is not a user-facing copy source.

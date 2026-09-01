# Student Guardian Tabs Contract

This contract defines what the API and frontend may implement for Student
Guardian Tabs.

## Scope Contract

This feature approves:

- Removing the standalone Guardians item from the administration sidebar.
- Redirecting direct standalone guardian administration routes to Create
  Student without loading guardian list data.
- Replacing Create Student with a tabbed Student and Guardians workflow.
- Submitting a student with zero, one, or two guardian entries.
- Creating new same-school guardians from the Guardians tab.
- Linking existing active same-school guardians from the Guardians tab.
- Allowing duplicate approved relationship labels across the two guardian
  entries.
- Enforcing guardian capture permission separately from student creation.
- Safe validation, denial, conflict, stale-response, and temporary failure
  feedback.

This feature does not approve:

- A standalone sidebar entry for guardian administration.
- Frontend use of standalone Guardian create as a chained second step after
  student creation.
- More than two guardians for one student through this workflow.
- Guardian self-service, guardian user-link management, lifecycle actions, bulk
  lifecycle actions, student transfer, roster membership, teacher assignment,
  reporting, messaging, billing, or undocumented behavior.
- Cross-school guardian references or cross-tenant association.

## OpenAPI Contract

OpenAPI must be updated before backend or frontend implementation.

### Operations

| Operation ID | Method and path | Feature use |
|--------------|-----------------|-------------|
| `createStudentProfile` | `POST /api/v1/student-profiles` | Create one same-school student with zero to two guardian entries in one atomic request. |
| `listGuardians` | `GET /api/v1/guardians` | Lookup active same-school existing guardians for selection inside the Guardians tab. |

The frontend must not use `createGuardian` as a chained post-student-create
step for this workflow. New guardian creation for this feature is part of the
student create request.

### Student Create Guardian Payload

`StudentProfileCreateRequest.guardian_associations` must allow zero to two
entries.

Each entry must include:

- `relationship_type`: required approved relationship label.

For an existing guardian entry, the entry must include:

- `guardian_id`: active same-school guardian UUID.

For a new guardian entry, the entry must include:

- `full_name`: required guardian name.
- `contact_email`: optional valid email.
- `contact_phone`: optional approved phone value.

Each entry must use exactly one mode: existing guardian reference or new
guardian data. Tenant ownership fields, status override fields, user-link
fields, lifecycle fields, and self-service fields are not accepted.

### Validation Requirements

The contract must document validation for:

- More than two guardian entries.
- Duplicate existing `guardian_id` references.
- Duplicate relationship labels must be accepted when each guardian identity or
  reference is otherwise valid.
- Same entry containing both `guardian_id` and new guardian identity fields.
- Same entry containing neither `guardian_id` nor new guardian identity fields.
- Missing or invalid `relationship_type`.
- Missing new guardian `full_name`.
- Invalid guardian contact email or phone.
- Missing, inactive, deleted, or cross-school existing guardian references.
- Actor lacking guardian management authority when entries are present.
- Any student field validation from the existing student creation contract.

### Response Contract

Success response for `createStudentProfile` must allow the frontend to confirm:

- created student profile
- zero to two associated guardian records
- relationship labels for created associations

Failure responses must use documented envelopes for:

- validation errors with field paths that identify Student or Guardians tab
  fields
- unauthorized
- forbidden
- tenant mismatch or inactive school
- not found for inaccessible references without cross-tenant disclosure
- conflict for duplicate or incompatible state
- temporary unavailable

The operation must be atomic: no student, new guardian, or association is
committed when any submitted guardian entry fails.

## Route Contract

- Administration sidebar metadata must not include a visible Guardians item.
- Permission ownership for hidden guardian route records may remain in source
  code only to support redirect behavior and future compatibility.
- Direct or bookmarked guardian list, create, detail, lifecycle, and bulk
  lifecycle administration routes redirect to Create Student.
- Redirects preserve only safe navigation context and active school context.
- Redirects must not call `listGuardians`, `getGuardian`, `createGuardian`,
  `updateGuardian`, lifecycle, or bulk lifecycle operations.
- Missing authentication, expired session, missing school, inactive school, or
  forbidden Create Student access follows existing protected-route behavior.

## Permission Contract

| Surface | Required authority |
|---------|--------------------|
| Create Student without guardians | Existing student profile creation authority |
| Guardians tab visibility or entry editing | Existing student profile creation authority and `guardians.manage` |
| Existing guardian lookup inside Guardians tab | `guardians.manage` and `guardians.view` where existing permission model requires list visibility |
| Student create request with one or two guardian entries | Existing student profile creation authority and `guardians.manage` |
| Former standalone guardian sidebar item | Never visible |

Backend authorization remains authoritative. Frontend visibility only prevents
invalid user actions.

## Frontend Form Contract

- Create Student contains Student and Guardians tabs in one route-level
  workflow.
- Student tab owns approved student fields.
- Guardians tab starts empty and supports adding up to two entries.
- Each guardian entry can be new guardian or existing same-school guardian.
- Tab switching preserves values and errors.
- A third entry cannot be added or submitted.
- Validation summary identifies the affected tab and field.
- Student-only creation remains available when guardians are omitted.
- Actors without guardian management authority can create a student without
  guardians, but cannot add, edit, or submit guardian entries.
- Submit is disabled or ignored while pending.
- Successful submission clears dirty state.
- Dirty navigation from either tab uses existing unsaved-change confirmation.

## Service and State Contract

- Axios calls exist only in service/API-client boundaries.
- Create Student page and tab components never call endpoints directly.
- Student create service maps camelCase form state to OpenAPI request fields.
- Guardian lookup service uses only `listGuardians` with documented tenant
  context, page, per-page, status, full-name, and contact-email filters.
- Route-local composables own tab state, dirty state, lookup state, validation
  mapping, pending state, and stale-response protection.
- Existing Pinia session and shell stores remain the source for user,
  permissions, active school, and navigation shell state.
- No sensitive guardian draft is persisted beyond route-local form state.

## Feedback and Privacy Contract

Supported normalized outcomes:

- loading
- validation
- maximum guardians
- duplicate guardian
- unauthorized
- forbidden
- tenant mismatch
- inactive school
- not found
- conflict
- temporary unavailable
- stale response
- unknown error

Visible errors and diagnostics must not expose protected student data, guardian
contact details, full request payloads, token values, permission payloads,
roles, or cross-tenant record existence.

## Verification Contract

OpenAPI and backend verification must cover:

- `guardian_associations` maximum of two entries.
- zero guardian submission.
- one new guardian submission.
- two new guardian submission.
- one or two existing same-school guardian references.
- mixed new and existing guardian entries.
- duplicate existing guardian rejection.
- third guardian rejection.
- invalid new guardian contact validation.
- invalid relationship validation.
- duplicate relationship label acceptance.
- inactive, missing, deleted, and cross-school existing guardian rejection.
- actor without guardian management authority submitting guardian entries.
- transaction rollback when any guardian entry fails.
- response shape includes created student and associated guardians.
- no-sensitive-data error and diagnostics behavior.

Frontend verification must cover:

- no standalone Guardians sidebar item for any permission combination.
- direct guardian route redirect to Create Student with no guardian list load.
- Create Student tab rendering and tab switching.
- zero, one, and two guardian submissions.
- blocked third guardian entry.
- new guardian and existing guardian modes.
- permission-gated Guardians tab behavior.
- tab-scoped validation summary and field errors.
- stale lookup and submit response protection.
- tenant change and unsaved-change handling.
- responsive and keyboard review at 390px, 768px, and 1440px.
- no direct Axios outside services and no endpoint strings in components,
  pages, composables, or router guards.

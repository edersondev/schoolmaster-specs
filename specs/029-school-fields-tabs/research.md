# Research: School Fields Tabs

## Decision: Treat the School contract as a breaking contract-first change

**Rationale**: The feature renames `cnpj` to `document`, changes status values
from strings to numeric `1`/`0`, makes address required, adds Institutional
references, and adds logo upload behavior. These changes affect request
payloads, response shapes, validation errors, backend persistence, and frontend
forms, so OpenAPI must lead implementation.

**Alternatives considered**: Frontend-only labels were rejected because the
clarification requires `document` everywhere. A compatibility-only additive
contract was rejected because numeric status and required address intentionally
change existing behavior.

## Decision: Use `document` as the canonical school identifier field

**Rationale**: The clarified requirement replaces the existing `cnpj` contract
field with `document` across create, update, read, validation, and frontend
form contracts. The value remains Brazilian CNPJ digits and is immutable after
creation.

**Alternatives considered**: Keeping `cnpj` with a UI label was rejected by
clarification. Supporting both fields was rejected because it creates ambiguous
validation and mapping behavior for a breaking contract update.

## Decision: Enforce uniqueness for INEP code, document, and email

**Rationale**: `inep_code`, `document`, and `email` identify or contact a
specific school and must not be shared across school records. Enforcing
uniqueness in backend validation and persistence prevents duplicate identities
and ambiguous administration/search behavior. Uniqueness applies across all
schools, including inactive and soft-deleted records, so restore and audit
flows cannot collide with later-created schools. Email uniqueness is
case-insensitive because users treat differently cased addresses as the same
school contact identity.

**Alternatives considered**: UI-only duplicate checks were rejected because
backend remains authoritative. Allowing duplicate email was rejected because the
user explicitly requires uniqueness for all three fields. Reusing identifiers
from inactive or soft-deleted schools was rejected because it creates lifecycle
and restore ambiguity. Case-sensitive email uniqueness was rejected because it
allows duplicate contacts that differ only by casing.

## Decision: Preserve existing update conflict behavior

**Rationale**: This feature refactors the school profile contract and tabbed
form. It does not introduce a new version token, `updated_at` precondition, or
optimistic-locking conflict flow for simultaneous school edits. Existing
backend update semantics remain authoritative.

**Alternatives considered**: Adding optimistic locking was rejected because it
would expand the API contract and frontend recovery workflow beyond the field
refactor. Implementing frontend-only stale-save detection was rejected because
backend authorization and persistence remain authoritative.

## Decision: Make Address required on create and edit

**Rationale**: The Address tab is now a required school profile group. Current
address field names and numeric rules remain the source of truth, but the
address object itself can no longer be omitted or null for create/edit saves.

**Alternatives considered**: Optional/null address was rejected by
clarification. Required only on create was rejected because edit saves must
also repair incomplete existing records.

## Decision: Do not migrate or backfill existing school-owned data

**Rationale**: The feature is allowed to break and reset any existing
school-owned data during pre-rollout setup or seed cleanup instead of carrying
legacy shapes forward. Runtime create/edit behavior must not delete unrelated
school-owned records. This keeps the implementation focused on the new
canonical contract. Runtime payloads using legacy `cnpj` are rejected
immediately with no alias support, avoiding compatibility code for `cnpj`,
string statuses, nullable addresses, missing Institutional/Branding fields, or
dependent school-owned records.

**Alternatives considered**: Backfilling existing records was rejected because
the user explicitly does not require existing school-owned data preservation.
Limiting resets to only the School profile was rejected by clarification.
Dual-read or dual-write compatibility was rejected because it would prolong
deprecated contract behavior.

## Decision: Use numeric school status values `1` and `0`

**Rationale**: `docs/school_new_fields.md` and clarification define status as
`1` active and `0` inactive. OpenAPI must update school request/response
schemas and validation errors so frontend and backend do not depend on the old
`active`/`inactive` string contract.

**Alternatives considered**: Keeping the existing shared `Status` schema was
rejected by clarification. Accepting both numeric and string values was
rejected because it weakens test expectations and prolongs ambiguous client
behavior.

## Decision: Expose Institutional options through lookup/reference operations

**Rationale**: Institutional fields use stable IDs with labels from
`docs/school_new_fields.md`. Documented lookup/reference operations keep the
frontend from hardcoding option labels and allow future label changes or
translation without a UI release.

**Alternatives considered**: Hardcoded frontend/backend constants were rejected
because they duplicate contract data. Submitting IDs with no option source was
rejected because administrators need selectable, reviewable labels.

## Decision: Seed Institutional lookup options during backend setup

**Rationale**: Option IDs and labels are static reference data from
`docs/school_new_fields.md`. Seeding them during backend setup gives the
lookup/reference endpoints stable numeric IDs without introducing runtime
management workflow.

**Alternatives considered**: Creating options lazily on first request was
rejected because it couples reads to writes and complicates tests. Managing
lookup options through administration UI was rejected as out of scope.

## Decision: Use JSON requests by default and multipart requests when logo file is present

**Rationale**: School create/edit normally submit structured profile data. Logo
upload adds binary content only when a file is selected. Supporting JSON for
normal submissions and multipart/form-data for logo submissions keeps no-logo
requests simple while documenting the file path clearly.

**Alternatives considered**: Always using multipart was rejected because it
adds complexity to every save path. A separate logo upload-only endpoint was
rejected for create because it cannot attach a logo before the school exists
without a multi-step workflow and partial-failure rules.

## Decision: Limit logo files to PNG, JPEG, or WebP up to 2 MB

**Rationale**: Logo files are user-provided uploads and must be validated and
sanitized before persistence. PNG, JPEG, and WebP cover common school logo
formats while avoiding SVG scripting and executable content risk. A 2 MB cap is
enough for a logo and keeps upload validation predictable.

**Alternatives considered**: Allowing SVG was rejected because safe
sanitization is additional scope. Larger or unrestricted uploads were rejected
because logos do not need large binary files. Hiding logo upload was rejected
by clarification.

## Decision: Delete old logo only after successful replacement

**Rationale**: The clarified scope adds upload/replacement behavior and old-file
cleanup after replacement, not a user-facing logo removal lifecycle. Create
without a file results in no logo. Edit without a new file preserves the
current logo reference. When a new logo is successfully stored and referenced,
the previous logo file is deleted to avoid orphaned storage.

**Alternatives considered**: Keeping old logo files was rejected because it
creates storage leakage. Scheduled cleanup only was rejected because replacement
has enough context for immediate safe cleanup. Treating blank logo as delete
was rejected because it risks accidental asset removal and was not requested.
Adding a separate delete action was rejected as lifecycle scope expansion.

## Decision: Backend uses DTO + Service for profile and logo changes

**Rationale**: The new School payload spans Basic, required Address,
Institutional references, Branding colors, logo upload, defaults, and edit-only
immutability rules. A DTO gives Form Requests and Services a clear validated
input boundary; a School service keeps controllers thin and coordinates
persistence, logo storage, and response preparation.

**Alternatives considered**: Controller-heavy implementation was rejected by
constitution. A Repository is not planned because the expected data access is
not complex enough beyond normal model relationships and lookup tables.

## Decision: Frontend uses a route-local tabbed form composable

**Rationale**: School create/edit form state is route-local and must preserve
values across validation, tab changes, and recoverable errors. A composable can
coordinate dirty state, validation routing to tabs, lookup loading, logo file
state, and stale response guards while services own transport.

**Alternatives considered**: A global Pinia school form store was rejected
because form state does not need to cross route boundaries and could leak stale
school data. Direct HTTP calls inside components were rejected by architecture.

## Decision: Validate through OpenAPI, PHPUnit, and Vitest

**Rationale**: This feature changes shared contracts and critical create/edit
behavior. OpenAPI validation catches contract drift, PHPUnit verifies backend
validation/authorization/persistence/upload behavior, and Vitest verifies UI
tab grouping, mapping, errors, lookups, upload handling, and stale states.

**Alternatives considered**: Manual-only verification was rejected because the
feature includes breaking contract changes. Frontend-only tests were rejected
because backend request/resource behavior changes materially.

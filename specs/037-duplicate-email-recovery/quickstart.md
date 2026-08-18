# Quickstart: Duplicate Email Recovery Verification

## Preconditions

- Feature 037 specification and OpenAPI changes are applied before backend code.
- Backend tests run against MySQL, not SQLite.
- Test actors exist for school `users.view` plus `users.manage` and platform
  `account_lifecycle.manage` permission combinations.
- The feature migration adding stored generated `identity_email_key` and
  `users_identity_email_key_index` is ready for MySQL verification.

## Contract verification

1. Bundle modular OpenAPI to a temporary file and review it:

   ```bash
   npx @redocly/cli bundle api/openapi.yaml --output /tmp/feature-037-openapi.yaml
   ```

2. Regenerate `specs/001-schoolmaster-platform/contracts/openapi.yaml` from the
   reviewed aggregate contract.
3. Lint both named APIs:

   ```bash
   npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
   ```

4. Confirm `createUser` publishes 201, 403, 409, and 422; invitation creation
   retains 201, 401, 403, 409, 422, and 503.
5. Confirm recovery 409 contains only standard envelope values plus public user
   UUID and `recommended_action=restore`.
6. Confirm generic 422 uses `error.details.fields.email` array with exact
   `The email is unavailable.` text.
7. Confirm invitation 409 retains only its existing lifecycle conflict contract
   and platform-provisioning email collisions are documented as generic 422.
8. Confirm `restoreUser` documents exact school context with `users.view` and
   `users.manage` and does not imply platform-user restoration.

## Backend verification

1. Create then soft-delete a school user. With exact school context plus
   `users.view` and `users.manage`, create with same normalized email and
   assert 409 recovery response, no duplicate/role row, and one safe audit.
2. Restore returned UUID through existing lifecycle endpoint, then update
   status, roles, and profile through existing operations. Assert original user
   identity and history remain. In a separate post-409 case, introduce a current
   dependency, uniqueness, or lifecycle blocker and assert the restore rejects
   without creating a replacement identity.
3. Repeat without either `users.view` or `users.manage`; assert generic 422 and
   audit has no target.
4. Test active, inactive, invited, cross-school, platform-opposite-mode,
   inactive-parent-scope, and missing-restore-authority targets; assert
   byte/shape-equivalent generic 422 and no target disclosure.
5. Submit exact, case-variant, surrounding-whitespace, and over-255-character
   input through direct create, platform invitation provisioning, and email
   update. Assert valid new or changed values store trimmed/lowercase, oversized
   values fail validation, and untouched legacy mixed-case rows remain unchanged.
6. For platform invitation provisioning, assert duplicate failure creates no
   user, role assignment, invitation, setup credential, delivery request, or
   email submission, always returns the generic 422, and never exposes a target
   UUID even for a soft-deleted platform-owned user.
7. Force `UniqueConstraintViolationException` for `users_email_unique` after
   precheck. Assert complete transaction rollback, exact generic 422, and one
   post-rollback audit with `reason_code=persistence_conflict`.
8. Force a different unique index violation and assert it is rethrown, not
   mislabeled as email validation.
9. Run barrier-controlled two-connection MySQL race repeatedly. Assert at most
   one user commits, every loser receives generic 422, and no partial state or
   plaintext audit data exists.
10. Assert each duplicate audit contains exactly one row with the expected
    actor, resolved school or platform scope, outcome, source IP, workflow,
    reason code, and canonical email hash; target UUID appears only for
    authorized recovery and metadata contains no plaintext email.
11. Assert `identity_email_key` is generated for legacy mixed-case/whitespace
    rows, `EXPLAIN` selects `users_identity_email_key_index`, ambiguous canonical
    legacy owners stay generic, and each availability decision executes at most
    one retained-owner lookup.

## Focused commands

```bash
docker exec schoolmaster-backend-app-1 php artisan test tests/Feature/Api/V1/UserDuplicateEmailRecoveryTest.php
docker exec schoolmaster-backend-app-1 php artisan test tests/Feature/AccountLifecycle/AccountInvitationDuplicateEmailTest.php
docker exec schoolmaster-backend-app-1 php artisan test tests/Feature/Api/V1/UserDuplicateEmailConcurrencyTest.php
docker exec schoolmaster-backend-app-1 php artisan test tests/Feature/IdentityEmailKeyTest.php
docker exec schoolmaster-backend-app-1 php artisan test tests/Unit/Services/Users/DuplicateEmailAuditServiceTest.php
docker exec schoolmaster-backend-app-1 php artisan test tests/Unit/Policies/AdministrationLifecyclePolicyTest.php
docker exec schoolmaster-backend-app-1 php artisan test tests/Unit/ApiResponseTest.php
docker exec schoolmaster-backend-app-1 php artisan test --filter=IdentityEmail
docker exec schoolmaster-backend-app-1 vendor/bin/pint --test
```

Run full backend regression after focused checks:

```bash
docker exec schoolmaster-backend-app-1 php artisan test
```

## Release checks

- Contract repository is published before backend deployment.
- Run the generated-key schema migration before deploying backend code; no
  stored-email backfill is required.
- Existing invitation clients keep the same 409 `conflict` behavior and must
  handle the documented generic 422 for provisioning email collisions.
- Monitor `user_creation_duplicate_email` volume and persistence-conflict
  outcomes without logging plaintext email.
- Frontend deployment is not required for feature completion.

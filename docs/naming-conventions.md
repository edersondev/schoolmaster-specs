# Naming Conventions

## Repository Names

- `schoolmaster-specs`
- `schoolmaster-backend`
- `schoolmaster-frontend`

## Documentation

- Use concise English titles
- Keep file names lowercase with hyphens
- Number architecture decisions with a three-digit prefix
- Add index files when a directory is intended to be consumed directly by
  backend or frontend repositories

## Contracts

- Prefer stable, explicit operation names
- Keep versioned API paths under `/api/v1`
- Use consistent schema naming once promoted into OpenAPI

## Domain Entities

- Use singular PascalCase for domain entities and OpenAPI schemas, such as
  `School`, `AcademicYear`, `StudentProfile`, and `LearningSet`.
- Use snake_case JSON fields and database columns, such as `school_id`,
  `academic_year_id`, and `recorded_by_user_id`.
- Use `school_id` as the concrete tenant identifier for v1 school-owned
  records. Use `tenant` only when describing the architecture pattern.

## Operation IDs

- Use lowerCamelCase verb-noun operation IDs, such as `listSchools`,
  `createAcademicYear`, and `requestReport`.
- Prefer verbs that reflect the contract action: `list`, `get`, `create`,
  `update`, `delete`, `activate`, `deactivate`, `publish`, or `request`.
- Operation IDs must remain stable once consumed by backend or frontend work.

## Frontend Modules

- Use lowercase hyphenated filenames for frontend services and tests, such as
  `academic-years.js` and `school-onboarding.spec.js`.
- Use feature-aligned folder names for major frontend areas, such as
  `admin-system`, `teacher`, `student`, `guardian`, and `reporting`.
- Use PascalCase for Vue component names and for Element Plus component tags in
  templates, such as `AssessmentTable`, `ElForm`, and `ElDatePicker`.
- Do not use kebab-case template tags for Element Plus components.
- Use PascalCase for layout, page, and reusable component filenames, such as
  `AdminSystemLayout.vue`, `DashboardPage.vue`, `BaseCrudPage.vue`, and
  `QuickActionsCard.vue`.
- Use lowerCamelCase with `use` prefixes for composables, such as `useCrud.js`,
  `usePagination.js`, `useFilters.js`, and `usePermissions.js`.
- Use lowerCamelCase with `.store.js` suffixes for Pinia store filenames, such
  as `auth.store.js`, `app.store.js`, and `permissions.store.js`.
- Use lowercase hyphenated router module filenames, such as
  `admin-system.routes.js` and `schools.routes.js`, even when route names
  themselves use lowerCamelCase or PascalCase constants internally.
- Use singular or narrowly scoped PascalCase names for shared frontend
  contracts and data shapes, such as `School`, `SchoolFormModel`,
  `CrudQueryParams`, and `PaginatedResponse`.
- Define shared frontend contract shapes with JSDoc typedefs in JavaScript
  files, plus service mapping helpers where API envelopes need normalization.
- Use plural service filenames only when they clearly represent a collection
  boundary, such as `schools.js`; otherwise prefer explicit purpose-driven
  service names such as `auth-session.js` or `notifications.js`.
- Use PascalCase names for Element Plus wrappers and shared UI primitives, such
  as `BaseDataTable`, `BaseFilterBar`, `BasePagination`, `BaseConfirmDialog`,
  `BasePageHeader`, and `StatCard`.

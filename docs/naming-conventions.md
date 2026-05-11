# Naming Conventions

## Repository Names

- `schoolmaster-specs`
- `schoolmaster-backend`
- `schoolmaster-frontend`

## Documentation

- Use concise English titles
- Keep file names lowercase with hyphens
- Number architecture decisions with a three-digit prefix

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
  `academic-years.ts` and `school-onboarding.spec.ts`.
- Use feature-aligned module folders, such as `admin`, `teacher`, `student`,
  and `reports`.

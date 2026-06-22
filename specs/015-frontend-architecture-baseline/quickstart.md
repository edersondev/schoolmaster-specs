# Quickstart: Frontend Architecture Baseline

Use this checklist when creating or reviewing future frontend specs and implementation work.

## 1. Confirm Scope

- Start from `specs/015-frontend-architecture-baseline/spec.md`.
- Use `docs/frontend-architecture.md`, `docs/frontend-guidelines.md`, and `docs/naming-conventions.md` as supporting guidance.
- Do not define concrete module behavior unless the consuming feature spec owns it.

## 2. Verify Approved Stack

Future `schoolmaster-frontend` work must use:

- JavaScript
- Vue 3 Composition API with `<script setup>`
- Vue Router
- Pinia
- Axios
- Element Plus
- `@element-plus/icons-vue`
- Vue I18n
- Tailwind CSS

Reject TypeScript requirements, alternate UI libraries, alternate state libraries, and direct component HTTP access unless a later approved architecture change supersedes this baseline.

## 3. Check API-First Readiness

Before frontend behavior consumes backend data, confirm:

- the endpoint is documented under `/api/v1`
- request fields, filters, pagination, status meanings, and error envelopes are documented
- authentication, authorization, and tenant semantics are documented
- the future feature spec links the consumed contract behavior

If any item is missing, block frontend implementation until the specification and OpenAPI contract are updated.

## 4. Check Folder and Boundary Fit

Route-facing views belong in `pages/`.
Reusable shells belong in `layouts/`.
Reusable UI belongs in `components/`.
Shared view coordination belongs in `composables/`.
Shared state belongs in Pinia `stores/`.
HTTP access belongs in `services/`.
Consumed data shapes belong in `contracts/`.
JavaScript contract definitions in `contracts/` should use JSDoc typedefs plus service mapping helpers.

Pages, layouts, and components must not call Axios directly.

## 5. Check UI Conventions

- Element Plus component tags must be PascalCase.
- Element Plus Icons should be the default icon source.
- Tailwind should handle spacing, layout, responsiveness, and restrained refinement.
- Vue I18n should centralize reusable UI text.
- Reusable UI must target WCAG 2.1 AA.

Example allowed template naming:

```vue
<template>
  <ElCard>
    <ElButton>Save</ElButton>
  </ElCard>
</template>
```

Example rejected template naming:

```vue
<template>
  <el-card>
    <el-button>Save</el-button>
  </el-card>
</template>
```

## 6. Check CRUD Reuse

For admin workflows, reuse or extend the baseline patterns for:

- list pages
- filters
- tables
- pagination
- forms
- dialogs
- loading states
- empty states
- error and validation states

Resource-specific rules belong in the consuming feature spec.

## 7. Suggested Review Commands

After implementation exists in `schoolmaster-frontend`, use checks like these from that repository:

```bash
rg "axios" src/pages src/components src/layouts
rg "<el-" src
rg "from ['\"]@element-plus/icons-vue['\"]" src
rg "useI18n|createI18n" src
```

The exact test commands belong to the frontend repository, but affected frontend services, stores, composables, and route workflows should be covered with Vitest where behavior risk justifies it.

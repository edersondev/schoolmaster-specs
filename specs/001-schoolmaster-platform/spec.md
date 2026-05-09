# Feature Specification: SchoolMaster Platform Foundation

**Feature Branch**: `001-schoolmaster-platform`  
**Created**: 2026-05-08  
**Status**: Draft  
**Input**: User description: "Create the initial product specification for the SchoolMaster SaaS platform.

SchoolMaster is a multi-tenant school management SaaS.

The application will be split into separate repositories:
- schoolmaster-specs: product specifications, business rules, architecture decisions and API contracts
- schoolmaster-backend: Laravel API
- schoolmaster-frontend: Vue 3 SPA

The backend must be API-first and the frontend must consume only the REST API.

Main user profiles:
- System administrator
- School administrator
- Teacher
- Student

Initial functional scope:
- Authentication and authorization
- School management
- User management
- Roles and permissions
- Academic year management
- Academic periods inside the academic year
- Student guardian management
- Teacher content management with folders and file uploads
- Questionnaires created by teachers
- Learning sets that organize content and questionnaires in chronological order
- Grades
- Attendance
- Reports

General SaaS requirements:
- Support multiple schools
- Data must be isolated by tenant
- Use UUIDs where applicable
- Use status fields for active/inactive records
- Use RESTful API standards
- API routes must be versioned using /api/v1
- OpenAPI must be the source of truth for communication between frontend and backend
- Business rules must be documented before implementation

The goal of this specification is to define the product vision, main modules, actors, business boundaries, high-level requirements, non-goals and acceptance criteria for the first version of SchoolMaster."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Onboard a school and its staff (Priority: P1)

A system administrator creates and activates a school tenant, then a school
administrator configures the academic structure, creates staff and student user
accounts, and assigns roles needed for school operations.

**Why this priority**: Without tenant onboarding, academic structure, and
role-based access, none of the operational modules can be used safely.

**Independent Test**: This story can be tested independently by creating a new
school, activating it, defining one academic year with periods, creating users
for key roles, and confirming each user sees only the permitted school data.

**Acceptance Scenarios**:

1. **Given** a system administrator has platform access, **When** they register
   a new school and mark it active, **Then** the school is provisioned as its
   own tenant and is available for school administrator setup.
2. **Given** a school administrator belongs to one active school, **When** they
   create users and assign roles, **Then** each user gains only the permissions
   associated with their role inside that school.
3. **Given** a school administrator is preparing an academic cycle, **When**
   they create an academic year and its academic periods, **Then** the school
   can organize students, teachers, attendance, grades, and reports within the
   defined time structure.

---

### User Story 2 - Run day-to-day teaching workflows (Priority: P2)

A teacher organizes instructional content in folders, uploads files, creates
questionnaires, assembles learning sets in chronological order, records
attendance, and submits grades for their students during an academic period.

**Why this priority**: Once a school is onboarded, the next core value is
enabling teachers to manage classroom delivery and student evaluation.

**Independent Test**: This story can be tested independently by having a
teacher add content, publish a learning set, record attendance for a class
session, and enter grades for students assigned to that teacher.

**Acceptance Scenarios**:

1. **Given** a teacher has permission to manage instructional materials,
   **When** they create folders and upload content, **Then** the materials are
   stored within their school context and can be organized for later use.
2. **Given** a teacher needs to structure instruction, **When** they create a
   learning set with content items and questionnaires in chronological order,
   **Then** the learning set preserves the intended sequence for student use.
3. **Given** a teacher is responsible for a class during an active academic
   period, **When** they record attendance and submit grades, **Then** the
   records are stored against the correct students, period, and school.

---

### User Story 3 - View student progress and official outputs (Priority: P3)

A student accesses their assigned learning materials and academic records, while
school administrators review reports covering attendance, grades, academic
structure, and user activity needed for school oversight.

**Why this priority**: Students and administrators need visibility into the
results of the operational workflows, but that depends on onboarding and
teacher activity already being in place.

**Independent Test**: This story can be tested independently by logging in as a
student to review assigned learning sets and grades, and as a school
administrator to generate reports for a selected academic period.

**Acceptance Scenarios**:

1. **Given** a student has access to assigned learning sets, **When** they open
   their learning timeline, **Then** they can see content and questionnaires in
   the sequence defined by their teacher.
2. **Given** a student has attendance and grade records, **When** they review
   their academic status, **Then** they can see only their own records within
   the relevant academic period.
3. **Given** a school administrator needs school oversight, **When** they
   generate reports, **Then** the outputs summarize the selected school's
   attendance, grades, and academic activity without exposing other schools'
   data.

### Edge Cases

- What happens when a user account is inactive but still assigned to a role or
  school relationship?
- How does the system handle a teacher attempting to record attendance or
  grades outside the active academic period?
- What happens when a student changes schools or is no longer active in the
  current academic year?
- How does the platform prevent a school administrator from seeing or modifying
  another school's users, reports, content, or academic structure?
- What happens when uploaded instructional files are unsupported, exceed
  allowed limits, or are attached to an inactive content item?
- How does the system handle a learning set whose referenced content or
  questionnaire becomes inactive after publication?

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Introduce the initial tenant-aware business
  capabilities for authentication, schools, users, roles, academic structure,
  guardians, teacher content, questionnaires, learning sets, grades,
  attendance, and reports.
- **Frontend repository impact**: Introduce the initial user journeys for
  sign-in, tenant-aware administration, teacher workflows, student views, and
  reporting screens that consume only the published product contracts.
- **Specification or contract repository impact**: Define the product vision,
  business rules, module boundaries, actor permissions, and initial OpenAPI
  contracts that govern backend and frontend delivery.
- **Delivery ownership and sequencing**: The specification repository leads
  first with business rules and contracts, the backend follows with contract-
  compliant capabilities, and the frontend follows by consuming the approved
  contracts for each delivered module.

### API Contract Impact

- **OpenAPI update required**: Yes. Initial contracts must be defined in the
  specification repository before backend or frontend implementation begins.
- **Versioned endpoints affected**: `/api/v1/auth`, `/api/v1/schools`,
  `/api/v1/users`, `/api/v1/roles`, `/api/v1/academic-years`,
  `/api/v1/academic-periods`, `/api/v1/guardians`,
  `/api/v1/teacher-content`, `/api/v1/questionnaires`,
  `/api/v1/learning-sets`, `/api/v1/grades`, `/api/v1/attendance`,
  `/api/v1/reports`
- **JSON response impact**: All product modules require a consistent response
  pattern for successful reads and writes, validation failures, permission
  denials, inactive-record handling, and tenant isolation errors.
- **Authentication/authorization impact**: Authentication establishes the user
  identity and school context, while authorization differentiates system-wide
  privileges from school-scoped privileges for administrators, teachers, and
  students.
- **Compatibility impact**: This is an initial greenfield contract set. Changes
  made after publication are expected to be additive unless an approved version
  or migration plan states otherwise.

### Data & Tenancy Impact

- **Tenant scoping impact**: Each school operates as an isolated tenant for its
  users, academic structures, guardians, content, questionnaires, learning
  sets, grades, attendance, and reports.
- **Tenant resolution impact**: Every authenticated school-scoped request must
  resolve an explicit tenant context and reject access when the tenant is
  missing, mismatched, or inactive.
- **Cross-tenant or platform access impact**: Only system administrators may
  perform platform-wide operations such as provisioning schools or reviewing
  platform-level status; all other roles remain limited to their own school.
- **Platform override impact**: Any cross-tenant operation available to a
  platform-scope user must be explicitly documented in policy and contract
  definitions and covered by authorization and regression tests.
- **Soft delete impact**: School-owned operational records and administrative
  reference records are expected to support reversible deactivation or removal
  where recovery is relevant to school operations and audits.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST support multiple schools as separate tenants within a
  shared SaaS platform.
- **FR-002**: System MUST authenticate users before granting access to any
  protected function.
- **FR-003**: System MUST authorize actions based on role and tenant context.
- **FR-004**: System MUST support the user profiles system administrator,
  school administrator, teacher, and student.
- **FR-005**: System MUST allow a system administrator to create, activate,
  deactivate, and review schools.
- **FR-006**: System MUST allow a school administrator to manage school profile
  information and operational status for their own school.
- **FR-007**: System MUST allow authorized administrators to create, activate,
  deactivate, and maintain user accounts within their permitted tenant scope.
- **FR-008**: System MUST allow authorized administrators to assign roles and
  permissions to users within their permitted tenant scope.
- **FR-009**: System MUST allow a school administrator to create and maintain
  academic years for their school.
- **FR-010**: System MUST allow a school administrator to define academic
  periods inside each academic year.
- **FR-011**: System MUST maintain guardian records for students and preserve
  the association between students and their guardians.
- **FR-012**: System MUST allow teachers to create folders and manage uploaded
  instructional content for their school.
- **FR-013**: System MUST allow teachers to create and maintain questionnaires
  for instructional use.
- **FR-014**: System MUST allow teachers to create learning sets that organize
  content and questionnaires in chronological order.
- **FR-015**: System MUST allow authorized users to record and review student
  grades within the relevant academic period.
- **FR-016**: System MUST allow authorized users to record and review student
  attendance within the relevant academic period.
- **FR-017**: System MUST provide reports for school administrators covering at
  least attendance, grades, academic structure, and school activity summaries.
- **FR-018**: Students MUST be able to view their assigned learning sets,
  grades, and attendance records for their own school context.
- **FR-019**: System MUST preserve tenant isolation so that school-scoped users
  cannot view or modify another school's data.
- **FR-019a**: System MUST resolve tenant context explicitly for every
  authenticated school-scoped request and deny access when tenant context is
  missing, mismatched, or inactive.
- **FR-020**: System MUST use UUID-based identifiers for entities that are
  exchanged across product boundaries or external references.
- **FR-021**: System MUST maintain active or inactive status for schools,
  users, academic structures, and other applicable operational records.
- **FR-021a**: System MUST use soft deletes for recoverable tenant-owned
  operational records unless a permanent deletion path is explicitly approved
  and documented.
- **FR-022**: System MUST define REST communication contracts in OpenAPI before
  implementation begins.
- **FR-023**: System MUST expose product routes through versioned `/api/v1`
  endpoints.
- **FR-024**: System MUST preserve a consistent response structure for
  successful outcomes and failure outcomes across product modules.
- **FR-024a**: System MUST document authentication, authorization, pagination,
  filtering, sorting, tenancy semantics, and standard error responses in
  OpenAPI for every affected endpoint.
- **FR-025**: System MUST document business rules for each module before
  implementation starts in the backend or frontend repositories.
- **FR-026**: System MUST identify affected repositories and delivery sequence
  whenever a feature spans specifications, backend, and frontend work.
- **FR-027**: System MUST define how permissions are assigned, inherited, and
  exposed through roles for platform and school scopes.
- **FR-028**: System MUST treat system administrator, school administrator,
  teacher, and student as actor profiles expressed through scoped roles and,
  where needed, related domain profiles such as `StudentProfile`.
- **FR-029**: System MUST validate, sanitize, and store uploaded instructional
  files in tenant-scoped private storage with enforced type and size
  restrictions.

### Key Entities *(include if feature involves data)*

- **School**: Represents a tenant school, including its identity, operational
  status, and lifecycle within the platform.
- **User**: Represents a person with platform access, including system
  administrators, school administrators, teachers, and students.
- **Role**: Represents a named authorization profile that determines which
  actions a user may perform in platform or school scope.
- **Permission**: Represents an individual allowed capability that may be
  assigned through a role.
- **Academic Year**: Represents the primary academic cycle used to group school
  operations.
- **Academic Period**: Represents a sub-division inside an academic year used
  for scheduling, attendance, grades, and reporting.
- **Student**: Represents a learner profile linked to a user and enrolled in a
  school context.
- **Guardian**: Represents a responsible contact linked to one or more
  students.
- **Teacher Content Item**: Represents an uploaded or authored learning asset
  managed by a teacher.
- **Content Folder**: Represents an organizational container for teacher
  content.
- **Questionnaire**: Represents an assessment or activity created by a teacher.
- **Learning Set**: Represents a chronological sequence of content items and
  questionnaires for student consumption.
- **Grade Record**: Represents an academic evaluation result associated with a
  student, teacher, and academic period.
- **Attendance Record**: Represents a student's presence status for a specific
  instructional event or date.
- **Report Definition or Output**: Represents a generated summary or formal
  view of school information for oversight.

## Non-Goals

- The first version does not include payroll, billing, accounting, or financial
  management features.
- The first version does not include parent self-service portals as a primary
  actor surface beyond guardian record management by school staff.
- The first version does not include live classroom delivery, messaging,
  notifications, or video conferencing workflows.
- The first version does not include advanced curriculum planning, timetable
  optimization, or national compliance reporting beyond the core school reports
  defined for launch.
- The first version does not include direct third-party integrations unless
  they are needed to support the defined core modules.

## Actor Profile Model

- `System administrator` is a platform-scope actor profile with explicit
  authority to provision and review school tenants and no implicit bypass of
  module-specific authorization rules.
- `School administrator`, `teacher`, and `student` are school-scoped actor
  profiles expressed through tenant-bound roles and permissions.
- `StudentProfile` is a domain profile linked to the `User` actor identity for
  student-specific enrollment and academic operations.
- For v1, permissions are managed through roles rather than direct per-user
  permission assignment.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A new school can be onboarded, activated, and configured with its
  first academic year, periods, and core user roles in under 60 minutes by an
  authorized administrator.
- **SC-002**: At least 95% of school administration tasks in the launch scope
  can be completed without assistance by a trained school administrator during
  acceptance testing.
- **SC-003**: Teachers can complete the end-to-end workflow of uploading
  content, assembling a learning set, recording attendance, and entering grades
  for a class during acceptance testing without switching to out-of-system
  tools.
- **SC-004**: Students can access their assigned learning materials and own
  academic records with a first-attempt task completion rate of at least 90%
  during usability validation.
- **SC-005**: Cross-tenant access attempts by school-scoped users are rejected
  in 100% of defined acceptance tests.
- **SC-006**: School administrators can generate the launch-scope reports for a
  selected academic period in 95% of acceptance test scenarios without manual
  data preparation outside the platform.

## Assumptions

- SchoolMaster v1 is intended for web-based administrative and academic
  operations performed by authenticated users with stable internet access.
- Each operational user belongs to exactly one primary school context for the
  first version, except system administrators who operate at platform scope.
- Student and guardian data are managed by school staff rather than through a
  self-service guardian portal in the first version.
- Business rule details for grading methods, attendance states, and report
  layouts will be documented as module-level follow-up specifications before
  implementation of those modules begins.
- Upload security rules, including allowed file types, maximum upload size,
  storage visibility, and validation or sanitization outcomes, will be defined
  before teacher content implementation begins.
- The product will be delivered incrementally across the specification,
  backend, and frontend repositories, with contracts and business rules defined
  before implementation work starts in the dependent repositories.

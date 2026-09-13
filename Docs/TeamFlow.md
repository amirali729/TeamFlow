# TeamFlow — Complete Requirements Document

**Version:** 1.0
**Status:** MVP Requirements Baseline

---

# 1. Product Overview

## 1.1 Product Name

**TeamFlow**

## 1.2 Product Type

TeamFlow is a SaaS team project and work-management platform for companies and teams.

## 1.3 Core Purpose

TeamFlow provides a single source of truth for:

* Team organization
* Workspace membership
* Project ownership
* Project membership
* Task assignment
* Task progress
* Project progress

The system allows a company/team to organize employees into workspaces, create projects, assign project managers and members, create tasks, assign tasks to employees, and track work progress.

## 1.4 Core Workflow

```text
User creates account
        ↓
Email verification
        ↓
Create workspace
        ↓
Add workspace members
        ↓
Create project
        ↓
Choose Project Manager
        ↓
Choose Project Members
        ↓
Create tasks
        ↓
Assign tasks
        ↓
Members work on tasks
        ↓
Track task/project progress
```

---

# 2. MVP Scope

## 2.1 Included

The MVP includes:

* User registration
* Login
* Logout
* Email verification
* Access-token authentication
* Refresh-token authentication
* Refresh sessions
* Workspaces
* Workspace membership
* Workspace roles
* Projects
* Project membership
* Project roles
* Tasks
* Task assignment
* Task priorities
* Task status management
* Project status management
* Manual project progress
* Soft deletion/archival where defined
* Authorization
* Input validation
* Consistent API responses
* Error handling
* Database constraints and indexes
* Basic security controls

## 2.2 Explicitly Excluded

The MVP does **not** include:

* Real-time chat
* WebSockets
* Comments
* File attachments
* Advanced notifications
* Background job queues
* Redis
* RabbitMQ
* Kafka
* Mobile application
* AI features
* Advanced analytics
* Advanced reporting
* Microservices
* Kubernetes
* Event sourcing
* CQRS
* Hard deletion of tasks
* Permanent user deletion
* Complex notification infrastructure

These features may be considered in future versions only when a real product or engineering requirement justifies them.

---

# 3. Users and Authentication

## 3.1 User Registration

A user can create an account using:

* Username
* Email
* Password

Passwords must never be stored in plaintext.

Passwords must be securely hashed.

## 3.2 Email Verification

A newly registered user must verify their email before creating a workspace.

Flow:

```text
Signup
  ↓
Create user
  ↓
Generate verification token
  ↓
Send verification email
  ↓
User opens verification link
  ↓
Validate token
  ↓
Mark email as verified
```

Verification tokens must:

* Be securely generated
* Be stored hashed
* Have an expiration time
* Be single-use
* Not be exposed in server logs

Expired tokens must be rejected.

A user should be able to request another verification email subject to appropriate rate limiting.

## 3.3 Login

A user can log in using their credentials.

Successful authentication produces:

* Short-lived access token
* Long-lived refresh token

The refresh token must be stored in an HTTP-only cookie.

The refresh token itself must not be stored directly in the database.

Only its secure hash is stored.

## 3.4 Refresh Sessions

Each refresh token corresponds to a server-side refresh session.

A user may have multiple active sessions, for example:

```text
User
├── Browser session
├── Laptop session
└── Mobile session
```

A refresh session contains:

* Session ID
* User ID
* Hashed refresh token
* Expiration time
* Revocation time
* Creation time
* Updated time

Expired or revoked sessions must not be accepted.

## 3.5 Logout

Logout must:

1. Revoke the refresh session.
2. Clear the refresh-token cookie.

The refresh token must no longer be usable after logout.

---

# 4. Workspaces

## 4.1 Workspace

A workspace represents a company or team.

A workspace contains:

* Workspace information
* Workspace members
* Projects

A workspace cannot be deleted in the MVP.

## 4.2 Workspace Membership

Users belong to workspaces through a membership relationship.

A user may belong to multiple workspaces.

A workspace may contain multiple users.

The membership contains:

* User
* Workspace
* Role
* Join timestamp
* Removal timestamp

A user cannot have multiple active memberships in the same workspace.

Historical membership records should be preserved when appropriate.

---

# 5. Workspace Roles

There are three workspace roles:

```text
OWNER
ADMIN
MEMBER
```

## 5.1 Owner

The Owner has full workspace control.

The Owner can:

* Manage workspace members
* Add members
* Remove members
* Promote members to Admin
* Demote Admins
* Create projects
* Choose Project Managers
* Manage project-level decisions
* Delete/archive projects according to the project lifecycle rules
* Manage project status/progress
* Manage tasks
* Assign/reassign tasks
* Drop tasks

A workspace must have exactly one active Owner.

Ownership transfer will be required before the current Owner leaves the workspace.

## 5.2 Admin

An Admin has operational workspace-management capabilities.

An Admin can:

* Add workspace members
* Create projects
* Edit task details
* Assign tasks
* Reassign tasks
* Change task status where permitted

An Admin cannot:

* Remove workspace members
* Promote a member to Admin
* Remove an Admin
* Demote an Admin
* Delete/archive projects
* Change the Project Manager

## 5.3 Member

A normal workspace Member can:

* View the workspace
* View projects they are allowed to see
* Participate in projects they are assigned to

A normal workspace Member cannot:

* Manage workspace membership
* Add workspace members
* Remove workspace members
* Promote/demote users
* Manage projects
* Assign tasks
* Reassign tasks
* Drop tasks

A workspace Member does not automatically receive visibility into all project members/managers.

---

# 6. Projects

## 6.1 Project

A project belongs to exactly one workspace.

A workspace can contain multiple projects.

A project contains:

* Name
* Description
* Project Manager
* Project Members
* Tasks
* Status
* Manual progress
* Creator
* Timestamps

## 6.2 Project Roles

There are two project roles:

```text
MANAGER
MEMBER
```

There is no separate Project Admin role in the MVP.

## 6.3 Project Manager

The Project Manager is selected by the Owner.

A project has exactly one active Project Manager.

A user may be Project Manager of multiple projects.

The Project Manager can:

* Manage project members
* Add project members
* Remove project members
* Assign tasks
* Reassign tasks
* Edit task details
* Change task status
* Drop tasks
* Edit project details
* Change project status from `IN_PROGRESS` to `COMPLETED`
* Manage project progress

The Project Manager cannot:

* Change the Project Manager
* Perform Owner-only workspace operations

## 6.4 Project Member

A Project Member can:

* View project details
* View the Project Manager
* View project members
* View tasks
* View task statuses
* View project status
* View project progress

A Project Member can only change the status of their own assigned task.

Allowed task status flow for a Project Member:

```text
PENDING
   ↓
IN_PROGRESS
   ↓
COMPLETED
```

A Project Member cannot:

* Assign tasks
* Reassign tasks
* Edit task details
* Drop tasks
* Change project details
* Change project status
* Change project progress
* Manage project membership

---

# 7. Project Visibility

Workspace membership does not automatically mean full project membership visibility.

A user who is not a member of a project should not receive internal project-member information unless their workspace-level permissions explicitly allow the operation.

Once a user becomes a Project Member, they can see:

* Project details
* Project Manager
* Project Members
* Project Tasks
* Task statuses
* Project status
* Project progress

Authorization must be enforced server-side.

---

# 8. Project Status

Project status values:

```text
NOT_STARTED
PENDING
IN_PROGRESS
COMPLETED
ABANDONED
```

Project status transitions are controlled by role.

The Owner has full project-status control.

The Project Manager can change:

```text
IN_PROGRESS → COMPLETED
```

The complete transition matrix must be finalized before implementation of project-status workflows.

`ABANDONED` is intended to represent an abandoned project; whether it is permanently terminal must be finalized during detailed business-rule design.

---

# 9. Project Progress

Project progress is manually controlled.

It is **not automatically calculated from task completion**.

Owner and Project Manager can control project progress.

Progress should represent a value from:

```text
0–100
```

The database should enforce the valid numeric range.

---

# 10. Tasks

## 10.1 Task

A task belongs to exactly one project.

A task contains:

* Title
* Description
* Assigned member
* Assigned by
* Status
* Priority
* Due date
* Creator
* Timestamps
* Dropped timestamp

## 10.2 Task Status

Task status values:

```text
NOT_ASSIGNED
PENDING
IN_PROGRESS
COMPLETED
DROPPED
```

## 10.3 Task Priority

Task priority values:

```text
LOW
MEDIUM
HIGH
URGENT
```

## 10.4 Task Assignment

A task can initially be created without an assignee.

When no member is assigned:

```text
status = NOT_ASSIGNED
```

When a member is assigned:

```text
status = PENDING
```

A task cannot be assigned to a user who is not a member of the project.

This rule is enforced at the application/domain level.

## 10.5 Assigned By

The system must preserve who assigned the task.

Therefore:

```text
assigned_to
assigned_by
```

are separate relationships.

Example:

```text
Manager
   ↓
assigns task
   ↓
Employee
```

The task records both:

```text
assigned_by = Manager
assigned_to = Employee
```

## 10.6 Task Status Permissions

Owner:

* Full task-status control.

Project Manager:

* Full task-status control subject to project rules.

Admin:

* Can change task status where authorized by workspace/project permissions.

Project Member:

* Can only change their own assigned task status.

Allowed Member flow:

```text
PENDING
   ↓
IN_PROGRESS
   ↓
COMPLETED
```

## 10.7 Dropping Tasks

Tasks are not hard-deleted in the MVP.

"Delete task" means:

```text
status = DROPPED
```

The task remains in the database.

Owner and Project Manager can drop tasks.

Admin and Project Member cannot drop tasks.

Dropped tasks preserve their historical information.

---

# 11. Member Removal and Tasks

If a project member is removed while they have assigned tasks:

```text
assigned_to = NULL
status = NOT_ASSIGNED
```

The task itself remains.

Historical information such as:

```text
created_by
assigned_by
```

must remain intact.

The operation should be performed transactionally so membership removal and task unassignment do not leave inconsistent state.

---

# 12. Due Dates

Every task has a required due date.

If the due date passes while the task is not:

```text
COMPLETED
```

or:

```text
DROPPED
```

the task is considered overdue.

The MVP does not require automatic overdue notifications.

---

# 13. Database Entities

The MVP database contains these primary entities:

```text
User
Workspace
WorkspaceMember

Project
ProjectMember

Task

RefreshSession
EmailVerification
```

---

# 14. Database Relationships

## User → Workspace

Many-to-many through `WorkspaceMember`.

```text
User N ─── WorkspaceMember ─── N Workspace
```

## Workspace → Project

One-to-many.

```text
Workspace 1 ─── N Project
```

## User → Project

Many-to-many through `ProjectMember`.

```text
User N ─── ProjectMember ─── N Project
```

## Project → Task

One-to-many.

```text
Project 1 ─── N Task
```

## User → Task

A user may:

* Create many tasks
* Assign many tasks
* Be assigned many tasks

A task can have zero or one current assignee.

## User → RefreshSession

One-to-many.

```text
User 1 ─── N RefreshSession
```

## User → EmailVerification

One-to-many.

```text
User 1 ─── N EmailVerification
```

---

# 15. Database Fields

## 15.1 Users

```text
id
username
email
password_hash
email_verified_at
created_at
updated_at
deleted_at
```

## 15.2 Workspaces

```text
id
name
created_by
created_at
updated_at
```

A workspace is not deletable in the MVP.

## 15.3 Workspace Members

```text
id
workspace_id
user_id
role
joined_at
removed_at
```

## 15.4 Projects

```text
id
workspace_id
name
description
status
progress
created_by
created_at
updated_at
deleted_at
```

## 15.5 Project Members

```text
id
project_id
user_id
role
joined_at
removed_at
```

## 15.6 Tasks

```text
id
project_id
title
description
assigned_to
assigned_by
status
priority
due_date
created_by
created_at
updated_at
dropped_at
```

## 15.7 Refresh Sessions

```text
id
user_id
token_hash
expires_at
revoked_at
created_at
updated_at
```

## 15.8 Email Verifications

```text
id
user_id
token_hash
expires_at
used_at
created_at
```

---

# 16. Database Constraints

## Users

```text
PRIMARY KEY (id)

UNIQUE (username)

UNIQUE (email)
```

## Workspace Members

```text
PRIMARY KEY (id)

FOREIGN KEY (workspace_id)
FOREIGN KEY (user_id)
```

Active membership must be unique:

```text
UNIQUE(workspace_id, user_id)
WHERE removed_at IS NULL
```

## Projects

```text
PRIMARY KEY (id)

FOREIGN KEY (workspace_id)

FOREIGN KEY (created_by)
```

Project progress:

```text
0 <= progress <= 100
```

## Project Members

```text
PRIMARY KEY (id)

FOREIGN KEY (project_id)

FOREIGN KEY (user_id)
```

Active membership:

```text
UNIQUE(project_id, user_id)
WHERE removed_at IS NULL
```

## Tasks

```text
PRIMARY KEY (id)

FOREIGN KEY (project_id)

FOREIGN KEY (assigned_to)

FOREIGN KEY (assigned_by)

FOREIGN KEY (created_by)
```

`assigned_to` is nullable.

## Refresh Sessions

```text
PRIMARY KEY (id)

UNIQUE(token_hash)

FOREIGN KEY (user_id)
```

## Email Verifications

```text
PRIMARY KEY (id)

UNIQUE(token_hash)

FOREIGN KEY (user_id)
```

---

# 17. Database Indexes

## Users

Unique indexes on:

```text
username
email
```

## Workspace Members

Indexes:

```text
workspace_id
user_id
(workspace_id, user_id) WHERE removed_at IS NULL
```

## Projects

Index:

```text
workspace_id
```

## Project Members

Indexes:

```text
project_id
user_id
(project_id, user_id) WHERE removed_at IS NULL
```

## Tasks

Initial indexes:

```text
project_id
assigned_to
```

Composite indexes should only be added when actual query patterns justify them.

Potential future indexes include:

```text
(project_id, status)
(assigned_to, status)
```

## Refresh Sessions

Indexes:

```text
token_hash
user_id
```

`token_hash` is already unique.

## Email Verifications

Indexes:

```text
token_hash
user_id
```

`token_hash` is already unique.

---

# 18. Business Rules vs Database Rules

The database is responsible primarily for **data integrity**.

The domain/application is responsible for **business behavior**.

## Database responsibilities

Examples:

* Primary keys
* Foreign keys
* Uniqueness
* Valid progress range
* Required fields
* Referential integrity

## Domain/Application responsibilities

Examples:

* A user must verify email before creating a workspace.
* Only Owner can promote an Admin.
* Admin cannot remove an Owner.
* A task assignee must belong to the project.
* Only one Project Manager is allowed.
* Project Members can only update their own tasks.
* Project status transitions.
* Task status transitions.
* Removing a project member unassigns their active tasks.
* Ownership transfer.
* Project Manager replacement.

---

# 19. API Response Standards

## Success

All successful API responses should follow:

```json
{
  "statusCode": 200,
  "message": "Success",
  "data": {}
}
```

The actual HTTP status code must correspond to the operation.

Examples:

```text
201 → resource created
200 → successful operation
204 → successful operation with no response body, if appropriate
```

## Error

Errors should follow:

```json
{
  "statusCode": 400,
  "code": "SOME_ERROR_CODE",
  "message": "Human-readable message"
}
```

Internal implementation details must not be exposed.

The API must not expose:

* Stack traces
* Database internals
* Secrets
* Password hashes
* Refresh-token hashes
* Sensitive infrastructure information

---

# 20. Authorization and Security

The backend is the authoritative security boundary.

The frontend must never be trusted to enforce authorization.

Security requirements include:

* Password hashing
* Secure authentication
* HTTP-only refresh cookie
* Secure cookie configuration in production
* Server-side authorization
* Input validation
* Protected endpoints
* Workspace isolation
* Project isolation
* Appropriate rate limiting
* No sensitive information in logs
* Secure secret management
* Expiration of authentication tokens
* Refresh-session revocation
* Verification-token expiration
* Verification-token single use

Unauthorized resource access should not unnecessarily reveal whether a protected resource exists.

---

# 21. Architecture Requirements

## Repository

TeamFlow uses a monorepo.

```text
teamflow/
├── apps/
│   ├── web/
│   └── api/
│
└── packages/
    ├── domain/
    ├── database/
    └── shared/
```

## Frontend

```text
Next.js
```

## Backend

```text
NestJS
```

The NestJS framework exists only inside:

```text
apps/api
```

## Backend Architecture

The backend follows Clean Architecture principles.

The major concerns are:

```text
Application
Infrastructure
Domain
```

The domain is independent of technology.

## Domain Package

```text
packages/domain
```

contains the business core.

It must not depend on:

* NestJS
* PostgreSQL
* Kysely
* Resend
* HTTP
* Next.js
* Express

Possible domain areas:

```text
domain/
├── user/
├── workspace/
├── project/
└── task/
```

## Database Package

```text
packages/database
```

contains database-level concerns.

It represents persistence models and Kysely/PostgreSQL infrastructure.

The database model is allowed to differ from the domain entity.

## Shared Package

```text
packages/shared
```

contains only definitions genuinely required by both frontend and backend.

Examples:

```text
Types
Constants
Shared enums
```

It must not contain:

* Business rules
* Domain entities
* Repository implementations
* Kysely code
* Database models
* NestJS modules
* Backend use cases

---

# 22. Technology Stack

The planned MVP stack is:

| Concern              | Technology                            |
| -------------------- | ------------------------------------- |
| Repository           | Monorepo                              |
| Frontend             | Next.js                               |
| Backend              | NestJS                                |
| Backend architecture | Clean Architecture / Modular Monolith |
| Database             | PostgreSQL                            |
| Query builder        | Kysely                                |
| Email                | Resend                                |
| Shared definitions   | TypeScript package                    |
| Domain               | Technology-independent TypeScript     |

---

# 23. Email

Email functionality is centralized behind an application-level email service.

Authentication code should not directly call the Resend API everywhere.

Conceptually:

```text
Authentication
      ↓
Email Service
      ↓
Resend
```

The email provider is an infrastructure concern.

---

# 24. Reliability Requirements

The system should:

* Return predictable errors.
* Maintain consistent response formats.
* Avoid partial database operations.
* Use transactions for operations requiring multiple related database changes.
* Preserve important history.
* Correctly handle expired sessions.
* Correctly handle revoked sessions.
* Avoid inconsistent membership/task state.
* Maintain workspace and project isolation.

---

# 25. Auditability

The system should preserve enough information to answer:

* Who created this task?
* Who assigned this task?
* Who is currently assigned?
* Who manages this project?
* When did a user join?
* When was a member removed?
* When was a task dropped?
* When was a session created/revoked?

Full event sourcing is not required.

The MVP uses normal relational data and timestamps to provide sufficient accountability.

---

# 26. Important Edge Cases

The following rules require explicit handling:

## Owner leaves

Ownership must be transferred before the Owner can leave.

## Project Manager leaves

A replacement Project Manager must be selected before the PM can leave the project.

## Project Member leaves

Their active assigned tasks become:

```text
assigned_to = NULL
status = NOT_ASSIGNED
```

Task history remains.

## Due date passes

The task becomes logically overdue if it is not completed or dropped.

No automatic notification is required in MVP.

## Project is abandoned

Tasks and project history remain.

## Email verification expires

The verification attempt is rejected.

The user can request a new verification email subject to rate limiting.

## Refresh session expires

The session cannot be used to obtain a new access token.

## Refresh session is revoked

The session cannot be used again.

## Unauthorized resource access

The API must not leak protected resource existence unnecessarily.

---

# 27. Deferred Decisions

The following are intentionally not finalized yet and should be resolved before implementing the relevant feature:

* Complete project status transition matrix
* Whether `ABANDONED` is permanently terminal
* Exact project progress update rules
* Exact workspace invitation/member-add workflow
* Multiple-workspace behavior
* Owner transfer workflow details
* Project Manager replacement workflow details
* Exact behavior of Admin-created projects
* Project membership authority between Owner/Admin and Project Manager
* Exact email verification token format
* Verification token expiration duration
* Refresh-token rotation strategy
* Exact authentication token expiration durations
* PostgreSQL UUID vs BIGINT identity strategy
* Exact PostgreSQL enum strategy
* Exact migration strategy
* Detailed API contracts
* Detailed database naming conventions

---

# 28. MVP Success Criteria

The MVP is considered functionally successful when a verified user can complete the entire core workflow:

```text
Register
   ↓
Verify email
   ↓
Login
   ↓
Create workspace
   ↓
Add employees
   ↓
Create project
   ↓
Choose Project Manager
   ↓
Add Project Members
   ↓
Create task
   ↓
Assign task
   ↓
Member works on task
   ↓
Task reaches Completed
   ↓
Owner/PM tracks project status and progress
```

The system must enforce authorization at every step and maintain consistent relational data.

---

# 29. Future Evolution

After the MVP is stable, future capabilities may include:

```text
Email notifications
      ↓
Background jobs
      ↓
Redis
      ↓
WebSockets / real-time features
      ↓
Comments / chat
      ↓
File attachments
      ↓
React/mobile clients
      ↓
AI features
      ↓
Advanced analytics
```

Infrastructure should be introduced because a real requirement exists, not merely because it is common in production systems.

Microservices should only be considered when the modular monolith develops a concrete scaling, deployment, organizational, or operational requirement that justifies the additional complexity.

---

# 30. Final MVP Model

```text
                         USER
                          │
             ┌────────────┼────────────┐
             │            │            │
             ▼            ▼            ▼
       WORKSPACE      REFRESH       EMAIL
       MEMBERS        SESSIONS      VERIFICATIONS
             │
             ▼
         WORKSPACE
             │
             ▼
          PROJECT
             │
        ┌────┴─────┐
        ▼          ▼
 PROJECT MEMBERS  TASKS
                     │
             ┌───────┼────────┐
             ▼       ▼        ▼
          CREATED  ASSIGNED  ASSIGNED
            BY       TO        BY
             │       │          │
             └───────┴──────────┘
                     │
                     ▼
                    USER
```

## Architectural Principle

> **The domain expresses what TeamFlow means and what rules TeamFlow follows. The application coordinates use cases. Infrastructure connects the system to technology. The database preserves data integrity and persistence. The frontend presents the system to users.**

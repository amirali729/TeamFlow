# TeamFlow — MVP API Documentation

**Version:** 1.0
**Status:** MVP API Contract
**Base Path:** `/api/v1`

---

# 1. API Overview

TeamFlow is a multi-tenant team project-management API.

The API manages:

* Authentication
* Users
* Workspaces
* Workspace members
* Projects
* Project members
* Tasks
* Refresh sessions
* Email verification

---

# 2. Base URL

```text
/api/v1
```

Example:

```text
POST /api/v1/auth/login
```

---

# 3. Authentication

Protected endpoints require an access token:

```http
Authorization: Bearer <accessToken>
```

The access token identifies the authenticated user.

The client must **not** send the authenticated user's ID when the API can obtain it from the access token.

For example, this is incorrect:

```json
{
  "userId": 15,
  "name": "New Project"
}
```

The API obtains the user identity from the authenticated request.

---

# 4. Authentication Model

TeamFlow uses:

```text
Access Token
+
Refresh Token
+
Refresh Session
```

The refresh token is stored in an HTTP-only cookie:

```text
refresh_token
```

The refresh token is not returned in the normal response body.

The database stores only a hash of the refresh token.

---

# 5. Common Response Structure

## Success

```json
{
  "statusCode": 200,
  "code": "OPERATION_SUCCESSFUL",
  "message": "Operation completed successfully",
  "data": {}
}
```

`data` can be:

* Object
* Array
* `null`

## Error

```json
{
  "statusCode": 400,
  "code": "VALIDATION_ERROR",
  "message": "Invalid request"
}
```

Internal implementation details must never be exposed.

---

# 6. HTTP Status Codes

The API uses standard HTTP status codes.

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Entity
429 Too Many Requests
500 Internal Server Error
```

---

# 7. Authentication Routes

## 7.1 Sign Up

Creates a new user account.

### Endpoint

```http
POST /api/v1/auth/signup
```

### Authentication

Not required.

### Request Body

```json
{
  "username": "amir",
  "email": "amir@example.com",
  "password": "Password123!"
}
```

### Response — `201 Created`

```json
{
  "statusCode": 201,
  "code": "USER_CREATED",
  "message": "Account created successfully",
  "data": {
    "id": 1,
    "username": "amir",
    "email": "amir@example.com",
    "emailVerified": false,
    "createdAt": "2026-09-13T10:00:00.000Z"
  }
}
```

### Rules

* Username must be unique.
* Email must be unique.
* Password must satisfy password-validation requirements.
* Password must be hashed before storage.
* The user is unverified after signup.

---

# 8. Email Verification

## 8.1 Send Verification Email

Requests a new verification email.

### Endpoint

```http
POST /api/v1/auth/email-verification/send
```

### Authentication

Required.

### Request Body

No body.

The authenticated user is obtained from the access token.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "VERIFICATION_EMAIL_SENT",
  "message": "Verification email sent successfully",
  "data": null
}
```

### Rules

* Already verified users should not receive another verification email.
* Verification requests should be rate limited.
* The verification token must expire.

---

# 9. Verify Email

## 9.1 Verify Email

Consumes an email verification token.

### Endpoint

```http
POST /api/v1/auth/email-verification/verify
```

### Authentication

Not required.

The verification token authenticates the operation.

### Request Body

```json
{
  "token": "verification-token"
}
```

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "EMAIL_VERIFIED",
  "message": "Email verified successfully",
  "data": null
}
```

### Rules

* Token must exist.
* Token must not be expired.
* Token must not already be used.
* Token must be securely stored as a hash.
* Token becomes unusable after successful verification.

---

# 10. Login

## 10.1 Login

Authenticates an existing user.

### Endpoint

```http
POST /api/v1/auth/login
```

### Authentication

Not required.

### Request Body

```json
{
  "email": "amir@example.com",
  "password": "Password123!"
}
```

### Response — `200 OK`

The response body contains the access token.

The refresh token is issued through an HTTP-only cookie.

```json
{
  "statusCode": 200,
  "code": "LOGIN_SUCCESSFUL",
  "message": "Login successful",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs..."
  }
}
```

### Response Cookie

```text
Set-Cookie: refresh_token=<refresh-token>; HttpOnly; ...
```

The exact `Secure`, `SameSite`, expiration and domain configuration depends on the environment.

---

# 11. Refresh Access Token

Obtains a new access token using the refresh-token cookie.

### Endpoint

```http
POST /api/v1/auth/refresh
```

### Authentication

Access token not required.

Refresh-token cookie required.

### Request Body

No body.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "TOKEN_REFRESHED",
  "message": "Access token refreshed successfully",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs..."
  }
}
```

### Rules

* Refresh token must exist.
* Session must exist.
* Session must not be revoked.
* Session must not be expired.
* Token must match the stored token hash according to the application's token-verification strategy.

---

# 12. Logout

## 12.1 Logout

Revokes the current refresh session and clears the refresh-token cookie.

### Endpoint

```http
POST /api/v1/auth/logout
```

### Authentication

Refresh-token cookie required.

Access token is not required for the logout operation itself.

### Request Body

No body.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "LOGOUT_SUCCESSFUL",
  "message": "Logout successful",
  "data": null
}
```

### Server Behavior

```text
Receive refresh token
        ↓
Find corresponding session
        ↓
Revoke session
        ↓
Clear refresh_token cookie
```

---

# 13. User Routes

## 13.1 Get Current User

Returns the authenticated user's profile.

### Endpoint

```http
GET /api/v1/users/me
```

### Authentication

Required.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "USER_RETRIEVED",
  "message": "User retrieved successfully",
  "data": {
    "id": 1,
    "username": "amir",
    "email": "amir@example.com",
    "emailVerified": true,
    "createdAt": "2026-09-13T10:00:00.000Z",
    "updatedAt": "2026-09-13T10:00:00.000Z"
  }
}
```

---

# 14. Update Current User

## 14.1 Update Profile

### Endpoint

```http
PATCH /api/v1/users/me
```

### Authentication

Required.

### Request Body

```json
{
  "username": "amir_sim"
}
```

All fields are optional.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "USER_UPDATED",
  "message": "Profile updated successfully",
  "data": {
    "id": 1,
    "username": "amir_sim",
    "email": "amir@example.com",
    "emailVerified": true,
    "updatedAt": "2026-09-13T12:00:00.000Z"
  }
}
```

### Rules

The user cannot modify:

```text
id
emailVerified
createdAt
```

through this endpoint.

---

# 15. Workspace Routes

## 15.1 Create Workspace

Creates a workspace.

### Endpoint

```http
POST /api/v1/workspaces
```

### Authentication

Required.

### Requirement

User's email must be verified.

### Request Body

```json
{
  "name": "Amir's Company"
}
```

### Response — `201 Created`

```json
{
  "statusCode": 201,
  "code": "WORKSPACE_CREATED",
  "message": "Workspace created successfully",
  "data": {
    "id": 1,
    "name": "Amir's Company",
    "createdBy": 1,
    "createdAt": "2026-09-13T10:00:00.000Z",
    "updatedAt": "2026-09-13T10:00:00.000Z"
  }
}
```

### Server Behavior

Creating a workspace also creates the creator's membership:

```text
role = OWNER
```

The operation should be transactional.

---

# 16. Get User Workspaces

Returns workspaces the authenticated user belongs to.

### Endpoint

```http
GET /api/v1/workspaces
```

### Authentication

Required.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "WORKSPACES_RETRIEVED",
  "message": "Workspaces retrieved successfully",
  "data": [
    {
      "id": 1,
      "name": "Amir's Company",
      "role": "OWNER",
      "createdAt": "2026-09-13T10:00:00.000Z"
    },
    {
      "id": 2,
      "name": "Another Company",
      "role": "MEMBER",
      "createdAt": "2026-09-13T11:00:00.000Z"
    }
  ]
}
```

---

# 17. Get Workspace

Returns workspace information.

### Endpoint

```http
GET /api/v1/workspaces/:workspaceId
```

### Authentication

Required.

### Path Parameters

```text
workspaceId: number
```

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "WORKSPACE_RETRIEVED",
  "message": "Workspace retrieved successfully",
  "data": {
    "id": 1,
    "name": "Amir's Company",
    "createdBy": 1,
    "createdAt": "2026-09-13T10:00:00.000Z",
    "updatedAt": "2026-09-13T10:00:00.000Z"
  }
}
```

---

# 18. Update Workspace

### Endpoint

```http
PATCH /api/v1/workspaces/:workspaceId
```

### Authentication

Required.

### Authorization

Owner.

### Request Body

```json
{
  "name": "Amir Technologies"
}
```

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "WORKSPACE_UPDATED",
  "message": "Workspace updated successfully",
  "data": {
    "id": 1,
    "name": "Amir Technologies",
    "updatedAt": "2026-09-13T12:00:00.000Z"
  }
}
```

---

# 19. Workspace Members

## 19.1 Get Workspace Members

Returns members of a workspace.

### Endpoint

```http
GET /api/v1/workspaces/:workspaceId/members
```

### Authentication

Required.

### Authorization

Owner/Admin.

A normal Member cannot retrieve the workspace's complete member list.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "WORKSPACE_MEMBERS_RETRIEVED",
  "message": "Workspace members retrieved successfully",
  "data": [
    {
      "id": 1,
      "user": {
        "id": 1,
        "username": "amir",
        "email": "amir@example.com"
      },
      "role": "OWNER",
      "joinedAt": "2026-09-13T10:00:00.000Z"
    },
    {
      "id": 2,
      "user": {
        "id": 2,
        "username": "ahmed",
        "email": "ahmed@example.com"
      },
      "role": "MEMBER",
      "joinedAt": "2026-09-13T10:30:00.000Z"
    }
  ]
}
```

---

# 20. Add Workspace Member

Adds an existing user to a workspace.

### Endpoint

```http
POST /api/v1/workspaces/:workspaceId/members
```

### Authentication

Required.

### Authorization

Owner or Admin.

### Request Body

```json
{
  "userId": 2,
  "role": "MEMBER"
}
```

### Response — `201 Created`

```json
{
  "statusCode": 201,
  "code": "WORKSPACE_MEMBER_ADDED",
  "message": "Workspace member added successfully",
  "data": {
    "id": 5,
    "userId": 2,
    "workspaceId": 1,
    "role": "MEMBER",
    "joinedAt": "2026-09-13T12:00:00.000Z"
  }
}
```

### Rules

Admin cannot create an Owner or promote a user to Admin.

Owner can add:

```text
ADMIN
MEMBER
```

Admin can add:

```text
MEMBER
```

The exact invitation mechanism is deferred; this endpoint assumes the target user already exists.

---

# 21. Update Workspace Member Role

### Endpoint

```http
PATCH /api/v1/workspaces/:workspaceId/members/:memberId
```

### Authentication

Required.

### Authorization

Owner only.

### Request Body

```json
{
  "role": "ADMIN"
}
```

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "WORKSPACE_MEMBER_UPDATED",
  "message": "Workspace member updated successfully",
  "data": {
    "id": 5,
    "userId": 2,
    "workspaceId": 1,
    "role": "ADMIN"
  }
}
```

### Rules

Owner can:

```text
MEMBER → ADMIN
ADMIN → MEMBER
```

Owner cannot create a second Owner.

Ownership transfer is a separate workflow.

---

# 22. Remove Workspace Member

### Endpoint

```http
DELETE /api/v1/workspaces/:workspaceId/members/:memberId
```

### Authentication

Required.

### Authorization

Owner only.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "WORKSPACE_MEMBER_REMOVED",
  "message": "Workspace member removed successfully",
  "data": null
}
```

### Rules

* Owner cannot remove themselves without transferring ownership.
* Admin cannot remove workspace members.
* Membership history should be preserved.
* The member's `removedAt` is recorded.

If the removed user has project memberships or assigned tasks, those lifecycle rules must also be applied.

---

# 23. Projects

## 23.1 Create Project

Creates a project inside a workspace.

### Endpoint

```http
POST /api/v1/workspaces/:workspaceId/projects
```

### Authentication

Required.

### Authorization

Owner or Admin.

### Request Body

```json
{
  "name": "TeamFlow Website",
  "description": "Build the TeamFlow web application"
}
```

### Response — `201 Created`

```json
{
  "statusCode": 201,
  "code": "PROJECT_CREATED",
  "message": "Project created successfully",
  "data": {
    "id": 1,
    "workspaceId": 1,
    "name": "TeamFlow Website",
    "description": "Build the TeamFlow web application",
    "status": "NOT_STARTED",
    "progress": 0,
    "createdBy": 1,
    "createdAt": "2026-09-13T12:00:00.000Z"
  }
}
```

### Rules

The creator does not automatically become Project Manager unless the application explicitly assigns them as such.

The Project Manager must be assigned before the project can enter its operational workflow.

---

# 24. Get Workspace Projects

Returns projects belonging to a workspace.

### Endpoint

```http
GET /api/v1/workspaces/:workspaceId/projects
```

### Authentication

Required.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "PROJECTS_RETRIEVED",
  "message": "Projects retrieved successfully",
  "data": [
    {
      "id": 1,
      "name": "TeamFlow Website",
      "status": "IN_PROGRESS",
      "progress": 65,
      "createdAt": "2026-09-13T12:00:00.000Z"
    }
  ]
}
```

Visibility must follow workspace/project authorization rules.

---

# 25. Get Project

Returns project details.

### Endpoint

```http
GET /api/v1/workspaces/:workspaceId/projects/:projectId
```

### Authentication

Required.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "PROJECT_RETRIEVED",
  "message": "Project retrieved successfully",
  "data": {
    "id": 1,
    "workspaceId": 1,
    "name": "TeamFlow Website",
    "description": "Build the TeamFlow web application",
    "status": "IN_PROGRESS",
    "progress": 65,
    "projectManager": {
      "id": 2,
      "username": "ahmed"
    },
    "createdBy": 1,
    "createdAt": "2026-09-13T12:00:00.000Z",
    "updatedAt": "2026-09-13T14:00:00.000Z"
  }
}
```

---

# 26. Update Project

### Endpoint

```http
PATCH /api/v1/workspaces/:workspaceId/projects/:projectId
```

### Authentication

Required.

### Authorization

Owner or Project Manager.

### Request Body

```json
{
  "name": "TeamFlow Web Platform",
  "description": "Updated project description"
}
```

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "PROJECT_UPDATED",
  "message": "Project updated successfully",
  "data": {
    "id": 1,
    "name": "TeamFlow Web Platform",
    "description": "Updated project description",
    "updatedAt": "2026-09-13T14:00:00.000Z"
  }
}
```

---

# 27. Update Project Status

### Endpoint

```http
PATCH /api/v1/workspaces/:workspaceId/projects/:projectId/status
```

### Authentication

Required.

### Request Body

```json
{
  "status": "COMPLETED"
}
```

### Authorization

Owner or Project Manager according to the project status transition rules.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "PROJECT_STATUS_UPDATED",
  "message": "Project status updated successfully",
  "data": {
    "id": 1,
    "status": "COMPLETED",
    "updatedAt": "2026-09-13T15:00:00.000Z"
  }
}
```

The domain must validate whether the requested transition is allowed.

---

# 28. Update Project Progress

### Endpoint

```http
PATCH /api/v1/workspaces/:workspaceId/projects/:projectId/progress
```

### Authentication

Required.

### Authorization

Owner or Project Manager.

### Request Body

```json
{
  "progress": 75
}
```

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "PROJECT_PROGRESS_UPDATED",
  "message": "Project progress updated successfully",
  "data": {
    "id": 1,
    "progress": 75,
    "updatedAt": "2026-09-13T15:00:00.000Z"
  }
}
```

### Rules

```text
0 <= progress <= 100
```

Progress is manually controlled.

It is not calculated automatically from task completion.

---

# 29. Delete/Archive Project

Because projects are not hard-deleted from the system's historical data, the endpoint represents the project's lifecycle removal/archive operation.

### Endpoint

```http
DELETE /api/v1/workspaces/:workspaceId/projects/:projectId
```

### Authentication

Required.

### Authorization

Owner only.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "PROJECT_DELETED",
  "message": "Project deleted successfully",
  "data": null
}
```

The underlying implementation must preserve required historical data.

---

# 30. Project Members

## 30.1 Get Project Members

### Endpoint

```http
GET /api/v1/workspaces/:workspaceId/projects/:projectId/members
```

### Authentication

Required.

### Authorization

Project Manager or Project Member.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "PROJECT_MEMBERS_RETRIEVED",
  "message": "Project members retrieved successfully",
  "data": [
    {
      "id": 1,
      "user": {
        "id": 2,
        "username": "ahmed",
        "email": "ahmed@example.com"
      },
      "role": "MANAGER",
      "joinedAt": "2026-09-13T12:10:00.000Z"
    },
    {
      "id": 2,
      "user": {
        "id": 3,
        "username": "ali",
        "email": "ali@example.com"
      },
      "role": "MEMBER",
      "joinedAt": "2026-09-13T12:15:00.000Z"
    }
  ]
}
```

---

# 31. Add Project Member

### Endpoint

```http
POST /api/v1/workspaces/:workspaceId/projects/:projectId/members
```

### Authentication

Required.

### Authorization

Owner or Project Manager.

### Request Body

```json
{
  "userId": 3
}
```

### Response — `201 Created`

```json
{
  "statusCode": 201,
  "code": "PROJECT_MEMBER_ADDED",
  "message": "Project member added successfully",
  "data": {
    "id": 2,
    "projectId": 1,
    "userId": 3,
    "role": "MEMBER",
    "joinedAt": "2026-09-13T12:15:00.000Z"
  }
}
```

### Rules

The user must already belong to the workspace.

The API/domain must reject adding a user who is not a workspace member.

A user cannot have multiple active memberships in the same project.

---

# 32. Remove Project Member

### Endpoint

```http
DELETE /api/v1/workspaces/:workspaceId/projects/:projectId/members/:memberId
```

### Authentication

Required.

### Authorization

Owner or Project Manager.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "PROJECT_MEMBER_REMOVED",
  "message": "Project member removed successfully",
  "data": null
}
```

### Rules

* The Project Manager cannot be removed without first selecting a replacement.
* Membership history is preserved.
* Active tasks assigned to the removed member become unassigned.

The operation should be transactional:

```text
Remove member
      ↓
Find active assigned tasks
      ↓
assignedTo = NULL
status = NOT_ASSIGNED
      ↓
Commit transaction
```

---

# 33. Assign Project Manager

The Owner chooses the Project Manager.

### Endpoint

```http
PATCH /api/v1/workspaces/:workspaceId/projects/:projectId/manager
```

### Authentication

Required.

### Authorization

Owner only.

### Request Body

```json
{
  "userId": 2
}
```

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "PROJECT_MANAGER_UPDATED",
  "message": "Project Manager updated successfully",
  "data": {
    "projectId": 1,
    "projectManager": {
      "id": 2,
      "username": "ahmed"
    }
  }
}
```

### Rules

The selected user must:

1. Belong to the workspace.
2. Be a project member.

If the user is not already a project member, the application must determine the appropriate workflow before assigning them as manager.

A project must have exactly one active Project Manager.

---

# 34. Tasks

## 34.1 Create Task

Creates a task inside a project.

### Endpoint

```http
POST /api/v1/workspaces/:workspaceId/projects/:projectId/tasks
```

### Authentication

Required.

### Authorization

Owner, Admin, or Project Manager according to project permissions.

### Request Body

```json
{
  "title": "Implement authentication",
  "description": "Implement signup, login and refresh-token authentication",
  "assignedTo": 3,
  "priority": "HIGH",
  "dueDate": "2026-10-01"
}
```

`assignedTo` is optional.

### Unassigned Task

```json
{
  "title": "Implement dashboard",
  "description": "Create the dashboard API",
  "priority": "MEDIUM",
  "dueDate": "2026-10-05"
}
```

### Response — Assigned

```json
{
  "statusCode": 201,
  "code": "TASK_CREATED",
  "message": "Task created successfully",
  "data": {
    "id": 1,
    "projectId": 1,
    "title": "Implement authentication",
    "description": "Implement signup, login and refresh-token authentication",
    "assignedTo": 3,
    "assignedBy": 2,
    "status": "PENDING",
    "priority": "HIGH",
    "dueDate": "2026-10-01",
    "createdBy": 2,
    "createdAt": "2026-09-13T13:00:00.000Z"
  }
}
```

### Response — Unassigned

```json
{
  "statusCode": 201,
  "code": "TASK_CREATED",
  "message": "Task created successfully",
  "data": {
    "id": 2,
    "projectId": 1,
    "title": "Implement dashboard",
    "description": "Create the dashboard API",
    "assignedTo": null,
    "assignedBy": null,
    "status": "NOT_ASSIGNED",
    "priority": "MEDIUM",
    "dueDate": "2026-10-05",
    "createdBy": 2,
    "createdAt": "2026-09-13T13:10:00.000Z"
  }
}
```

### Rules

If:

```text
assignedTo = null
```

then:

```text
status = NOT_ASSIGNED
```

If a member is assigned during creation:

```text
status = PENDING
```

The assigned user must be a Project Member.

---

# 35. Get Project Tasks

### Endpoint

```http
GET /api/v1/workspaces/:workspaceId/projects/:projectId/tasks
```

### Authentication

Required.

### Authorization

Users with appropriate project visibility.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "TASKS_RETRIEVED",
  "message": "Tasks retrieved successfully",
  "data": [
    {
      "id": 1,
      "title": "Implement authentication",
      "status": "IN_PROGRESS",
      "priority": "HIGH",
      "assignedTo": {
        "id": 3,
        "username": "ali"
      },
      "dueDate": "2026-10-01"
    },
    {
      "id": 2,
      "title": "Implement dashboard",
      "status": "NOT_ASSIGNED",
      "priority": "MEDIUM",
      "assignedTo": null,
      "dueDate": "2026-10-05"
    }
  ]
}
```

---

# 36. Get Task

### Endpoint

```http
GET /api/v1/workspaces/:workspaceId/projects/:projectId/tasks/:taskId
```

### Authentication

Required.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "TASK_RETRIEVED",
  "message": "Task retrieved successfully",
  "data": {
    "id": 1,
    "projectId": 1,
    "title": "Implement authentication",
    "description": "Implement signup, login and refresh-token authentication",
    "assignedTo": {
      "id": 3,
      "username": "ali"
    },
    "assignedBy": {
      "id": 2,
      "username": "ahmed"
    },
    "status": "IN_PROGRESS",
    "priority": "HIGH",
    "dueDate": "2026-10-01",
    "createdBy": {
      "id": 2,
      "username": "ahmed"
    },
    "createdAt": "2026-09-13T13:00:00.000Z",
    "updatedAt": "2026-09-13T14:00:00.000Z"
  }
}
```

---

# 37. Update Task

Edits task details.

### Endpoint

```http
PATCH /api/v1/workspaces/:workspaceId/projects/:projectId/tasks/:taskId
```

### Authentication

Required.

### Authorization

Owner, Admin, or Project Manager according to permissions.

### Request Body

```json
{
  "title": "Implement complete authentication",
  "description": "Updated requirements",
  "priority": "URGENT",
  "dueDate": "2026-10-03"
}
```

All fields are optional.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "TASK_UPDATED",
  "message": "Task updated successfully",
  "data": {
    "id": 1,
    "title": "Implement complete authentication",
    "description": "Updated requirements",
    "priority": "URGENT",
    "dueDate": "2026-10-03",
    "updatedAt": "2026-09-13T15:00:00.000Z"
  }
}
```

---

# 38. Assign/Reassign Task

### Endpoint

```http
PATCH /api/v1/workspaces/:workspaceId/projects/:projectId/tasks/:taskId/assignee
```

### Authentication

Required.

### Authorization

Owner, Admin, or Project Manager.

### Request Body

```json
{
  "userId": 3
}
```

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "TASK_ASSIGNED",
  "message": "Task assigned successfully",
  "data": {
    "id": 1,
    "assignedTo": 3,
    "assignedBy": 2,
    "status": "PENDING",
    "updatedAt": "2026-09-13T15:00:00.000Z"
  }
}
```

### Rules

The target user must be a Project Member.

When an unassigned task is assigned:

```text
NOT_ASSIGNED → PENDING
```

---

# 39. Unassign Task

Allows an authorized user to remove the current assignee.

### Endpoint

```http
DELETE /api/v1/workspaces/:workspaceId/projects/:projectId/tasks/:taskId/assignee
```

### Authentication

Required.

### Authorization

Owner, Admin, or Project Manager.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "TASK_UNASSIGNED",
  "message": "Task unassigned successfully",
  "data": {
    "id": 1,
    "assignedTo": null,
    "status": "NOT_ASSIGNED"
  }
}
```

---

# 40. Update Task Status

### Endpoint

```http
PATCH /api/v1/workspaces/:workspaceId/projects/:projectId/tasks/:taskId/status
```

### Authentication

Required.

### Request Body

```json
{
  "status": "IN_PROGRESS"
}
```

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "TASK_STATUS_UPDATED",
  "message": "Task status updated successfully",
  "data": {
    "id": 1,
    "status": "IN_PROGRESS",
    "updatedAt": "2026-09-13T15:30:00.000Z"
  }
}
```

### Authorization

The domain/application layer determines whether the authenticated user can perform the requested transition.

Project Member:

```text
PENDING → IN_PROGRESS
IN_PROGRESS → COMPLETED
```

Owner/Project Manager have broader control according to the task-status rules.

---

# 41. Drop Task

A dropped task remains in the database.

### Endpoint

```http
PATCH /api/v1/workspaces/:workspaceId/projects/:projectId/tasks/:taskId/drop
```

### Authentication

Required.

### Authorization

Owner or Project Manager.

### Request Body

No body.

### Response — `200 OK`

```json
{
  "statusCode": 200,
  "code": "TASK_DROPPED",
  "message": "Task dropped successfully",
  "data": {
    "id": 1,
    "status": "DROPPED",
    "droppedAt": "2026-09-13T16:00:00.000Z"
  }
}
```

This does not physically delete the task.

---

# 42. Task Filtering

Task collection endpoints may support query parameters rather than creating separate endpoints for every filter.

### Endpoint

```http
GET /api/v1/workspaces/:workspaceId/projects/:projectId/tasks
```

### Example

```http
GET /api/v1/workspaces/1/projects/1/tasks?status=IN_PROGRESS
```

Possible MVP filters:

```text
status
priority
assignedTo
```

Example:

```http
GET /api/v1/workspaces/1/projects/1/tasks?status=IN_PROGRESS&priority=HIGH
```

The API should validate query parameters.

Pagination can be added if the initial task collection requires it.

---

# 43. Authorization Model

Authorization must be enforced on the backend.

## Workspace

| Operation            | Owner | Admin | Member |
| -------------------- | ----: | ----: | -----: |
| View workspace       |     ✓ |     ✓ |      ✓ |
| View all members     |     ✓ |     ✓ |      ✗ |
| Add member           |     ✓ |     ✓ |      ✗ |
| Remove member        |     ✓ |     ✗ |      ✗ |
| Promote/demote Admin |     ✓ |     ✗ |      ✗ |
| Create project       |     ✓ |     ✓ |      ✗ |
| Update workspace     |     ✓ |     ✗ |      ✗ |

## Project

| Operation               | Owner | Admin |      PM | Member |
| ----------------------- | ----: | ----: | ------: | -----: |
| View project            |     ✓ |     ✓ |       ✓ |     ✓* |
| Update project          |     ✓ |     ✗ |       ✓ |      ✗ |
| Manage project members  |     ✓ |   TBD |       ✓ |      ✗ |
| Change PM               |     ✓ |     ✗ |       ✗ |      ✗ |
| Change project status   |     ✓ |     ✗ | Limited |      ✗ |
| Change project progress |     ✓ |     ✗ |       ✓ |      ✗ |

`*` Project visibility depends on the user's project membership/authorized workspace access.

## Tasks

| Operation          | Owner | Admin | PM |   Member |
| ------------------ | ----: | ----: | -: | -------: |
| View project tasks |     ✓ |     ✓ |  ✓ |       ✓* |
| Create task        |     ✓ |     ✓ |  ✓ |        ✗ |
| Edit task          |     ✓ |     ✓ |  ✓ |        ✗ |
| Assign task        |     ✓ |     ✓ |  ✓ |        ✗ |
| Reassign task      |     ✓ |     ✓ |  ✓ |        ✗ |
| Change task status |     ✓ |     ✓ |  ✓ | Own only |
| Drop task          |     ✓ |     ✗ |  ✓ |        ✗ |

---

# 44. Resource Ownership and Isolation

Every protected resource must be checked against the authenticated user's authorized scope.

For example:

```text
User
 ↓
Workspace membership
 ↓
Project membership/permission
 ↓
Task access
```

A user must not be able to access another workspace's:

* Projects
* Project members
* Tasks

simply by changing an ID in the URL.

Example:

```http
GET /api/v1/workspaces/999/projects/1
```

must not return project data merely because project `1` exists.

---

# 45. Important Domain Rules

The API layer must delegate business rules to the application/domain layer.

Examples:

### Workspace creation

```text
Email must be verified.
```

### Workspace owner

```text
Exactly one active Owner.
```

### Project Manager

```text
Exactly one active Project Manager.
```

### Project membership

```text
User must belong to workspace.
```

### Task assignment

```text
Assignee must belong to project.
```

### Task assignment state

```text
No assignee → NOT_ASSIGNED
Assigned → PENDING
```

### Member removal

```text
Remove member
→ assigned tasks become unassigned
→ task status becomes NOT_ASSIGNED
```

### Project Member status

```text
PENDING → IN_PROGRESS → COMPLETED
```

### Project progress

```text
0–100
```

### Dropped task

```text
Task remains stored.
status = DROPPED
```

---

# 46. Validation Requirements

All request bodies and query parameters must be validated.

Examples:

### Username

```text
required
string
valid length
```

### Email

```text
required
valid email format
```

### Password

```text
required
minimum security requirements
```

### Workspace name

```text
required
string
valid length
```

### Project progress

```text
integer
0–100
```

### Task priority

```text
LOW
MEDIUM
HIGH
URGENT
```

### Task status

```text
NOT_ASSIGNED
PENDING
IN_PROGRESS
COMPLETED
DROPPED
```

---

# 47. Resource Not Found

When a requested resource does not exist or the user is not authorized to know about it, the API should avoid unnecessarily revealing protected resource existence.

Example:

```json
{
  "statusCode": 404,
  "code": "RESOURCE_NOT_FOUND",
  "message": "Resource not found"
}
```

---

# 48. Conflict Errors

Examples include:

### Duplicate email

```json
{
  "statusCode": 409,
  "code": "EMAIL_ALREADY_EXISTS",
  "message": "An account with this email already exists"
}
```

### Duplicate workspace membership

```json
{
  "statusCode": 409,
  "code": "ALREADY_WORKSPACE_MEMBER",
  "message": "User is already a member of this workspace"
}
```

### Duplicate project membership

```json
{
  "statusCode": 409,
  "code": "ALREADY_PROJECT_MEMBER",
  "message": "User is already a member of this project"
}
```

---

# 49. Authentication Error Examples

## Invalid Credentials

```json
{
  "statusCode": 401,
  "code": "INVALID_CREDENTIALS",
  "message": "Invalid email or password"
}
```

## Invalid Access Token

```json
{
  "statusCode": 401,
  "code": "INVALID_ACCESS_TOKEN",
  "message": "Authentication required"
}
```

## Expired Refresh Session

```json
{
  "statusCode": 401,
  "code": "REFRESH_SESSION_EXPIRED",
  "message": "Refresh session has expired"
}
```

## Unverified Email

```json
{
  "statusCode": 403,
  "code": "EMAIL_NOT_VERIFIED",
  "message": "Email verification is required"
}
```

---

# 50. Complete MVP Route List

## Authentication

```text
POST   /api/v1/auth/signup
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
POST   /api/v1/auth/logout

POST   /api/v1/auth/email-verification/send
POST   /api/v1/auth/email-verification/verify
```

## Users

```text
GET    /api/v1/users/me
PATCH  /api/v1/users/me
```

## Workspaces

```text
POST   /api/v1/workspaces
GET    /api/v1/workspaces
GET    /api/v1/workspaces/:workspaceId
PATCH  /api/v1/workspaces/:workspaceId
```

## Workspace Members

```text
GET    /api/v1/workspaces/:workspaceId/members
POST   /api/v1/workspaces/:workspaceId/members
PATCH  /api/v1/workspaces/:workspaceId/members/:memberId
DELETE /api/v1/workspaces/:workspaceId/members/:memberId
```

## Projects

```text
POST   /api/v1/workspaces/:workspaceId/projects
GET    /api/v1/workspaces/:workspaceId/projects
GET    /api/v1/workspaces/:workspaceId/projects/:projectId
PATCH  /api/v1/workspaces/:workspaceId/projects/:projectId
DELETE /api/v1/workspaces/:workspaceId/projects/:projectId

PATCH  /api/v1/workspaces/:workspaceId/projects/:projectId/status
PATCH  /api/v1/workspaces/:workspaceId/projects/:projectId/progress
PATCH  /api/v1/workspaces/:workspaceId/projects/:projectId/manager
```

## Project Members

```text
GET    /api/v1/workspaces/:workspaceId/projects/:projectId/members
POST   /api/v1/workspaces/:workspaceId/projects/:projectId/members
DELETE /api/v1/workspaces/:workspaceId/projects/:projectId/members/:memberId
```

## Tasks

```text
POST   /api/v1/workspaces/:workspaceId/projects/:projectId/tasks
GET    /api/v1/workspaces/:workspaceId/projects/:projectId/tasks
GET    /api/v1/workspaces/:workspaceId/projects/:projectId/tasks/:taskId
PATCH  /api/v1/workspaces/:workspaceId/projects/:projectId/tasks/:taskId

PATCH  /api/v1/workspaces/:workspaceId/projects/:projectId/tasks/:taskId/assignee
DELETE /api/v1/workspaces/:workspaceId/projects/:projectId/tasks/:taskId/assignee

PATCH  /api/v1/workspaces/:workspaceId/projects/:projectId/tasks/:taskId/status
PATCH  /api/v1/workspaces/:workspaceId/projects/:projectId/tasks/:taskId/drop
```

---

# 51. MVP Route Count

```text
Authentication       6
Users                2
Workspaces           4
Workspace Members    4
Projects             8
Project Members      3
Tasks                8
──────────────────────
Total                35 routes
```

These are the intended **MVP HTTP endpoints**.

---

# 52. Explicitly Not Included

The following endpoints should **not** be implemented as MVP routes:

```text
POST   /auth/forgot-password
POST   /auth/reset-password

DELETE /users/me

POST   /workspaces/:id/invitations
GET    /notifications
POST   /comments
GET    /comments
POST   /attachments

WebSocket endpoints
Chat endpoints
AI endpoints
Analytics/reporting endpoints
```

They can be introduced in later versions when their requirements are finalized.

---

# 53. API Design Principle

TeamFlow follows this separation:

```text
HTTP Request
     ↓
NestJS Controller
     ↓
Application Use Case
     ↓
Domain
     ↓
Repository Interface
     ↓
Infrastructure
     ↓
Kysely
     ↓
PostgreSQL
```

Controllers should not contain business rules.

For example, the controller should not decide:

```text
"Is this user allowed to assign this task?"
```

It should pass the request to the application layer, which coordinates the domain and authorization rules.

---

# 54. Final MVP API Workflow

A normal TeamFlow workflow looks like:

```text
POST /auth/signup
        ↓
POST /auth/email-verification/send
        ↓
POST /auth/email-verification/verify
        ↓
POST /auth/login
        ↓
POST /workspaces
        ↓
POST /workspaces/:workspaceId/members
        ↓
POST /workspaces/:workspaceId/projects
        ↓
PATCH /workspaces/:workspaceId/projects/:projectId/manager
        ↓
POST /workspaces/:workspaceId/projects/:projectId/members
        ↓
POST /workspaces/:workspaceId/projects/:projectId/tasks
        ↓
PATCH /.../tasks/:taskId/assignee
        ↓
PATCH /.../tasks/:taskId/status
        ↓
PATCH /.../projects/:projectId/progress
```

This represents the core TeamFlow MVP workflow from account creation through project/task management.
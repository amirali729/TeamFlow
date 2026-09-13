# TeamFlow — DataBase Modeling 

## Entities
    - Users
    - Workspace
    - WorkspaceMember
    - Project
    - ProjectMember
    - Task
    - RefreshSession
    - EmailVerification

### Users
```
    Attributes
    ├── id
    ├── username
    ├── email
    ├── password_hash
    ├── email_verified_at
    ├── created_at
    ├── updated_at
    └── deleted_at 
```

| Field               | Type              | Constraints      | Purpose                 |
| ------------------- | ----------------- | ---------------- | ----------------------- |
| `id`                | `bigint` / `uuid` | PK               | User identity           |
| `username`          | `varchar`         | UNIQUE, NOT NULL | Display/login identity  |
| `email`             | `varchar`         | UNIQUE, NOT NULL | Account email           |
| `password_hash`     | `varchar`         | NOT NULL         | Hashed password         |
| `email_verified_at` | `boolean`         | false            |`FALSE` means unverified |
| `created_at`        | `timestamptz`     | NOT NULL         | Creation time           |
| `updated_at`        | `timestamptz`     | NOT NULL         | Last update             |
| `deleted_at`        | `timestamptz`     | NULL             | Soft deletion           |

### workspaces
    
    ├── id
    ├── name
    ├── created_by
    ├── created_at
    ├── updated_at
    └── deleted_at

| Field        | Type          | Constraints | Purpose            |
| ------------ | ------------- | ----------- | ------------------ |
| `id`         | bigint/uuid   | PK          | Workspace identity |
| `name`       | varchar       | NOT NULL    | Workspace name     |
| `created_by` | FK → users.id | NOT NULL    | Who created it     |
| `created_at` | timestamptz   | NOT NULL    | Creation time      |
| `updated_at` | timestamptz   | NOT NULL    | Last update        |
| `deleted_at` | timestamptz   | NULL        | Soft deletion      |

    UNIQUE(userId, name)

### workspace_members
    
    ├── id
    ├── workspace_id
    ├── user_id
    ├── role
    ├── joined_at
    └── removed_at

| Field          | Type        | Constraints | Purpose                |
| -------------- | ----------- | ----------- | ---------------------- |
| `id`           | bigint/uuid | PK          | Membership identity    |
| `workspace_id` | FK          | NOT NULL    | Workspace              |
| `user_id`      | FK          | NOT NULL    | User                   |
| `role`         | enum        | NOT NULL    | OWNER / ADMIN / MEMBER |
| `joined_at`    | timestamptz | NOT NULL    | When joined            |
| `removed_at`   | timestamptz | NULL        | Membership removal     |

    UNIQUE(userId, name)

### Projects 
    
    ├── id
    ├── workspace_id
    ├── name
    ├── description
    ├── status
    ├── progress
    ├── created_by
    ├── created_at
    ├── updated_at
    └── deleted_at

| Field          | Type          | Constraints | Purpose                      |
| -------------- | ------------- | ----------- | ---------------------------- |
| `id`           | bigint/uuid   | PK          | Project identity             |
| `workspace_id` | FK            | NOT NULL    | Parent workspace             |
| `name`         | varchar       | NOT NULL    | Project name                 |
| `description`  | text          | NULL        | Project description          |
| `status`       | enum          | NOT NULL    | Project status               |
| `progress`     | smallint/int  | NOT NULL    | Manually controlled progress |
| `created_by`   | FK → users.id | NOT NULL    | Creator                      |
| `created_at`   | timestamptz   | NOT NULL    | Creation                     |
| `updated_at`   | timestamptz   | NOT NULL    | Last update                  |
| `deleted_at`   | timestamptz   | NULL        | Soft deletion                |

### project_members
    ├── id
    ├── project_id
    ├── user_id
    ├── role
    ├── joined_at
    └── removed_at
| Field        | Type        | Constraints | Purpose             |
| ------------ | ----------- | ----------- | ------------------- |
| `id`         | bigint/uuid | PK          | Membership identity |
| `project_id` | FK          | NOT NULL    | Project             |
| `user_id`    | FK          | NOT NULL    | User                |
| `role`       | enum        | NOT NULL    | MANAGER / MEMBER    |
| `joined_at`  | timestamptz | NOT NULL    | Joining time        |
| `removed_at` | timestamptz | NULL        | Removal time        |

### tasks
    ├── id
    ├── project_id
    ├── title
    ├── description
    ├── assigned_to
    ├── assigned_by
    ├── status
    ├── priority
    ├── due_date
    ├── created_by
    ├── created_at
    ├── updated_at
    └── dropped_at
| Field         | Type           | Constraints | Purpose          |
| ------------- | -------------- | ----------- | ---------------- |
| `id`          | bigint/uuid    | PK          | Task identity    |
| `project_id`  | FK             | NOT NULL    | Parent project   |
| `title`       | varchar        | NOT NULL    | Task title       |
| `description` | text           | NULL        | Details          |
| `assigned_to` | FK → users.id  | NULL        | Current assignee |
| `assigned_by` | FK → users.id  | NULL        | Who assigned it  |
| `status`      | enum           | NOT NULL    | Task status      |
| `priority`    | enum           | NOT NULL    | Priority         |
| `due_date`    | date/timestamp | NOT NULL    | Deadline         |
| `created_by`  | FK → users.id  | NOT NULL    | Creator          |
| `created_at`  | timestamptz    | NOT NULL    | Creation         |
| `updated_at`  | timestamptz    | NOT NULL    | Last update      |
| `dropped_at`  | timestamptz    | NULL        | When dropped     |

### refresh_sessions
    ├── id
    ├── user_id
    ├── token_hash
    ├── expires_at
    ├── revoked_at
    ├── created_at
    └── updated_at

| Field        | Type        | Constraints      | Purpose              |
| ------------ | ----------- | ---------------- | -------------------- |
| `id`         | bigint/uuid | PK               | Session identity     |
| `user_id`    | FK          | NOT NULL         | Session owner        |
| `token_hash` | varchar     | UNIQUE, NOT NULL | Hashed refresh token |
| `expires_at` | timestamptz | NOT NULL         | Expiration           |
| `revoked_at` | timestamptz | NULL             | Logout/revocation    |
| `created_at` | timestamptz | NOT NULL         | Session creation     |
| `updated_at` | timestamptz | NOT NULL         | Last modification    |

### email_verifications

    ├── id
    ├── user_id
    ├── token_hash
    ├── expires_at
    ├── used_at
    └── created_at
| Field        | Type        | Purpose                   |
| ------------ | ----------- | ------------------------- |
| `id`         | bigint/uuid | Verification record       |
| `user_id`    | FK          | User                      |
| `type`       | enum        | type of Verification      |  
| `token_hash` | varchar     | Hashed verification token |
| `expires_at` | timestamptz | Expiration                |
| `used_at`    | timestamptz | `NULL` until consumed     |
| `created_at` | timestamptz | Creation                  |

## Relationships:

    User M:N Workspace 
        new Table will be WorkspaceMember
    
    WorkSpace 1:N Project

    User M:N Project
        new Table will be ProjectMember


    Project 1:N Task

    User 1:N RefreshSession

    User 1:N EmailVerification

 

        
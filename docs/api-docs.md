# Saturn-X REST API Documentation

## Table of Contents

- [Authentication](#authentication)
  - [POST /api/auth/login](#post-apiauthlogin)
  - [POST /api/auth/callback](#post-apiauthcallback)
  - [POST /api/auth/refresh](#post-apiauthrefresh)
  - [POST /api/auth/logout](#post-apiauthlogout)
- [Users](#users)
  - [GET /api/users/me](#get-apiusersme)
  - [PUT /api/users/me](#put-apiusersme)
  - [DELETE /api/users/me](#delete-apiusersme)
- [Preferences](#preferences)
  - [GET /api/preferences](#get-apipreferences)
  - [PUT /api/preferences](#put-apipreferences)
  - [PATCH /api/preferences](#patch-apipreferences)
- [Tasks](#tasks)
  - [POST /api/tasks](#post-apitasks)
  - [GET /api/tasks](#get-apitasks)
  - [GET /api/tasks/{taskId}](#get-apitaskstaskid)
  - [PUT /api/tasks/{taskId}](#put-apitaskstaskid)
  - [PATCH /api/tasks/{taskId}](#patch-apitaskstaskid)
  - [DELETE /api/tasks/{taskId}](#delete-apitaskstaskid)
  - [POST /api/tasks/{taskId}/complete](#post-apitaskstaskidcomplete)
  - [POST /api/tasks/{taskId}/generate-subtasks](#post-apitaskstaskidgenerate-subtasks)
- [Subtasks](#subtasks)
  - [GET /api/tasks/{taskId}/subtasks](#get-apitaskstaskidsubtasks)
  - [POST /api/tasks/{taskId}/subtasks](#post-apitaskstaskidsubtasks)
  - [GET /api/subtasks/{subtaskId}](#get-apisubtaskssubtaskid)
  - [PUT /api/subtasks/{subtaskId}](#put-apisubtaskssubtaskid)
  - [DELETE /api/subtasks/{subtaskId}](#delete-apisubtaskssubtaskid)
  - [POST /api/subtasks/{subtaskId}/complete](#post-apisubtaskssubtaskidcomplete)
- [Calendar](#calendar)
  - [GET /api/calendar/weekly](#get-apicalendarweekly)
  - [GET /api/calendar/monthly](#get-apicalendarmonthly)
  - [GET /api/calendar/tasks](#get-apicalendartasks)
- [Streaks](#streaks)
  - [GET /api/streaks/current](#get-apistreakscurrent)
  - [GET /api/streaks/history](#get-apistreakshistory)
- [Reminders](#reminders)
  - [GET /api/reminders/today](#get-apireminderstoday)
  - [GET /api/reminders](#get-apireminders)
- [Google Calendar Sync](#google-calendar-sync)
  - [POST /api/google-calendar/connect](#post-apigoogle-calendarconnect)
  - [DELETE /api/google-calendar/disconnect](#delete-apigoogle-calendardisconnect)
  - [POST /api/google-calendar/sync](#post-apigoogle-calendarsync)
  - [POST /api/google-calendar/webhook](#post-apigoogle-calendarwebhook)

---

## General Information

### Base URL
```
http://localhost:8080/api
```

### Authentication
Most endpoints require JWT authentication. Include the token in the Authorization header:
```
Authorization: Bearer <your_jwt_token>
```

### Pagination
List endpoints support offset-based pagination using query parameters:
- `page`: Page number (0-indexed, default: 0)
- `size`: Number of items per page (default: 20, max: 100)

### Common Response Codes
- `200 OK`: Request succeeded
- `201 Created`: Resource created successfully
- `204 No Content`: Request succeeded with no response body
- `400 Bad Request`: Invalid request parameters or body
- `401 Unauthorized`: Missing or invalid authentication token
- `403 Forbidden`: Authenticated but not authorized for this resource
- `404 Not Found`: Resource not found
- `500 Internal Server Error`: Server error

### Error Response Format
```json
{
  "timestamp": "2025-11-06T12:34:56.789Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Detailed error message",
  "path": "/api/tasks"
}
```

---

## Authentication

### POST /api/auth/login

Initiates OAuth authentication flow for Google or GitHub.

**Authentication Required**: No

#### Query Parameters

| Name     | Type   | Required | Description                    |
|----------|--------|----------|--------------------------------|
| provider | string | yes      | OAuth provider: `google` or `github` |

#### Response
**Status**: `302 Found`

Redirects to the OAuth provider's login page.

---

### POST /api/auth/callback

Handles OAuth callback and issues JWT tokens.

**Authentication Required**: No

#### Query Parameters

| Name  | Type   | Required | Description                        |
|-------|--------|----------|------------------------------------|
| code  | string | yes      | Authorization code from OAuth provider |
| state | string | yes      | State parameter for CSRF protection    |

#### Response
**Status**: `200 OK`

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600,
  "user": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "email": "user@example.com",
    "name": "John Doe",
    "picture": "https://example.com/avatar.jpg",
    "provider": "google"
  }
}
```

---

### POST /api/auth/refresh

Refreshes an expired access token using a refresh token.

**Authentication Required**: No

#### Request Body

**Content-Type**: `application/json`

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### Response
**Status**: `200 OK`

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

---

### POST /api/auth/logout

Logs out the current user and invalidates tokens.

**Authentication Required**: Yes

#### Request Body

**Content-Type**: `application/json`

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### Response
**Status**: `204 No Content`

---

## Users

### GET /api/users/me

Retrieves the current authenticated user's profile.

**Authentication Required**: Yes

#### Response
**Status**: `200 OK`

```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "email": "user@example.com",
  "name": "John Doe",
  "picture": "https://example.com/avatar.jpg",
  "provider": "google",
  "createdAt": "2025-01-15T10:30:00Z",
  "updatedAt": "2025-01-15T10:30:00Z"
}
```

---

### PUT /api/users/me

Updates the current user's profile.

**Authentication Required**: Yes

#### Request Body

**Content-Type**: `application/json`

```json
{
  "name": "John Doe",
  "picture": "https://example.com/new-avatar.jpg"
}
```

#### Response
**Status**: `200 OK`

```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "email": "user@example.com",
  "name": "John Doe",
  "picture": "https://example.com/new-avatar.jpg",
  "provider": "google",
  "createdAt": "2025-01-15T10:30:00Z",
  "updatedAt": "2025-11-06T14:22:00Z"
}
```

---

### DELETE /api/users/me

Deletes the current user's account and all associated data.

**Authentication Required**: Yes

#### Response
**Status**: `204 No Content`

**Note**: This action is irreversible and will delete all tasks, subtasks, preferences, and streak data.

---

## Preferences

### GET /api/preferences

Retrieves the current user's study preferences and habits.

**Authentication Required**: Yes

#### Response
**Status**: `200 OK`

```json
{
  "id": "pref-123",
  "userId": "123e4567-e89b-12d3-a456-426614174000",
  "priorityMode": "hardest_first",
  "workingHoursStart": "09:00",
  "workingHoursEnd": "17:00",
  "preferredSessionDuration": 90,
  "breakDuration": 15,
  "daysOfWeek": ["monday", "tuesday", "wednesday", "thursday", "friday"],
  "timezone": "America/New_York",
  "createdAt": "2025-01-15T10:30:00Z",
  "updatedAt": "2025-01-20T11:00:00Z"
}
```

#### Field Descriptions

- `priorityMode`: How to order tasks - `hardest_first`, `easiest_first`, `earliest_deadline`, `manual`
- `workingHoursStart`: Preferred start time for study sessions (HH:mm format)
- `workingHoursEnd`: Preferred end time for study sessions (HH:mm format)
- `preferredSessionDuration`: Ideal study session length in minutes
- `breakDuration`: Break duration between sessions in minutes
- `daysOfWeek`: Days available for studying
- `timezone`: User's timezone for scheduling

---

### PUT /api/preferences

Updates all user preferences (full replacement).

**Authentication Required**: Yes

#### Request Body

**Content-Type**: `application/json`

```json
{
  "priorityMode": "earliest_deadline",
  "workingHoursStart": "10:00",
  "workingHoursEnd": "18:00",
  "preferredSessionDuration": 120,
  "breakDuration": 20,
  "daysOfWeek": ["monday", "wednesday", "friday", "saturday"],
  "timezone": "America/Los_Angeles"
}
```

#### Response
**Status**: `200 OK`

```json
{
  "id": "pref-123",
  "userId": "123e4567-e89b-12d3-a456-426614174000",
  "priorityMode": "earliest_deadline",
  "workingHoursStart": "10:00",
  "workingHoursEnd": "18:00",
  "preferredSessionDuration": 120,
  "breakDuration": 20,
  "daysOfWeek": ["monday", "wednesday", "friday", "saturday"],
  "timezone": "America/Los_Angeles",
  "createdAt": "2025-01-15T10:30:00Z",
  "updatedAt": "2025-11-06T14:30:00Z"
}
```

---

### PATCH /api/preferences

Partially updates user preferences (only specified fields).

**Authentication Required**: Yes

#### Request Body

**Content-Type**: `application/json`

```json
{
  "priorityMode": "hardest_first",
  "preferredSessionDuration": 90
}
```

#### Response
**Status**: `200 OK`

```json
{
  "id": "pref-123",
  "userId": "123e4567-e89b-12d3-a456-426614174000",
  "priorityMode": "hardest_first",
  "workingHoursStart": "10:00",
  "workingHoursEnd": "18:00",
  "preferredSessionDuration": 90,
  "breakDuration": 20,
  "daysOfWeek": ["monday", "wednesday", "friday", "saturday"],
  "timezone": "America/Los_Angeles",
  "createdAt": "2025-01-15T10:30:00Z",
  "updatedAt": "2025-11-06T14:35:00Z"
}
```

---

## Tasks

### POST /api/tasks

Creates a new task/assignment. Supports both text description and file upload.

**Authentication Required**: Yes

#### Request Body (Text Description)

**Content-Type**: `application/json`

```json
{
  "title": "Research Paper on Machine Learning",
  "description": "Write a 10-page research paper on the applications of machine learning in healthcare",
  "dueDate": "2025-11-20T23:59:59Z",
  "estimatedHours": 15,
  "difficultyLevel": "hard"
}
```

#### Request Body (File Upload)

**Content-Type**: `multipart/form-data`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| file | file | yes | Assignment file (PDF, DOCX, TXT, etc.) |
| title | string | yes | Task title |
| dueDate | datetime | yes | Due date in ISO 8601 format |
| estimatedHours | number | no | Estimated hours to complete |
| difficultyLevel | string | no | `easy`, `medium`, `hard` |

#### Response
**Status**: `201 Created`

```json
{
  "id": "task-456",
  "userId": "123e4567-e89b-12d3-a456-426614174000",
  "title": "Research Paper on Machine Learning",
  "description": "Write a 10-page research paper on the applications of machine learning in healthcare",
  "dueDate": "2025-11-20T23:59:59Z",
  "estimatedHours": 15,
  "difficultyLevel": "hard",
  "status": "pending",
  "completedAt": null,
  "fileUrl": null,
  "createdAt": "2025-11-06T15:00:00Z",
  "updatedAt": "2025-11-06T15:00:00Z",
  "daysRemaining": 14,
  "subtaskCount": 0,
  "completedSubtaskCount": 0
}
```

---

### GET /api/tasks

Retrieves a paginated list of all tasks for the current user.

**Authentication Required**: Yes

#### Query Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| page | number | no | Page number (0-indexed, default: 0) |
| size | number | no | Page size (default: 20, max: 100) |
| status | string | no | Filter by status: `pending`, `in_progress`, `completed` |
| sortBy | string | no | Sort field: `dueDate`, `createdAt`, `title` (default: `dueDate`) |
| order | string | no | Sort order: `asc`, `desc` (default: `asc`) |

#### Response
**Status**: `200 OK`

```json
{
  "content": [
    {
      "id": "task-456",
      "userId": "123e4567-e89b-12d3-a456-426614174000",
      "title": "Research Paper on Machine Learning",
      "description": "Write a 10-page research paper...",
      "dueDate": "2025-11-20T23:59:59Z",
      "estimatedHours": 15,
      "difficultyLevel": "hard",
      "status": "pending",
      "completedAt": null,
      "fileUrl": null,
      "createdAt": "2025-11-06T15:00:00Z",
      "updatedAt": "2025-11-06T15:00:00Z",
      "daysRemaining": 14,
      "subtaskCount": 5,
      "completedSubtaskCount": 2
    }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 20,
    "sort": {
      "sorted": true,
      "unsorted": false,
      "empty": false
    },
    "offset": 0,
    "paged": true,
    "unpaged": false
  },
  "totalPages": 1,
  "totalElements": 1,
  "last": true,
  "size": 20,
  "number": 0,
  "first": true,
  "numberOfElements": 1,
  "empty": false
}
```

---

### GET /api/tasks/{taskId}

Retrieves a single task by ID.

**Authentication Required**: Yes

#### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| taskId | string | yes | Task UUID |

#### Response
**Status**: `200 OK`

```json
{
  "id": "task-456",
  "userId": "123e4567-e89b-12d3-a456-426614174000",
  "title": "Research Paper on Machine Learning",
  "description": "Write a 10-page research paper on the applications of machine learning in healthcare",
  "dueDate": "2025-11-20T23:59:59Z",
  "estimatedHours": 15,
  "difficultyLevel": "hard",
  "status": "in_progress",
  "completedAt": null,
  "fileUrl": null,
  "createdAt": "2025-11-06T15:00:00Z",
  "updatedAt": "2025-11-06T15:00:00Z",
  "daysRemaining": 14,
  "subtaskCount": 5,
  "completedSubtaskCount": 2,
  "subtasks": [
    {
      "id": "subtask-1",
      "title": "Research healthcare ML applications",
      "scheduledDate": "2025-11-07",
      "status": "completed",
      "completedAt": "2025-11-07T16:30:00Z"
    },
    {
      "id": "subtask-2",
      "title": "Create outline",
      "scheduledDate": "2025-11-08",
      "status": "completed",
      "completedAt": "2025-11-08T14:20:00Z"
    },
    {
      "id": "subtask-3",
      "title": "Write introduction",
      "scheduledDate": "2025-11-09",
      "status": "pending",
      "completedAt": null
    }
  ]
}
```

---

### PUT /api/tasks/{taskId}

Updates all fields of a task (full replacement).

**Authentication Required**: Yes

#### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| taskId | string | yes | Task UUID |

#### Request Body

**Content-Type**: `application/json`

```json
{
  "title": "Research Paper on AI in Healthcare",
  "description": "Updated description with more details",
  "dueDate": "2025-11-25T23:59:59Z",
  "estimatedHours": 20,
  "difficultyLevel": "hard"
}
```

#### Response
**Status**: `200 OK`

```json
{
  "id": "task-456",
  "userId": "123e4567-e89b-12d3-a456-426614174000",
  "title": "Research Paper on AI in Healthcare",
  "description": "Updated description with more details",
  "dueDate": "2025-11-25T23:59:59Z",
  "estimatedHours": 20,
  "difficultyLevel": "hard",
  "status": "in_progress",
  "completedAt": null,
  "fileUrl": null,
  "createdAt": "2025-11-06T15:00:00Z",
  "updatedAt": "2025-11-06T16:00:00Z",
  "daysRemaining": 19,
  "subtaskCount": 5,
  "completedSubtaskCount": 2
}
```

---

### PATCH /api/tasks/{taskId}

Partially updates a task (only specified fields).

**Authentication Required**: Yes

#### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| taskId | string | yes | Task UUID |

#### Request Body

**Content-Type**: `application/json`

```json
{
  "dueDate": "2025-11-22T23:59:59Z",
  "status": "in_progress"
}
```

#### Response
**Status**: `200 OK`

```json
{
  "id": "task-456",
  "userId": "123e4567-e89b-12d3-a456-426614174000",
  "title": "Research Paper on AI in Healthcare",
  "description": "Updated description with more details",
  "dueDate": "2025-11-22T23:59:59Z",
  "estimatedHours": 20,
  "difficultyLevel": "hard",
  "status": "in_progress",
  "completedAt": null,
  "fileUrl": null,
  "createdAt": "2025-11-06T15:00:00Z",
  "updatedAt": "2025-11-06T16:05:00Z",
  "daysRemaining": 16,
  "subtaskCount": 5,
  "completedSubtaskCount": 2
}
```

---

### DELETE /api/tasks/{taskId}

Deletes a task and all associated subtasks.

**Authentication Required**: Yes

#### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| taskId | string | yes | Task UUID |

#### Response
**Status**: `204 No Content`

---

### POST /api/tasks/{taskId}/complete

Marks a task as completed.

**Authentication Required**: Yes

#### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| taskId | string | yes | Task UUID |

#### Response
**Status**: `200 OK`

```json
{
  "id": "task-456",
  "userId": "123e4567-e89b-12d3-a456-426614174000",
  "title": "Research Paper on AI in Healthcare",
  "description": "Updated description with more details",
  "dueDate": "2025-11-22T23:59:59Z",
  "estimatedHours": 20,
  "difficultyLevel": "hard",
  "status": "completed",
  "completedAt": "2025-11-06T16:30:00Z",
  "fileUrl": null,
  "createdAt": "2025-11-06T15:00:00Z",
  "updatedAt": "2025-11-06T16:30:00Z",
  "daysRemaining": 16,
  "subtaskCount": 5,
  "completedSubtaskCount": 5
}
```

---

### POST /api/tasks/{taskId}/generate-subtasks

Generates AI-powered subtasks for a task based on user preferences.

**Authentication Required**: Yes

#### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| taskId | string | yes | Task UUID |

#### Response
**Status**: `200 OK`

**Note**: This is a synchronous operation that may take several seconds to complete.

```json
{
  "taskId": "task-456",
  "subtasks": [
    {
      "id": "subtask-1",
      "taskId": "task-456",
      "title": "Research healthcare ML applications",
      "description": "Research and compile 10-15 recent applications of ML in healthcare",
      "scheduledDate": "2025-11-07",
      "estimatedDuration": 120,
      "order": 1,
      "status": "pending",
      "completedAt": null,
      "createdAt": "2025-11-06T16:45:00Z"
    },
    {
      "id": "subtask-2",
      "taskId": "task-456",
      "title": "Create paper outline",
      "description": "Develop detailed outline with sections and key points",
      "scheduledDate": "2025-11-08",
      "estimatedDuration": 90,
      "order": 2,
      "status": "pending",
      "completedAt": null,
      "createdAt": "2025-11-06T16:45:00Z"
    }
  ],
  "totalGenerated": 8,
  "message": "Successfully generated 8 subtasks based on your preferences"
}
```

---

## Subtasks

### GET /api/tasks/{taskId}/subtasks

Retrieves all subtasks for a specific task.

**Authentication Required**: Yes

#### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| taskId | string | yes | Task UUID |

#### Query Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| status | string | no | Filter by status: `pending`, `completed` |

#### Response
**Status**: `200 OK`

```json
[
  {
    "id": "subtask-1",
    "taskId": "task-456",
    "title": "Research healthcare ML applications",
    "description": "Research and compile 10-15 recent applications of ML in healthcare",
    "scheduledDate": "2025-11-07",
    "estimatedDuration": 120,
    "order": 1,
    "status": "completed",
    "completedAt": "2025-11-07T16:30:00Z",
    "createdAt": "2025-11-06T16:45:00Z",
    "updatedAt": "2025-11-07T16:30:00Z"
  },
  {
    "id": "subtask-2",
    "taskId": "task-456",
    "title": "Create paper outline",
    "description": "Develop detailed outline with sections and key points",
    "scheduledDate": "2025-11-08",
    "estimatedDuration": 90,
    "order": 2,
    "status": "pending",
    "completedAt": null,
    "createdAt": "2025-11-06T16:45:00Z",
    "updatedAt": "2025-11-06T16:45:00Z"
  }
]
```

---

### POST /api/tasks/{taskId}/subtasks

Creates a new manual subtask for a task.

**Authentication Required**: Yes

#### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| taskId | string | yes | Task UUID |

#### Request Body

**Content-Type**: `application/json`

```json
{
  "title": "Proofread final draft",
  "description": "Review and fix grammar, spelling, and formatting issues",
  "scheduledDate": "2025-11-19",
  "estimatedDuration": 60
}
```

#### Response
**Status**: `201 Created`

```json
{
  "id": "subtask-9",
  "taskId": "task-456",
  "title": "Proofread final draft",
  "description": "Review and fix grammar, spelling, and formatting issues",
  "scheduledDate": "2025-11-19",
  "estimatedDuration": 60,
  "order": 9,
  "status": "pending",
  "completedAt": null,
  "createdAt": "2025-11-06T17:00:00Z",
  "updatedAt": "2025-11-06T17:00:00Z"
}
```

---

### GET /api/subtasks/{subtaskId}

Retrieves a specific subtask by ID.

**Authentication Required**: Yes

#### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| subtaskId | string | yes | Subtask UUID |

#### Response
**Status**: `200 OK`

```json
{
  "id": "subtask-1",
  "taskId": "task-456",
  "title": "Research healthcare ML applications",
  "description": "Research and compile 10-15 recent applications of ML in healthcare",
  "scheduledDate": "2025-11-07",
  "estimatedDuration": 120,
  "order": 1,
  "status": "completed",
  "completedAt": "2025-11-07T16:30:00Z",
  "createdAt": "2025-11-06T16:45:00Z",
  "updatedAt": "2025-11-07T16:30:00Z"
}
```

---

### PUT /api/subtasks/{subtaskId}

Updates a subtask (full replacement).

**Authentication Required**: Yes

#### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| subtaskId | string | yes | Subtask UUID |

#### Request Body

**Content-Type**: `application/json`

```json
{
  "title": "Deep research on ML in diagnostics",
  "description": "Focus specifically on diagnostic applications with case studies",
  "scheduledDate": "2025-11-07",
  "estimatedDuration": 150
}
```

#### Response
**Status**: `200 OK`

```json
{
  "id": "subtask-1",
  "taskId": "task-456",
  "title": "Deep research on ML in diagnostics",
  "description": "Focus specifically on diagnostic applications with case studies",
  "scheduledDate": "2025-11-07",
  "estimatedDuration": 150,
  "order": 1,
  "status": "pending",
  "completedAt": null,
  "createdAt": "2025-11-06T16:45:00Z",
  "updatedAt": "2025-11-06T17:10:00Z"
}
```

---

### DELETE /api/subtasks/{subtaskId}

Deletes a subtask.

**Authentication Required**: Yes

#### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| subtaskId | string | yes | Subtask UUID |

#### Response
**Status**: `204 No Content`

---

### POST /api/subtasks/{subtaskId}/complete

Marks a subtask as completed.

**Authentication Required**: Yes

#### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| subtaskId | string | yes | Subtask UUID |

#### Response
**Status**: `200 OK`

```json
{
  "id": "subtask-2",
  "taskId": "task-456",
  "title": "Create paper outline",
  "description": "Develop detailed outline with sections and key points",
  "scheduledDate": "2025-11-08",
  "estimatedDuration": 90,
  "order": 2,
  "status": "completed",
  "completedAt": "2025-11-08T15:45:00Z",
  "createdAt": "2025-11-06T16:45:00Z",
  "updatedAt": "2025-11-08T15:45:00Z"
}
```

---

## Calendar

### GET /api/calendar/weekly

Retrieves tasks organized in a weekly calendar view.

**Authentication Required**: Yes

#### Query Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| startDate | date | no | Start of week (ISO 8601 date). Defaults to current week Monday |

#### Response
**Status**: `200 OK`

```json
{
  "weekStart": "2025-11-03",
  "weekEnd": "2025-11-09",
  "days": [
    {
      "date": "2025-11-03",
      "dayOfWeek": "monday",
      "subtasks": [
        {
          "id": "subtask-5",
          "taskId": "task-456",
          "title": "Write section 1",
          "scheduledDate": "2025-11-03",
          "estimatedDuration": 120,
          "status": "pending",
          "task": {
            "id": "task-456",
            "title": "Research Paper on AI in Healthcare",
            "dueDate": "2025-11-22T23:59:59Z"
          }
        }
      ],
      "totalEstimatedMinutes": 120,
      "completedSubtasks": 0,
      "totalSubtasks": 1
    },
    {
      "date": "2025-11-04",
      "dayOfWeek": "tuesday",
      "subtasks": [],
      "totalEstimatedMinutes": 0,
      "completedSubtasks": 0,
      "totalSubtasks": 0
    }
  ]
}
```

---

### GET /api/calendar/monthly

Retrieves tasks organized in a monthly calendar view.

**Authentication Required**: Yes

#### Query Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| year | number | no | Year (defaults to current year) |
| month | number | no | Month (1-12, defaults to current month) |

#### Response
**Status**: `200 OK`

```json
{
  "year": 2025,
  "month": 11,
  "monthName": "November",
  "weeks": [
    {
      "weekNumber": 44,
      "days": [
        {
          "date": "2025-11-01",
          "dayOfWeek": "saturday",
          "isCurrentMonth": true,
          "subtaskCount": 0,
          "completedSubtaskCount": 0,
          "hasDeadline": false
        },
        {
          "date": "2025-11-02",
          "dayOfWeek": "sunday",
          "isCurrentMonth": true,
          "subtaskCount": 0,
          "completedSubtaskCount": 0,
          "hasDeadline": false
        }
      ]
    }
  ],
  "totalSubtasks": 25,
  "completedSubtasks": 8,
  "tasksDue": 2
}
```

---

### GET /api/calendar/tasks

Retrieves all tasks and subtasks within a date range.

**Authentication Required**: Yes

#### Query Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| startDate | date | yes | Start date (ISO 8601 format) |
| endDate | date | yes | End date (ISO 8601 format) |

#### Response
**Status**: `200 OK`

```json
{
  "startDate": "2025-11-01",
  "endDate": "2025-11-30",
  "tasks": [
    {
      "id": "task-456",
      "title": "Research Paper on AI in Healthcare",
      "dueDate": "2025-11-22T23:59:59Z",
      "status": "in_progress",
      "subtasks": [
        {
          "id": "subtask-1",
          "title": "Research healthcare ML applications",
          "scheduledDate": "2025-11-07",
          "status": "completed"
        },
        {
          "id": "subtask-2",
          "title": "Create paper outline",
          "scheduledDate": "2025-11-08",
          "status": "pending"
        }
      ]
    }
  ]
}
```

---

## Streaks

### GET /api/streaks/current

Retrieves the user's current completion streak information.

**Authentication Required**: Yes

#### Response
**Status**: `200 OK`

```json
{
  "userId": "123e4567-e89b-12d3-a456-426614174000",
  "currentStreak": 7,
  "longestStreak": 15,
  "lastCompletionDate": "2025-11-06",
  "streakStartDate": "2025-10-31",
  "totalTasksCompleted": 42,
  "totalSubtasksCompleted": 156
}
```

#### Field Descriptions

- `currentStreak`: Consecutive days with completed subtasks
- `longestStreak`: Best streak ever achieved
- `lastCompletionDate`: Most recent day with completed subtasks
- `streakStartDate`: When current streak began
- `totalTasksCompleted`: All-time completed tasks count
- `totalSubtasksCompleted`: All-time completed subtasks count

---

### GET /api/streaks/history

Retrieves detailed completion history for streak calculation.

**Authentication Required**: Yes

#### Query Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| days | number | no | Number of days to retrieve (default: 30, max: 365) |

#### Response
**Status**: `200 OK`

```json
{
  "userId": "123e4567-e89b-12d3-a456-426614174000",
  "currentStreak": 7,
  "history": [
    {
      "date": "2025-11-06",
      "tasksCompleted": 1,
      "subtasksCompleted": 3,
      "totalEstimatedMinutes": 240,
      "actualMinutesWorked": 220
    },
    {
      "date": "2025-11-05",
      "tasksCompleted": 0,
      "subtasksCompleted": 2,
      "totalEstimatedMinutes": 150,
      "actualMinutesWorked": 165
    },
    {
      "date": "2025-11-04",
      "tasksCompleted": 0,
      "subtasksCompleted": 4,
      "totalEstimatedMinutes": 180,
      "actualMinutesWorked": 175
    }
  ]
}
```

---

## Reminders

### GET /api/reminders/today

Retrieves all tasks and subtasks due or scheduled for today.

**Authentication Required**: Yes

#### Response
**Status**: `200 OK`

```json
{
  "date": "2025-11-06",
  "scheduledSubtasks": [
    {
      "id": "subtask-10",
      "taskId": "task-456",
      "title": "Write section 2",
      "scheduledDate": "2025-11-06",
      "estimatedDuration": 120,
      "status": "pending",
      "task": {
        "id": "task-456",
        "title": "Research Paper on AI in Healthcare",
        "dueDate": "2025-11-22T23:59:59Z"
      }
    }
  ],
  "tasksDueToday": [
    {
      "id": "task-789",
      "title": "Math Homework Set 5",
      "dueDate": "2025-11-06T23:59:59Z",
      "status": "in_progress",
      "subtaskCount": 3,
      "completedSubtaskCount": 2
    }
  ],
  "totalScheduledSubtasks": 1,
  "totalTasksDue": 1,
  "totalEstimatedMinutes": 120
}
```

---

### GET /api/reminders

Retrieves tasks and subtasks for a specific date.

**Authentication Required**: Yes

#### Query Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| date | date | yes | Date in ISO 8601 format (YYYY-MM-DD) |

#### Response
**Status**: `200 OK`

```json
{
  "date": "2025-11-10",
  "scheduledSubtasks": [
    {
      "id": "subtask-15",
      "taskId": "task-456",
      "title": "Write conclusion",
      "scheduledDate": "2025-11-10",
      "estimatedDuration": 90,
      "status": "pending",
      "task": {
        "id": "task-456",
        "title": "Research Paper on AI in Healthcare",
        "dueDate": "2025-11-22T23:59:59Z"
      }
    }
  ],
  "tasksDueToday": [],
  "totalScheduledSubtasks": 1,
  "totalTasksDue": 0,
  "totalEstimatedMinutes": 90
}
```

---

## Google Calendar Sync

### POST /api/google-calendar/connect

Initiates Google Calendar OAuth flow for calendar synchronization.

**Authentication Required**: Yes

#### Response
**Status**: `200 OK`

```json
{
  "authorizationUrl": "https://accounts.google.com/o/oauth2/v2/auth?client_id=...",
  "state": "random-state-token"
}
```

**Note**: Client should redirect user to `authorizationUrl` to complete OAuth flow.

---

### DELETE /api/google-calendar/disconnect

Disconnects Google Calendar synchronization.

**Authentication Required**: Yes

#### Response
**Status**: `204 No Content`

**Note**: This will revoke calendar access and stop syncing tasks to Google Calendar.

---

### POST /api/google-calendar/sync

Manually triggers synchronization between Saturn-X tasks and Google Calendar.

**Authentication Required**: Yes

#### Request Body

**Content-Type**: `application/json`

```json
{
  "direction": "both"
}
```

#### Field Descriptions

- `direction`: Sync direction - `to_google` (push to Google), `from_google` (pull from Google), `both` (bidirectional)

#### Response
**Status**: `200 OK`

```json
{
  "syncedAt": "2025-11-06T18:00:00Z",
  "eventsCreated": 5,
  "eventsUpdated": 2,
  "eventsDeleted": 0,
  "tasksUpdated": 1,
  "conflicts": []
}
```

---

### POST /api/google-calendar/webhook

Webhook endpoint for Google Calendar to notify of changes.

**Authentication Required**: No (uses webhook verification)

#### Headers

| Name | Type | Required | Description |
|------|------|----------|-------------|
| X-Goog-Channel-ID | string | yes | Channel ID from subscription |
| X-Goog-Resource-State | string | yes | State of resource: `sync`, `exists`, `not_exists` |
| X-Goog-Resource-ID | string | yes | Opaque ID for watched resource |

#### Response
**Status**: `200 OK`

```json
{
  "received": true,
  "processedAt": "2025-11-06T18:05:00Z"
}
```

**Note**: This endpoint processes calendar change notifications and triggers background sync.

---

## Data Models

### User
```json
{
  "id": "uuid",
  "email": "string",
  "name": "string",
  "picture": "string (url)",
  "provider": "google | github",
  "createdAt": "datetime",
  "updatedAt": "datetime"
}
```

### Preferences
```json
{
  "id": "uuid",
  "userId": "uuid",
  "priorityMode": "hardest_first | easiest_first | earliest_deadline | manual",
  "workingHoursStart": "string (HH:mm)",
  "workingHoursEnd": "string (HH:mm)",
  "preferredSessionDuration": "number (minutes)",
  "breakDuration": "number (minutes)",
  "daysOfWeek": ["monday", "tuesday", ...],
  "timezone": "string (IANA timezone)",
  "createdAt": "datetime",
  "updatedAt": "datetime"
}
```

### Task
```json
{
  "id": "uuid",
  "userId": "uuid",
  "title": "string",
  "description": "string",
  "dueDate": "datetime",
  "estimatedHours": "number",
  "difficultyLevel": "easy | medium | hard",
  "status": "pending | in_progress | completed",
  "completedAt": "datetime | null",
  "fileUrl": "string (url) | null",
  "createdAt": "datetime",
  "updatedAt": "datetime",
  "daysRemaining": "number",
  "subtaskCount": "number",
  "completedSubtaskCount": "number"
}
```

### Subtask
```json
{
  "id": "uuid",
  "taskId": "uuid",
  "title": "string",
  "description": "string",
  "scheduledDate": "date (YYYY-MM-DD)",
  "estimatedDuration": "number (minutes)",
  "order": "number",
  "status": "pending | completed",
  "completedAt": "datetime | null",
  "createdAt": "datetime",
  "updatedAt": "datetime"
}
```

---

## Notes

- All timestamps use ISO 8601 format with UTC timezone
- File uploads for tasks support: PDF, DOCX, TXT, MD (max 10MB)
- AI task generation considers user preferences for scheduling
- Google Calendar sync requires calendar.events OAuth scope
- Streak calculation: A day counts if at least one subtask is completed
- JWT tokens expire after 1 hour; use refresh token to obtain new access token

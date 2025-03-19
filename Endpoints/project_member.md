# Project Members API Documentation

---

## Table of Contents
1. [Add Project Member](#add-project-member)
2. [Get Project Members](#get-project-members)
3. [Update Member Role](#update-member-role)
4. [Remove Project Member](#remove-project-member)

---

## Add Project Member
- **Endpoint**: `POST /projects/{project_uuid}/members`
- **Method**: `POST`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Adds a new member to a project. Only the project owner can perform this action.
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
- **Request Body** (JSON):
  ```json
  {
    "email": "member@example.com",
    "role": "editor"
  }
  ```
  | Field    | Type       | Required | Description                                  |
  |----------|-----------|----------|----------------------------------------------|
  | `email`  | string (email) | Required | Email of the user being added as a member.   |
  | `role`   | "owner"\|"editor"\|"viewer" | Optional (defaults to "editor") | Desired role for this user in the project.  |

- **Response**: `201 Created`  
  Returns a **ProjectMemberResponse** object:
  ```json
  {
    "name": "Jane Smith",
    "email": "member@example.com",
    "role": "editor",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-05T12:34:56.789Z"
  }
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: Only the project owner can add members.
  - `404 Not Found`: Project or current user not found.
  - `400 Bad Request`: Trying to add oneself, user already an owner, or other validation issues.
  - `500 Internal Server Error`: Database or other internal error.

---

## Get Project Members
- **Endpoint**: `GET /projects/{project_uuid}/members`
- **Method**: `GET`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Retrieves all members of a project.  
  If the project is private, only the owner or a member can view members.
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
- **Response**: `200 OK`  
  Returns a list of **ProjectMemberResponse** objects:
  ```json
  [
    {
      "name": "John Doe",
      "email": "john.doe@example.com",
      "role": "owner",
      "created_at": "2023-10-05T12:34:56.789Z",
      "updated_at": "2023-10-05T12:34:56.789Z"
    },
    {
      "name": "Jane Smith",
      "email": "jane.smith@example.com",
      "role": "editor",
      "created_at": "2023-10-05T12:34:56.789Z",
      "updated_at": "2023-10-05T12:34:56.789Z"
    }
  ]
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: Attempting to view members of a private project without access.
  - `404 Not Found`: Project not found.
  - `400 Bad Request`: General error or malformed parameters.
  - `500 Internal Server Error`: Database or other internal error.

---

## Update Member Role
- **Endpoint**: `PATCH /projects/{project_uuid}/members/{email}`
- **Method**: `PATCH`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Updates a project member’s role by specifying their email. Only the project owner can perform this action.
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
  - `email` (string): The email address of the member to update.
- **Request Body** (JSON):
  ```json
  {
    "role": "viewer"
  }
  ```
  | Field  | Type        | Required | Description                                   |
  |--------|------------|----------|-----------------------------------------------|
  | `role` | "editor"\|"viewer" | Required | The new role for the project member (cannot change an owner to something else). |

- **Response**: `200 OK`  
  Returns an updated **ProjectMemberResponse** object:
  ```json
  {
    "name": "Jane Smith",
    "email": "jane.smith@example.com",
    "role": "viewer",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-06T09:12:45.321Z"
  }
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: Only the project owner can update member roles.
  - `404 Not Found`: Project or member not found.
  - `400 Bad Request`: Invalid role assignment or other validation issue.
  - `500 Internal Server Error`: Database or other internal error.

---

## Remove Project Member
- **Endpoint**: `DELETE /projects/{project_uuid}/members/{email}`
- **Method**: `DELETE`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Removes a member from the project by specifying their email. Only the project owner can remove members.
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
  - `email` (string): The email address of the member to remove.
- **Response**: `204 No Content`  
  Returns no data if the removal is successful.
  
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: Only the project owner can remove members.
  - `404 Not Found`: Project or member not found.
  - `400 Bad Request`: Attempted to remove oneself (the owner) or other validation issue.
  - `500 Internal Server Error`: Database or other internal error.

---

## Summary of Endpoints

| Method | Endpoint                                               | Description                                              |
|--------|--------------------------------------------------------|----------------------------------------------------------|
| POST   | `/projects/{project_uuid}/members`                     | **Add Project Member** – only owner can do it            |
| GET    | `/projects/{project_uuid}/members`                     | **Get Project Members** – must have access to project    |
| PATCH  | `/projects/{project_uuid}/members/{email}`            | **Update Member Role** – only owner can do it            |
| DELETE | `/projects/{project_uuid}/members/{email}`            | **Remove Project Member** – only owner can do it         |

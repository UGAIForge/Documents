# Projects API Documentation

---

## Table of Contents
1. [Create Project](#create-project)
2. [Get Paginated Projects](#get-paginated-projects)
3. [Get Project with Versions](#get-project-with-versions)
4. [Update Project](#update-project)
5. [Delete Project](#delete-project)

---

## Create Project
- **Endpoint**: `POST /projects`
- **Method**: `POST`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Creates a new project and an initial project version (version 1).
- **Request Body** (JSON):
  ```json
  {
    "name": "Project Name",
    "visibility": "private"
  }
  ```
  | Field         | Type          | Required | Description                                                  |
  |---------------|---------------|----------|--------------------------------------------------------------|
  | `name`        | string        | Required | Project name.                                                |
  | `visibility`  | "private"\|"public" | Required | Defaults to `private` if not provided.                       |

- **Response**: `201 Created`  
  Returns a **ProjectWithVersionResponse** object, including the initial version and members:
  ```json
  {
    "id": 1,
    "uuid": "1234567890abcdef1234567890abcdef",
    "name": "Project Name",
    "user_uuid": "abcdef1234567890abcdef1234567890",
    "visibility": "private",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-05T12:34:56.789Z",
    "versions": [
      {
        "id": 1,
        "uuid": "abcdefabcdefabcdefabcdefabcdefab",
        "project_id": 1,
        "version_number": 1,
        "tag": "Initial version",
        "created_at": "2023-10-05T12:34:56.789Z"
      }
    ],
    "members": [
      {
        "name": "John Doe",
        "email": "john.doe@example.com",
        "role": "owner",
        "created_at": "2023-10-05T12:34:56.789Z",
        "updated_at": "2023-10-05T12:34:56.789Z"
      }
    ]
  }
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `404 Not Found`: If the current user (from token) cannot be found.
  - `400 Bad Request`: General error, including invalid input data.
  - `500 Internal Server Error`: Database or other internal error.

---

## Get Paginated Projects
- **Endpoint**: `GET /projects`
- **Method**: `GET`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Returns a paginated list of projects in which the user is a member. Supports optional filtering by role.
- **Query Params**:
  - `page` (int): Page number (default `1`).
  - `page_size` (int): Number of items per page (default `10`, max `100`).
  - `role` ("owner"\|"editor"\|"viewer"): Optional. If provided, returns only projects where the user has that role. Otherwise returns all.
  
- **Response**: `200 OK`  
  Returns a **ProjectPaginationResponse** object:
  ```json
  {
    "items": [
      {
        "uuid": "1234567890abcdef1234567890abcdef",
        "name": "Project Name",
        "visibility": "private",
        "owner": "abcdef1234567890abcdef1234567890",
        "owner_name": "John Doe",
        "created_at": "2023-10-05T12:34:56.789Z",
        "updated_at": "2023-10-05T12:34:56.789Z",
        "role": "owner"
      }
    ],
    "total": 1,
    "page": 1,
    "page_size": 10,
    "total_pages": 1
  }
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `400 Bad Request`: General error (e.g., invalid query parameters).
  - `500 Internal Server Error`: Database or other internal error.

---

## Get Project with Versions
- **Endpoint**: `GET /projects/{project_uuid}`
- **Method**: `GET`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Fetches a single project by its UUID, including its versions and members.
- **URL Params**:
  - `project_uuid` (string): The UUID of the project to retrieve.
- **Response**: `200 OK`  
  Returns a **ProjectWithVersionResponse** object:
  ```json
  {
    "uuid": "1234567890abcdef1234567890abcdef",
    "name": "Project Name",
    "user_uuid": "abcdef1234567890abcdef1234567890",
    "visibility": "private",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-05T12:34:56.789Z",
    "versions": [
      {
        "id": 1,
        "uuid": "abcdefabcdefabcdefabcdefabcdefab",
        "project_id": 1,
        "version_number": 1,
        "tag": "Initial version",
        "created_at": "2023-10-05T12:34:56.789Z"
      },
      ...
    ],
    "members": [
      {
        "name": "John Doe",
        "email": "john.doe@example.com",
        "role": "owner",
        "created_at": "2023-10-05T12:34:56.789Z",
        "updated_at": "2023-10-05T12:34:56.789Z"
      },
      ...
    ]
  }
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: Attempting to view a private project that you do not own (or have access to).
  - `404 Not Found`: Project does not exist.
  - `400 Bad Request`: General error (e.g., malformed UUID).
  - `500 Internal Server Error`: Database or other internal error.

---

## Update Project
- **Endpoint**: `PATCH /projects/{project_uuid}`
- **Method**: `PATCH`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Partially updates a project (e.g., its name or visibility). Only the project owner can update it.
- **URL Params**:
  - `project_uuid` (string): The UUID of the project to update.
- **Request Body** (JSON):
  ```json
  {
    "name": "New Project Name",
    "visibility": "public"
  }
  ```
  | Field         | Type          | Required | Description                                |
  |---------------|---------------|----------|--------------------------------------------|
  | `name`        | string        | Optional | New project name.                          |
  | `visibility`  | "private"\|"public" | Optional | Change visibility of the project.           |

- **Response**: `200 OK`  
  Returns a **ProjectUpdateResponse** object:
  ```json
  {
    "uuid": "1234567890abcdef1234567890abcdef",
    "name": "New Project Name",
    "visibility": "public",
    "updated_at": "2023-10-06T14:22:33.789Z"
  }
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: Only the project owner can update the project.
  - `404 Not Found`: Project not found.
  - `400 Bad Request`: Invalid request data or other errors.
  - `500 Internal Server Error`: Database or other internal error.

---

## Delete Project
- **Endpoint**: `DELETE /projects/{project_uuid}`
- **Method**: `DELETE`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Deletes a project. Only the project owner can perform this action.
- **URL Params**:
  - `project_uuid` (string): The UUID of the project to delete.
- **Response**: `204 No Content`  
  Returns no data if deletion is successful.

- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: Only the project owner can delete the project.
  - `404 Not Found`: Project not found.
  - `400 Bad Request`: General error (e.g., malformed UUID).
  - `500 Internal Server Error`: Database or other internal error.

---

## Summary of Endpoints

| Method | Endpoint                     | Description                            |
|--------|------------------------------|----------------------------------------|
| POST   | `/projects`                  | **Create Project** – with initial version |
| GET    | `/projects`                  | **Get Paginated Projects** – for current user |
| GET    | `/projects/{project_uuid}`   | **Get Project with Versions**          |
| PATCH  | `/projects/{project_uuid}`   | **Update Project** – only owner        |
| DELETE | `/projects/{project_uuid}`   | **Delete Project** – only owner        |

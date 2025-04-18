# Project Versions API Documentation

---

## Table of Contents
1. [Get Project Version](#get-project-version)
2. [Create Project Version](#create-project-version)
3. [Update Project Version](#update-project-version)
4. [Delete Project Version](#delete-project-version)

---

## Get Project Version
- **Endpoint**: `GET /projects/{project_uuid}/versions/{version_number}`
- **Method**: `GET`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Fetches a specific version of a project, including the files (in that version).
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
  - `version_number` (integer): The version number to retrieve.
- **Response**: `200 OK`  
  Returns a **ProjectVersionWithFilesResponse** object:
  ```json
  {
    "version_number": 1,
    "tag": "Initial version",
    "updated_at": "2023-10-05T12:34:56.789Z",
    "uuid": "abcdef1234567890abcdef1234567890",
    "name": "My Project",
    "visibility": "private",
    "files": [
      {
        "uuid": "1234567890abcdef1234567890abcdef",
        "name": "document.txt",
        "type": "file",
        "parent_uuid": null,
        "project_uuid": "abcdef1234567890abcdef1234567890",
        "user_uuid": "abcdef1234567890abcdef1234567890",
        "created_at": "2023-10-05T12:34:56.789Z",
        "updated_at": "2023-10-05T12:34:56.789Z"
      },
      ...
    ]
  }
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: Attempting to view a private project you do not own (or have access to).
  - `404 Not Found`: Project or specified version does not exist.
  - `400 Bad Request`: Malformed request or other general errors.
  - `500 Internal Server Error`: Database or other internal error.

---

## Create Project Version
- **Endpoint**: `POST /projects/{project_uuid}/versions`
- **Method**: `POST`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Creates a new version for a project, automatically assigning it the next version number. Only the project owner can create new versions.
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
- **Request Body** (JSON):
  ```json
  {
    "version_number": 1,
    "tag": "New version tag"
  }
  ```
  | Field           | Type   | Required | Description                                |
  |-----------------|--------|----------|--------------------------------------------|
  | `version_number`| int    | Required | Source version number from which to branch or reference. |
  | `tag`           | string | Optional | Tag or description for the version.        |

- **Response**: `201 Created`  
  Returns a **ProjectVersionResponse** object:
  ```json
  {
    "version_number": 2,
    "tag": "New version tag",
    "updated_at": "2023-10-06T14:22:33.789Z"
  }
  ```
  > Note: The `version_number` in the response is the newly assigned version number.

- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: Only the project owner can create versions.
  - `404 Not Found`: Project not found or current user not found.
  - `400 Bad Request`: Invalid request data or other general errors.
  - `500 Internal Server Error`: Database or other internal error.

---

## Update Project Version
- **Endpoint**: `PATCH /projects/{project_uuid}/versions/{version_number}`
- **Method**: `PATCH`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Partially updates a project version’s properties (currently only the `tag` can be updated). Only the project owner can perform this action.
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
  - `version_number` (integer): The version number to update.
- **Request Body** (JSON):
  ```json
  {
    "tag": "Updated version tag"
  }
  ```
  | Field | Type   | Required | Description              |
  |-------|--------|----------|--------------------------|
  | `tag` | string | Optional | New tag or description.  |

- **Response**: `200 OK`  
  Returns a **ProjectVersionResponse** object:
  ```json
  {
    "version_number": 2,
    "tag": "Updated version tag",
    "updated_at": "2023-10-07T10:15:20.789Z"
  }
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: Only the project owner can update versions.
  - `404 Not Found`: Project or specified version not found.
  - `400 Bad Request`: Invalid request data or other general errors.
  - `500 Internal Server Error`: Database or other internal error.

---

## Delete Project Version
- **Endpoint**: `DELETE /projects/{project_uuid}/versions/{version_number}`
- **Method**: `DELETE`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Deletes a project version. Only the project owner can perform this action.
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
  - `version_number` (integer): The version number to delete.
- **Response**: `204 No Content`  
  Returns no data if the deletion is successful.

- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: Only the project owner can delete versions.
  - `404 Not Found`: Project or specified version not found.
  - `400 Bad Request`: Invalid request or general error.
  - `500 Internal Server Error`: Database or other internal error.

---

## Summary of Endpoints

| Method | Endpoint                                                             | Description                                        |
|--------|----------------------------------------------------------------------|----------------------------------------------------|
| GET    | `/projects/{project_uuid}/versions/{version_number}`                | **Get Project Version** – retrieve a specific version and its files |
| POST   | `/projects/{project_uuid}/versions`                                 | **Create Project Version** – create a new version  |
| PATCH  | `/projects/{project_uuid}/versions/{version_number}`                | **Update Project Version** – only owner can do it  |
| DELETE | `/projects/{project_uuid}/versions/{version_number}`                | **Delete Project Version** – only owner can do it  |

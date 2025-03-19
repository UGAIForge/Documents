# Files API Documentation

---

## Table of Contents
1. [Create File](#create-file)
2. [Get File Content](#get-file-content)
3. [Update File Content (Partial)](#update-file-content-partial)
4. [Update File Content (Full)](#update-file-content-full)
5. [Update File Metadata](#update-file-metadata)
6. [Delete File](#delete-file)

---

## Create File
- **Endpoint**: `POST /projects/{project_uuid}/versions/{version_number}/files`
- **Method**: `POST`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Creates a new file (or folder) under a specific project version. A new file version record is also created (with empty or default content).
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
  - `version_number` (integer): The version number of the project where the file should be created.
- **Request Body** (JSON):
  ```json
  {
    "name": "my_document",
    "type": "file",
    "parent_uuid": "abcdef1234567890abcdef1234567890" // optional
  }
  ```
  | Field         | Type         | Required | Description                                                                              |
  |---------------|-------------|----------|------------------------------------------------------------------------------------------|
  | `name`        | string      | Required | The name of the file or folder.                                                         |
  | `type`        | "file"\|"folder" | Required | Indicates whether this is a file or a folder.                                            |
  | `parent_uuid` | string (UUID) | Optional | The UUID of the parent folder, if applicable. Null or omitted if creating at root level. |

- **Response**: `201 Created`  
  Returns a **FileResponse** object:
  ```json
  {
    "uuid": "abcdef1234567890abcdef1234567890",
    "name": "my_document",
    "type": "file",
    "parent_uuid": "abcdef1234567890abcdef1234567890",
    "project_uuid": "1234567890abcdef1234567890abcdef",
    "user_uuid": "abcdef1234567890abcdef1234567890",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-05T12:34:56.789Z"
  }
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `404 Not Found`: Project or user not found; specified parent folder not found.
  - `400 Bad Request`: General error (e.g., validation issues, etc.).
  - `500 Internal Server Error`: Database or other internal error.

---

## Get File Content
- **Endpoint**: `GET /projects/{project_uuid}/versions/{version_number}/files/{file_uuid}`
- **Method**: `GET`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Retrieves the content of a file in a particular project version. (Folders typically have no content.)
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
  - `version_number` (integer): The version number of the project.
  - `file_uuid` (string): The UUID of the file to retrieve content for.
- **Response**: `200 OK`  
  Returns a **FileVersionResponse** object:
  ```json
  {
    "file_uuid": "abcdef1234567890abcdef1234567890",
    "content": {
      "someKey": "someValue"
    },
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-05T12:34:56.789Z"
  }
  ```
  - For folders, `content` may be empty or omitted depending on the backend logic.

- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: Attempting to access a private project without proper permissions.
  - `404 Not Found`: Project, version, or file does not exist.
  - `400 Bad Request`: General error (e.g., malformed parameters).
  - `500 Internal Server Error`: Database or other internal error.

---

## Update File Content (Partial)
- **Endpoint**: `PATCH /projects/{project_uuid}/versions/{version_number}/files/{file_uuid}/content/partial`
- **Method**: `PATCH`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Partially updates the file’s JSON content by specifying a path (array of keys) and a new value for that path.  
  For example, if the content is:
  ```json
  {
    "data": {
      "metadata": {
        "version": "1.0"
      }
    }
  }
  ```
  and you PATCH with `path=["data","metadata","version"]` and `value="1.1"`, the `version` is updated to `"1.1"`.
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
  - `version_number` (integer): The version number of the project.
  - `file_uuid` (string): The UUID of the file to update.
- **Request Body** (JSON):
  ```json
  {
    "path": ["data", "metadata", "version"],
    "value": "1.1"
  }
  ```
  | Field     | Type            | Required | Description                                                 |
  |-----------|-----------------|----------|-------------------------------------------------------------|
  | `path`    | string[]        | Required | A list of keys defining the JSON path to update.           |
  | `value`   | any (JSON data) | Required | The new value to set at the specified path.                 |

- **Response**: `200 OK`  
  Returns a **FileContentPartialResponse** object:
  ```json
  {
    "path": ["data", "metadata", "version"],
    "value": "1.1",
    "updated_at": "2023-10-05T12:40:56.789Z"
  }
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `404 Not Found`: Project, version, user, or file not found.
  - `400 Bad Request`: Could not update the content (e.g., the path is invalid).
  - `500 Internal Server Error`: Database or other internal error.

---

## Update File Content (Full)
- **Endpoint**: `PATCH /projects/{project_uuid}/versions/{version_number}/files/{file_uuid}/content/full`
- **Method**: `PATCH`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Replaces the entire file content with a new JSON object.
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
  - `version_number` (integer): The version number of the project.
  - `file_uuid` (string): The UUID of the file to update.
- **Request Body** (JSON):
  ```json
  {
    "content": {
      "title": "My Document",
      "body": "Lorem ipsum..."
    }
  }
  ```
  | Field       | Type            | Required | Description                            |
  |-------------|-----------------|----------|----------------------------------------|
  | `content`   | any (JSON data) | Required | The complete new content of the file.  |

- **Response**: `200 OK`  
  Returns a **FileContentResponse** object:
  ```json
  {
    "content": {
      "title": "My Document",
      "body": "Lorem ipsum..."
    }
  }
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `404 Not Found`: Project, version, user, or file not found.
  - `400 Bad Request`: Invalid new content or other update error.
  - `500 Internal Server Error`: Database or other internal error.

---

## Update File Metadata
- **Endpoint**: `PATCH /projects/{project_uuid}/versions/{version_number}/files/{file_uuid}`
- **Method**: `PATCH`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Partially updates file metadata (e.g., `name` or `parent_uuid`). Note that the file content is unchanged by this endpoint.
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
  - `version_number` (integer): The version number of the project.
  - `file_uuid` (string): The UUID of the file to update.
- **Request Body** (JSON):
  ```json
  {
    "name": "updated_document_name",
    "parent_uuid": null
  }
  ```
  | Field         | Type    | Required | Description                                                           |
  |---------------|---------|----------|-----------------------------------------------------------------------|
  | `name`        | string  | Optional | New file or folder name.                                              |
  | `parent_uuid` | string  | Optional | New parent folder UUID, or `null` to move it to the root of the project. |

- **Response**: `200 OK`  
  Returns an updated **FileResponse** object:
  ```json
  {
    "uuid": "abcdef1234567890abcdef1234567890",
    "name": "updated_document_name",
    "type": "file",
    "parent_uuid": null,
    "project_uuid": "1234567890abcdef1234567890abcdef",
    "user_uuid": "abcdef1234567890abcdef1234567890",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-06T08:22:33.123Z"
  }
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: The file does not belong to the given project or no permission to modify.
  - `404 Not Found`: Project, version, or file not found.
  - `400 Bad Request`: Invalid metadata changes or other update error.
  - `500 Internal Server Error`: Database or other internal error.

---

## Delete File
- **Endpoint**: `DELETE /projects/{project_uuid}/versions/{version_number}/files/{file_uuid}`
- **Method**: `DELETE`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Deletes a file (or folder) from a project version. If a folder is deleted, all its children are also removed (cascading delete).
- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
  - `version_number` (integer): The version number of the project.
  - `file_uuid` (string): The UUID of the file or folder to delete.
- **Response**: `204 No Content`  
  Returns no data if the deletion is successful.

- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `403 Forbidden`: The file does not belong to this project or no permission.
  - `404 Not Found`: File or project version not found.
  - `400 Bad Request`: General error.
  - `500 Internal Server Error`: Database or other internal error.

---

## Summary of Endpoints

| Method | Endpoint                                                                                      | Description                                   |
|--------|-----------------------------------------------------------------------------------------------|-----------------------------------------------|
| POST   | `/projects/{project_uuid}/versions/{version_number}/files`                                   | **Create File** – add a file or folder        |
| GET    | `/projects/{project_uuid}/versions/{version_number}/files/{file_uuid}`                       | **Get File Content** – retrieve version’s file|
| PATCH  | `/projects/{project_uuid}/versions/{version_number}/files/{file_uuid}/content/partial`       | **Update File Content (Partial)**             |
| PATCH  | `/projects/{project_uuid}/versions/{version_number}/files/{file_uuid}/content/full`          | **Update File Content (Full)**                |
| PATCH  | `/projects/{project_uuid}/versions/{version_number}/files/{file_uuid}`                       | **Update File Metadata** (name, parent, etc.) |
| DELETE | `/projects/{project_uuid}/versions/{version_number}/files/{file_uuid}`                       | **Delete File**                               |

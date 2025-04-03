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
    "parent_uuid": "abcdef1234567890abcdef1234567890" 
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
      "agentName": "agent name",
      "description": "",
      "context": {},
      "workers": {}
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
- **Description**: Partially updates the file’s JSON content by specifying a path (array of keys or indices) and a new value for that path.

### Example: Update a Field in a Nested Object
Suppose the original file content is:
```json
{
  "workers": [
    {
      "name": "worker name",
      "SPL": {
        "workFlows": [
          {
            "flowContent": [
              {
                "type": "command",
                "content": "original command"
              }
            ]
          }
        ]
      }
    }
  ]
}
```

Then you PATCH with the request body:
```json
{
  "path": ["workers", 0, "SPL", "workFlows", 0, "flowContent", 0, "content"],
  "value": "updated command"
}
```

The updated file content becomes:
```json
{
  "workers": [
    {
      "name": "worker name",
      "SPL": {
        "workFlows": [
          {
            "flowContent": [
              {
                "type": "command",
                "content": "updated command"
              }
            ]
          }
        ]
      }
    }
  ]
}
```

The response is:
```json
{
  "path": ["workers", 0, "SPL", "workFlows", 0, "flowContent", 0, "content"],
  "value": "updated command",
  "updated_at": "2025-03-21T15:32:56.789Z"
}
```

---

### How to Add an Item to a List

To **append a new item to an existing list**, provide a path where the last element is equal to the current list's length.

#### Example: Append a new `command` to `flowContent`

Original content:
```json
{
  "workers": [
    {
      "SPL": {
        "workFlows": [
          {
            "flowContent": [
              {
                "type": "command",
                "content": "command A"
              }
            ]
          }
        ]
      }
    }
  ]
}
```

Request body:
```json
{
  "path": ["workers", 0, "SPL", "workFlows", 0, "flowContent", 1],
  "value": {
    "type": "command",
    "content": "command B"
  }
}
```

Updated content:
```json
{
  "workers": [
    {
      "SPL": {
        "workFlows": [
          {
            "flowContent": [
              {
                "type": "command",
                "content": "command A"
              },
              {
                "type": "command",
                "content": "command B"
              }
            ]
          }
        ]
      }
    }
  ]
}
```

Response:
```json
{
  "path": ["workers", 0, "SPL", "workFlows", 0, "flowContent", 1],
  "value": {
    "type": "command",
    "content": "command B"
  },
  "updated_at": "2025-03-21T15:35:22.123Z"
}
```

> ⚠️ **Important**: When adding to a list, **you must provide the exact next index** — for example, use `2` if the list already has two items. The system does not support automatic appending without the correct index.

---

### How to Add an Item to a Dictionary

To **add a new key-value pair** to an existing dictionary (object), specify the path to the dictionary and the new key as the final element. If the key does not yet exist, it will be created.

#### Example: Add a new config key to `settings`

Suppose the original file content is:
```json
{
  "workers": [
    {
      "SPL": {
        "settings": {
          "env": "development"
        }
      }
    }
  ]
}
```

Request body:
```json
{
  "path": ["workers", 0, "SPL", "settings", "replicas"],
  "value": 2
}
```

Updated content:
```json
{
  "workers": [
    {
      "SPL": {
        "settings": {
          "env": "development",
          "replicas": 2
        }
      }
    }
  ]
}
```

Response:
```json
{
  "path": ["workers", 0, "SPL", "settings", "replicas"],
  "value": 2,
  "updated_at": "2025-03-21T16:00:10.456Z"
}
```

> ⚠️ **Important**: If the final key already exists, the value will be overwritten. If the key does not exist, it is created.

### How to Remove an Item in a List

Currently, there is **no direct "remove index" operation**. Instead, you **overwrite** the parent array with a new array that excludes the item you want removed.

#### Example: Remove the second `command` from `flowContent`

Suppose the original file content is:
```json
{
  "workers": [
    {
      "SPL": {
        "workFlows": [
          {
            "flowContent": [
              {
                "type": "command",
                "content": "command A"
              },
              {
                "type": "command",
                "content": "command B"
              }
            ]
          }
        ]
      }
    }
  ]
}
```

You want to remove the item at index `1` (which has `"content": "command B"`). First, **fetch** the list from your application side, remove the undesired item, and then **send the updated list** as the new `value`.

Request body:
```json
{
  "path": ["workers", 0, "SPL", "workFlows", 0, "flowContent"],
  "value": [
    {
      "type": "command",
      "content": "command A"
    }
  ]
}
```
- Here, `path` points to the **array** itself (not an individual element).
- `value` is the **entire array** after removing `"command B"`.

The updated content becomes:
```json
{
  "workers": [
    {
      "SPL": {
        "workFlows": [
          {
            "flowContent": [
              {
                "type": "command",
                "content": "command A"
              }
            ]
          }
        ]
      }
    }
  ]
}
```

Response:
```json
{
  "path": ["workers", 0, "SPL", "workFlows", 0, "flowContent"],
  "value": [
    {
      "type": "command",
      "content": "command A"
    }
  ],
  "updated_at": "2025-03-21T16:15:42.789Z"
}
```

> ⚠️ **Important**: The final element in `path` must be the array you’re replacing, and `value` must be the complete updated array. Any item not present in `value` is effectively removed.

---

### How to Remove a Key from a Dictionary

Similarly, to remove a key from a dictionary, **overwrite** the parent dictionary with a new dictionary that omits the unwanted key.

#### Example: Remove `"replicas"` from `settings`

Suppose the original file content is:
```json
{
  "workers": [
    {
      "SPL": {
        "settings": {
          "env": "development",
          "replicas": 2
        }
      }
    }
  ]
}
```

To remove `"replicas"`, you fetch the existing `settings` object, delete `"replicas"`, and then overwrite the `settings` key with the updated object.

Request body:
```json
{
  "path": ["workers", 0, "SPL", "settings"],
  "value": {
    "env": "development"
  }
}
```
- Here, `path` points to the **dictionary** object you are replacing.
- `value` is the **entire dictionary** minus the `"replicas"` key.

Updated content:
```json
{
  "workers": [
    {
      "SPL": {
        "settings": {
          "env": "development"
        }
      }
    }
  ]
}
```

Response:
```json
{
  "path": ["workers", 0, "SPL", "settings"],
  "value": {
    "env": "development"
  },
  "updated_at": "2025-03-21T16:17:09.123Z"
}
```

> ⚠️ **Important**: The final element in `path` is the **dictionary** you’re overwriting, and `value` must contain the remaining keys. Any omitted key is removed.
---

- **URL Params**:
  - `project_uuid` (string): The UUID of the project.
  - `version_number` (integer): The version number of the project.
  - `file_uuid` (string): The UUID of the file to update.

- **Request Body** (JSON):
  ```json
  {
    "path": ["workers", 0, "SPL", "workFlows", 0, "flowContent", 0, "content"],
    "value": "updated command"
  }
  ```
  | Field     | Type              | Required | Description                                                                 |
  |-----------|-------------------|----------|-----------------------------------------------------------------------------|
  | `path`    | (string \| int)[] | Required | A list of keys (for dicts) and/or indices (for lists) defining the JSON path to update. |
  | `value`   | any (JSON data)   | Required | The new value to set at the specified path.                                 |

- **Response**: `200 OK`  
  Returns a **FileContentPartialResponse** object:
  ```json
  {
    "path": ["workers", 0, "SPL", "workFlows", 0, "flowContent", 0, "content"],
    "value": "updated command",
    "updated_at": "2025-03-21T15:32:56.789Z"
  }
  ```

- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `404 Not Found`: Project, version, user, or file not found.
  - `400 Bad Request`: Could not update the content (e.g., the path is invalid, list index is out of bounds, or type mismatch).
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

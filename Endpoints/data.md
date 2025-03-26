# Data Endpoints API Documentation

---

## Table of Contents
1. [Create Data Entry](#create-data-entry)
2. [Get Data by File](#get-data-by-file)
3. [Get Data by UUID](#get-data-by-uuid)
4. [Get Data at Project Version](#get-data-at-project-version)
5. [Update Data](#update-data)
6. [Delete Data](#delete-data)
7. [Summary of Data Endpoints](#summary-of-data-endpoints)

---

## Create Data Entry

- **Endpoint**: `POST /data`
- **Method**: `POST`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Creates a new data entry, associated with a specific file.

- **Request Body** (JSON):
  ```json
  {
    "file_uuid": "abcdef1234567890abcdef1234567890",
    "name": "SomeData",
    "type": "someType",
    "description": "Optional detailed description",
    "dataschema": {
      "schemaKey": "schemaValue"
    },
    "datasource": {
      "sourceKey": "sourceValue"
    }
  }
  ```
  | Field         | Type               | Required | Description                                                                       |
  |---------------|--------------------|----------|-----------------------------------------------------------------------------------|
  | `file_uuid`   | string (UUID)     | **Yes**  | The UUID of the file this data is associated with.                                |
  | `name`        | string            | **Yes**  | A name for the data entry.                                                        |
  | `type`        | string            | **Yes**  | A general or specific type (e.g., "csv", "json", or a custom type in your system).|
  | `description` | string            | No       | A textual description for documentation purposes.                                 |
  | `dataschema`  | object (JSON)     | No       | JSON schema or structure defining the data format.                                |
  | `datasource`  | object (JSON)     | No       | JSON object describing the data’s source or metadata.                             |

- **Response**: `201 Created`  
  Returns a **DataResponse** object:
  ```json
  {
    "uuid": "1234567890abcdef1234567890abcdef",
    "file_uuid": "abcdef1234567890abcdef1234567890",
    "name": "SomeData",
    "type": "someType",
    "description": "Optional detailed description",
    "dataschema": {
      "schemaKey": "schemaValue"
    },
    "datasource": {
      "sourceKey": "sourceValue"
    },
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-05T12:34:56.789Z"
  }
  ```
- **Errors**:
  - `400 Bad Request`: Could not create data due to validation or other issues.
  - `500 Internal Server Error`: Database or other internal error.

---

## Get Data by File

- **Endpoint**: `GET /data`
- **Method**: `GET`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Retrieves a list of data entries for a single `file_uuid`.  

- **Query Parameters**:
  - **`file_uuid`** (string, required): The UUID of the file for which to retrieve data entries.

- **Response**: `200 OK`  
  Returns a **DataListResponse** object, containing a list of data UUIDs and names:
  ```json
  {
    "items": [
      {
        "uuid": "1234567890abcdef1234567890abcdef",
        "name": "SomeData"
      },
      {
        "uuid": "abcdefabcdefabcdefabcdefabcdefab",
        "name": "AnotherData"
      }
    ]
  }
  ```
- **Errors**:
  - `500 Internal Server Error`: Database or other internal error.

---

## Get Data by UUID

- **Endpoint**: `GET /data/{data_uuid}`
- **Method**: `GET`
- **Authentication**: Depends on your system’s needs (not shown here).
- **Description**: Fetch a single data entry by its UUID.

- **URL Params**:
  - `data_uuid` (string, UUID): The UUID of the data entry to retrieve.

- **Response**: `200 OK`  
  Returns a **DataResponse** object:
  ```json
  {
    "uuid": "1234567890abcdef1234567890abcdef",
    "file_uuid": "abcdef1234567890abcdef1234567890",
    "name": "SomeData",
    "type": "someType",
    "description": "Optional detailed description",
    "dataschema": {
      "schemaKey": "schemaValue"
    },
    "datasource": {
      "sourceKey": "sourceValue"
    },
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-06T09:45:22.789Z"
  }
  ```
- **Errors**:
  - `404 Not Found`: No data entry with the specified UUID exists.
  - `500 Internal Server Error`: Database or other internal error.

---

## Get Data at Project Version

- **Endpoint**: `GET /data/at`
- **Method**: `GET**
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Retrieves various **Data**, **Context**, or **Worker** items within a specific project version.  
  - **Data** items have a valid `uuid` and `type` of `"Data"`, with no `path`.  
  - **Context** or **Worker** items have no `uuid` (null), but a `path` array describing their location.

- **Query Parameters**:
  - `project_uuid` (string, required): The UUID of the project.
  - `version_number` (integer, required): The version number of the project.

- **Response**: `200 OK`  
  Returns a **DataAtResponse** object, containing a list of items with varying fields depending on their `type`:

  ```json
  {
    "items": [
      {
        "display_name": "Test Data 2",
        "type": "Data",
        "uuid": "data_IPRAYThdTXyL2Pv1yoxfXw",
        "path": null
      },
      {
        "display_name": "Test Agent.persona.personality",
        "type": "Context",
        "uuid": null,
        "path": [
          "Test Agent",
          "context",
          0,
          "content",
          0,
          "key"
        ]
      },
      {
        "display_name": "Test Agent.persona.background",
        "type": "Context",
        "uuid": null,
        "path": [
          "Test Agent",
          "context",
          0,
          "content",
          1,
          "key"
        ]
      },
      {
        "display_name": "Test Agent.audience.target_users",
        "type": "Context",
        "uuid": null,
        "path": [
          "Test Agent",
          "context",
          1,
          "content",
          0,
          "key"
        ]
      },
      {
        "display_name": "Test Agent.code_helper",
        "type": "Worker",
        "uuid": null,
        "path": [
          "Test Agent",
          "workers",
          0
        ]
      },
      {
        "display_name": "Test Agent.test_writer",
        "type": "Worker",
        "uuid": null,
        "path": [
          "Test Agent",
          "workers",
          1
        ]
      }
    ]
  }
  ```

  | Field               | Type             | Description                                                                          |
  |---------------------|-----------------|--------------------------------------------------------------------------------------|
  | `display_name`      | string          | A descriptive name to show in the UI.                                               |
  | `type`             | "Data"\|"Context"\|"Worker" | Type of the item. "Data" items have UUID, "Context"/"Worker" items have path only. |
  | `uuid`             | string \| null   | UUID for "Data" items only. null for "Context" or "Worker".                         |
  | `path`             | array \| null    | Path array for "Context" or "Worker" items. null for "Data" items.                  |

- **Errors**:
  - `403 Forbidden`: Attempting to view items in a private project without access.
  - `404 Not Found`: Project not found.
  - `400 Bad Request`: Other validation issues.
  - `500 Internal Server Error`: Database or other internal error.

---

## Update Data

- **Endpoint**: `PATCH /data/{data_uuid}`
- **Method**: `PATCH`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Partially updates an existing data entry. Only the fields provided in the request body will be changed.

- **URL Params**:
  - `data_uuid` (string, UUID): The UUID of the data entry to update.
- **Request Body** (JSON):
  ```json
  {
    "name": "UpdatedData",
    "type": "updatedType",
    "description": "New description",
    "dataschema": {
      "schemaKey": "newSchemaValue"
    },
    "datasource": {
      "sourceKey": "updatedSourceValue"
    }
  }
  ```
  | Field         | Type               | Required | Description                                                     |
  |---------------|--------------------|----------|-----------------------------------------------------------------|
  | `name`        | string            | No       | New name for the data entry.                                    |
  | `type`        | string            | No       | New type for the data entry.                                    |
  | `description` | string            | No       | Updated description.                                            |
  | `dataschema`  | object (JSON)     | No       | Updated schema definition.                                      |
  | `datasource`  | object (JSON)     | No       | Updated source/metadata object.                                 |

- **Response**: `200 OK`  
  Returns the updated **DataResponse** object:
  ```json
  {
    "uuid": "1234567890abcdef1234567890abcdef",
    "file_uuid": "abcdef1234567890abcdef1234567890",
    "name": "UpdatedData",
    "type": "updatedType",
    "description": "New description",
    "dataschema": {
      "schemaKey": "newSchemaValue"
    },
    "datasource": {
      "sourceKey": "updatedSourceValue"
    },
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-06T10:55:43.123Z"
  }
  ```
- **Errors**:
  - `404 Not Found`: Data entry with the given UUID does not exist.
  - `500 Internal Server Error`: Database or other internal error.

---

## Delete Data

- **Endpoint**: `DELETE /data/{data_uuid}`
- **Method**: `DELETE`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Deletes an existing data entry from the system.

- **URL Params**:
  - `data_uuid` (string, UUID): The UUID of the data entry to delete.

- **Response**: `204 No Content`  
  Returns no data if the deletion is successful.

- **Errors**:
  - `404 Not Found`: Data entry with the given UUID does not exist.
  - `500 Internal Server Error`: Database or other internal error.

---

## Summary of Data Endpoints

| Method | Endpoint             | Description                                                      |
|-------:|----------------------|------------------------------------------------------------------|
| **POST**   | `/data`                  | **Create Data Entry** – attach data to a file                      |
| **GET**    | `/data`                  | **Get Data by File** – list data for a specific file               |
| **GET**    | `/data/{data_uuid}`      | **Get Data by UUID** – fetch a single data record                  |
| **GET**    | `/data/at`               | **Get Data at Project Version** – retrieve Data/Context/Worker items|
| **PATCH**  | `/data/{data_uuid}`      | **Update Data** – partially modify a data entry                    |
| **DELETE** | `/data/{data_uuid}`      | **Delete Data** – remove a data entry                              |

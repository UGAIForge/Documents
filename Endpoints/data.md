# Data Endpoints API Documentation

---

## Table of Contents
1. [Create Data Entry](#create-data-entry)
2. [Get Data by Scope](#get-data-by-scope)
3. [Get Data by UUID](#get-data-by-uuid)
4. [Update Data](#update-data)
5. [Delete Data](#delete-data)
6. [Summary of Data Endpoints](#summary-of-data-endpoints)

---

## Create Data Entry

- **Endpoint**: `POST /data`
- **Method**: `POST`
- **Authentication**: Depends on your system’s needs (not shown here).  
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

## Get Data by Scope
- **Endpoint**: `GET /data`
- **Method**: `GET`
- **Authentication**: Depends on your system’s needs (not shown here).
- **Description**: Retrieves a list of data entries either by `file_uuid` **or** by `(project_uuid + version_number)`. Exactly one of these approaches should be used per request.

- **Query Parameters**:
  - **`file_uuid`** (string, optional): If provided, returns data entries associated with this file.
  - **`project_uuid`** (string, optional) & **`version_number`** (integer, optional): If provided together, returns data entries belonging to the specified project version.
  
  > **Important**:  
  > - You **cannot** specify both `file_uuid` **and** `(project_uuid + version_number)`.
  > - If `project_uuid` is supplied, `version_number` must be supplied as well (and vice versa).

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
  - `400 Bad Request`: If query parameters are invalid (e.g., both `file_uuid` and `project_uuid` are specified, or only one of `project_uuid`/`version_number` is given).
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

## Update Data
- **Endpoint**: `PATCH /data/{data_uuid}`
- **Method**: `PATCH`
- **Authentication**: Depends on your system’s needs (not shown here).
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
- **Authentication**: Depends on your system’s needs (not shown here).
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

| Method | Endpoint         | Description                                      |
|-------:|------------------|--------------------------------------------------|
| **POST**   | `/data`            | **Create Data Entry** – attach data to a file         |
| **GET**    | `/data`            | **Get Data by Scope** – filter by file or project version |
| **GET**    | `/data/{data_uuid}`| **Get Data by UUID** – fetch a single data record     |
| **PATCH**  | `/data/{data_uuid}`| **Update Data** – partially modify a data entry       |
| **DELETE** | `/data/{data_uuid}`| **Delete Data** – remove a data entry from the system |

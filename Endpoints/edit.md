# Edit Endpoints Documentation

---

## Table of Contents
1. [Create Item](#create-item)
2. [Get Edit List](#get-edit-list)
3. [Summary of Edit Endpoints](#summary-of-edit-endpoints)

---

## Create Item
- **Endpoint**: `POST /edit/item`
- **Method**: `POST`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Creates a new item in a file’s JSON content. The new item can be of type `persona`, `audience`, `constraints`, `concept`, or `worker`.

### Query Parameters
- **`file_uuid`** (string, required):  
  The UUID of the file where you want to add the new item.  
  Must be provided as a query parameter, e.g.:  
  ```
  /edit/item?file_uuid={file_uuid}
  ```

### Request Body (JSON)
```json
{
  "path": ["string", ...],
  "type": "persona" | "audience" | "constraints" | "concept" | "worker"
}
```
| Field   | Type                    | Required | Description                                                                      |
|---------|-------------------------|----------|----------------------------------------------------------------------------------|
| `path`  | string[] (array of keys) | **Yes**  | The path in JSON where the new item will be added.                               |
| `type`  | one of the listed enums | **Yes**  | Type of the new item. Accepted values are `"persona"`, `"audience"`, `"constraints"`, `"concept"`, or `"worker"`. |

### Response
```json
{
  "path": ["string", ...],
  "value": {}
}
```
| Field    | Type                  | Description                                                         |
|----------|-----------------------|---------------------------------------------------------------------|
| `path`   | string[]             | The final JSON path where the item was created.                     |
| `value`  | object (JSON)        | The newly created item in JSON form.                                |

---

### Examples

Below are some examples of how to use **Create Item** in different scenarios:

#### Add a Context

**Context** items typically go under a `context` key in your file content.

**Request**  
```plaintext
POST /edit/item?file_uuid=file_HuvSxlGCS0-hZlxMwOXwvg
```
```json
{
  "path": ["context"],
  "type": "persona"
}
```

**Response**  
```json
{
  "path": [
    "context",
    "context_wwRcMzJBRa2CeF2y9Y6TOA"
  ],
  "value": {
    "type": "persona",
    "content": {}
  }
}
```
This creates a new context entry with the path `["context","context_wwRcMzJBRa2CeF2y9Y6TOA"]` and type `"persona"`.

---

#### Add Content to a Context

Inside a context object (e.g., `persona`), you may want to add a key-value pair describing some detail.

**Request**  
```plaintext
POST /edit/item?file_uuid=file_HuvSxlGCS0-hZlxMwOXwvg
```
```json
{
  "path": [
    "context",
    "context_wwRcMzJBRa2CeF2y9Y6TOA",
    "content",
  ],
  "type": "persona"
}
```

**Response**  
```json
{
  "path": [
    "context",
    "context_wwRcMzJBRa2CeF2y9Y6TOA",
    "content",
    "persona_vGY1JJv4T1OdQzjRmfOydQ"
  ],
  "value": {
    "key": "",
    "value": ""
  }
}
```
A new sub-object is created under `"content"` with the generated key like `"persona_vGY1JJv4T1OdQzjRmfOydQ"`.

---

#### Add a Worker

Specify the `type` as `"worker"` and a suitable path, typically under `workers`.

**Request**  
```plaintext
POST /edit/item?file_uuid=file_HuvSxlGCS0-hZlxMwOXwvg
```
```json
{
  "path": ["workers"],
  "type": "worker"
}
```

**Response**  
```json
{
  "path": [
    "workers",
    "worker_Gvd_dQa3TPyJkitwzdOSCQ"
  ],
  "value": {
    "name": "",
    "description": "",
    "entry": false,
    "SPL": {
      "input": "",
      "output": "",
      "flows": []
    },
    "useCases": {},
    "code": {},
    "diagrams": {}
  }
}
```
A new worker object is created under `workers` at the generated key `"worker_Gvd_dQa3TPyJkitwzdOSCQ"`.

---

### Errors

- `404 Not Found`: File (via `file_uuid`) not found.
- `400 Bad Request`: Validation error (e.g., invalid path or type).
- `401 Unauthorized`: Missing or invalid bearer token.
- `500 Internal Server Error`: Other internal errors (e.g., database failure).

---

## Get Edit List
- **Endpoint**: `GET /edit/at`
- **Method**: `GET**
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Retrieves a list of **Data**, **Context**, or **Worker** items that can be edited within a specific project version.

### Query Parameters
- **`project_uuid`** (string, required):  
  The UUID of the project.
- **`version_number`** (integer, required):  
  The version number of the project.

### Response
```json
{
  "items": [
    {
      "display_name": "Some Data",
      "type": "Data",
      "reference": "ref:data:data_JSvFhnihR625jFmlzs2lYA"
    },
    {
      "display_name": "edit_test.json.persona.",
      "type": "Context",
      "reference": "ref:context:path_file_HuvSxlGCS0-hZlxMwOXwvg_context_context_..."
    },
    {
      "display_name": "edit_test.json.",
      "type": "Worker",
      "reference": "ref:worker:path_file_HuvSxlGCS0-hZlxMwOXwvg_workers_worker_..."
    }
  ]
}
```
Where each item in `items` is an **EditAtItem** containing:
- `display_name`: A user-friendly name for display.
- `type`: The item type (e.g., `"Data"`, `"Context"`, or `"Worker"`).
- `reference`: A unique reference string to further identify or manipulate the item.

### Example

Suppose you created:
1. A file named `edit_test.json`
2. Data entries: `"Test Data 1"`, `"Test Data 2"`
3. A context item
4. A worker

When you call:
```plaintext
GET /edit/at?project_uuid=project_bXBOgtA0QeucarLjCE8QCg&version_number=1
```
You might see a response like:
```json
{
  "items": [
    {
      "display_name": "Test Data 1",
      "type": "Data",
      "reference": "ref:data:data_JSvFhnihR625jFmlzs2lYA"
    },
    {
      "display_name": "Test Data 2",
      "type": "Data",
      "reference": "ref:data:data_kFODLQhkSSefaJ0zpIu0Gw"
    },
    {
      "display_name": "edit_test.json.persona.",
      "type": "Context",
      "reference": "ref:context:path_file_HuvSxlGCS0-hZlxMwOXwvg_context_context_wwRcMzJBRa2CeF2y9Y6TOA_content_persona_vGY1JJv4T1OdQzjRmfOydQ"
    },
    {
      "display_name": "edit_test.json.",
      "type": "Worker",
      "reference": "ref:worker:path_file_HuvSxlGCS0-hZlxMwOXwvg_workers_worker_Gvd_dQa3TPyJkitwzdOSCQ"
    }
  ]
}
```

### Errors

- `403 Forbidden`: Attempting to view items in a private project without access.
- `404 Not Found`: Project does not exist.
- `401 Unauthorized`: Missing or invalid bearer token.
- `400 Bad Request`: Other validation issues.
- `500 Internal Server Error`: General or database-related error.

---

## Summary of Edit Endpoints

| Method | Endpoint               | Description                                                         |
|-------:|------------------------|---------------------------------------------------------------------|
| **POST**   | `/edit/item`            | **Create Item** – add a `persona`, `audience`, `constraints`, `concept`, or `worker` to file content |
| **GET**    | `/edit/at`              | **Get Edit List** – retrieve Data/Context/Worker items in a project version |

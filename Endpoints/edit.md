# Edit Endpoints Documentation

---

## Table of Contents
1. [Create Item (`/edit/item`)](#create-item-edititem)
   - [Add a Context](#add-a-context)
   - [Add Content to a Context](#add-content-to-a-context)
   - [Add a Worker](#add-a-worker)
2. [Get Edit List (`/edit/at`)](#get-edit-list-editat)

---

## Create Item (`/edit/item`)

- **Endpoint**: `POST /edit/item`
- **Method**: `POST`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Creates a new item in a file’s JSON content. The new item can be of type `persona`, `audience`, `constraints`, `concept`, or `worker`.

### Query Parameters
- **`file_uuid`** (string, required):  
  The UUID of the file where you want to add the new item.  
  Must be provided as a query parameter, e.g.:  
  ```
  /edit/item?file_uuid=file_HuvSxlGCS0-hZlxMwOXwvg
  ```

### Request Body
```json
{
  "path": ["string", ...],
  "type": "persona" | "audience" | "constraints" | "concept" | "worker"
}
```
| Field   | Type                    | Required | Description                                               |
|---------|-------------------------|----------|-----------------------------------------------------------|
| `path`  | string[] (array of keys)| **Yes**  | The path in JSON where the new item will be added.        |
| `type`  | one of the listed enums | **Yes**  | Type of the new item (`persona`, `audience`, `constraints`, `concept`, or `worker`). |

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

### **Add a Context**

**Context** items typically go under a `context` key in your file content. For example:

**Request** (POST `/edit/item?file_uuid=file_HuvSxlGCS0-hZlxMwOXwvg`):
```json
{
  "path": ["context", "context_wwRcMzJBRa2CeF2y9Y6TOA"],
  "type": "persona"
}
```

**Response**:
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
This creates a new context entry with the specified path and type `persona`.

---

### **Add Content to a Context**

Inside a context object (e.g., `persona`), you may want to add a key-value pair describing some detail. For instance, adding a piece of persona content.

**Request** (POST `/edit/item?file_uuid=file_HuvSxlGCS0-hZlxMwOXwvg`):
```json
{
  "path": [
    "context",
    "context_wwRcMzJBRa2CeF2y9Y6TOA",
    "content",
    "persona_vGY1JJv4T1OdQzjRmfOydQ"
  ],
  "type": "persona"
}
```

**Response**:
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
Here, the system created a new sub-object (with a generated key like `"persona_vGY1JJv4T1OdQzjRmfOydQ"`) under `"content"`.

---

### **Add a Worker**

To add a **Worker** item, specify the `type` as `"worker"` and a suitable path, typically under `workers`.

**Request** (POST `/edit/item?file_uuid=file_HuvSxlGCS0-hZlxMwOXwvg`):
```json
{
  "path": ["workers", "worker_Gvd_dQa3TPyJkitwzdOSCQ"],
  "type": "worker"
}
```

**Response**:
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

---

## Get Edit List (`/edit/at`)

- **Endpoint**: `GET /edit/at`
- **Method**: `GET`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Retrieves a list of **Data**, **Context**, or **Worker** items that can be edited within a specific project version.

### Query Parameters
- **`project_uuid`** (string, required): The UUID of the project.
- **`version_number`** (integer, required): The version number of the project.

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
- `display_name`: A user-friendly name to show in the UI.
- `type`: The type of item (e.g., `"Data"`, `"Context"`, or `"Worker"`).
- `reference`: A unique reference string that can be used to locate or further manipulate the item.

### Example Workflow

1. **Create a file** and **attach data entries** (type: `Data`).
2. **Add a context** or **worker** to the file’s JSON content via the `/edit/item` endpoint.
3. **Call** `/edit/at?project_uuid=xxx&version_number=yyy` to see a combined list of:
   - Data entries from the file
   - Context items
   - Worker items
4. Each item in the list will have its own `display_name`, `type`, and `reference`.

**Sample** `GET /edit/at` response:
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

---

This documentation outlines how to:
- **Create** new items (`persona`, `audience`, `constraints`, `concept`, or `worker`) under file content via `/edit/item`.
- **Fetch** the consolidated edit list of data/context/worker items in a project version via `/edit/at`.

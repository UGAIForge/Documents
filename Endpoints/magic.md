# Magic WebSocket API Documentation

---

## Table of Contents
1. [Magic WebSocket (`/magic/ws`)](#magic-websocket-endpoint)

---

## Magic WebSocket (`/magic/ws`)

- **Endpoint**: `GET /magic/ws` → *upgrades to a WebSocket*
- **Method**: `GET` (with protocol upgrade)
- **Authentication**: **Query Parameter** &nbsp;`token=<access_token>` — same JWT/Bearer token used for REST endpoints.
- **Description**: Opens a persistent WebSocket that streams incremental LLM messages (`llm_response`) and tool‑execution results (`tool_result`) while “magic” operations modify an agent (SPL) file in real time.

---

### Query Parameters
| Param | Type   | Required | Allowed Values                         | Description                                                      |
|-------|--------|----------|----------------------------------------|------------------------------------------------------------------|
| `type` | string | **Yes** | `nl_to_spl` \| `gen_alt_exc` \| `data` | Chooses the magic operation (currently only `nl_to_spl` is live).|
| `token` | string | **Yes** | —                                      | Access token for the current user.                               |

**Connection example**

```text
ws://localhost:5000/magic/ws?type=nl_to_spl&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

### Client → Server Message  
Send exactly **one** JSON object immediately after the socket is accepted:

```json
{
  "project_uuid": "project_bXBOgtA0QeucarLjCE8QCg",
  "version_number": 1,
  "file_uuid": "file_HuvSxlGCS0-hZlxMwOXwvg"
}
```

| Field            | Type    | Required | Description                                    |
|------------------|---------|----------|------------------------------------------------|
| `project_uuid`   | string  | **Yes**  | Target project.                                |
| `version_number` | integer | **Yes**  | Project version to operate on.                 |
| `file_uuid`      | string  | **Yes**  | Agent/SPL JSON file that will be modified.     |

---

### Server → Client Events

Each frame is a JSON envelope:

```json
{
  "type": "llm_response" | "tool_result" | "error",
  "data": ...
}
```

| `type` | `data` shape & front‑end handling |
|--------|-----------------------------------|
| `llm_response` | Plain **string** — display as assistant text. |
| `tool_result` | ```json<br>{ "tool": "create_item", "result": {...} }``` or ```json<br>{ "tool": "update_file_content_partial", "result": {...} }```<br>Handle as follows:<br>• `create_item` → update the file per the **Create Item** spec (<https://github.com/UGAIForge/Documents/blob/main/Endpoints/edit.md#create-item>).<br>• `update_file_content_partial` → apply each `updates` entry per **Update File Content (Partial)** (<https://github.com/UGAIForge/Documents/blob/main/Endpoints/file.md#update-file-content-partial>). |
| `error` | **String** — show to user; stop progress indicators. |

---

### Errors & Close Codes
| Scenario                                   | Behaviour / Code |
|--------------------------------------------|------------------|
| Missing / invalid `token`                  | Close **1008 (POLICY_VIOLATION)** |
| Unsupported `type` value                   | Sends an `"error"` frame, then closes **1000** |
| Internal server exception                  | Sends `"error"` frame (if possible) then closes **1011 (INTERNAL_ERROR)** |
| Client closes socket                       | Server stops processing gracefully |

---

### Summary of WebSocket Interaction

| Step | Actor  | Payload / Event                                      | Purpose                                          |
|----: |--------|------------------------------------------------------|--------------------------------------------------|
| 1    | Client | **Connect** `ws://.../magic/ws?type=…&token=…`        | Opens socket; server validates parameters.       |
| 2    | Client | `{ project_uuid, version_number, file_uuid }`         | Declares which file/version to operate on.       |
| 3    | Server | `llm_response` frames                                 | Streams natural‑language reasoning.              |
| 4    | Server | `tool_result` frames                                  | Instructs front‑end to mutate agent JSON.        |
| 5    | Server | `error` frames (optional)                             | Communicates problems; may close connection.     |

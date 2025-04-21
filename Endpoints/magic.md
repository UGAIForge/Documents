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
wss://ugaiforge.ai/magic/ws?type=nl_to_spl&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
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
  "type": "llm_response | tool_result | error",
  "data": "..."
}
```

| `type`         | `data` shape & front‑end handling                                                                                                                                                                                                                              |
|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `llm_response` | *String* – plain assistant text. Display it in the chat / console UI.                                                                                                                                                                                            |
| `tool_result`  | JSON object, e.g. `{ "tool": "create_item", "result": … }` or `{ "tool": "update_file_content_partial", "result": … }`.  If `tool` = `create_item`, update the agent file exactly as in the [Create Item](https://github.com/UGAIForge/Documents/blob/main/Endpoints/edit.md#create-item).  If `tool` = `update_file_content_partial`, apply each update per the [Update File Content (Partial)](https://github.com/UGAIForge/Documents/blob/main/Endpoints/file.md#update-file-content-partial). |
| `error`        | *String* – error message. Show to the user and stop any progress indicators. |

---

### Errors

- `1008 Policy Violation`: Missing or invalid `token` query parameter.  
- `1000 Normal Closure`: Connection closed after the server sends an `"error"` frame because the `type` query value is not one of `nl_to_spl | gen_alt_exc | data`.  
- `1011 Internal Error`: Unhandled server exception while processing the WebSocket.  
- `400 Bad Request` (frame payload in `"error"`): General validation failure inside a `tool_result` (e.g., bad path, out‑of‑bounds index, malformed JSON).  
- `403 Forbidden` (frame payload in `"error"`): The file does not belong to the specified project or the user lacks permission.  
- `404 Not Found` (frame payload in `"error"`): Project, version, or file cannot be located.  
- `500 Internal Server Error` (frame payload in `"error"`): Database or other internal failure encountered by the server.
---

### Summary of WebSocket Interaction

| Step | Actor  | Payload / Event                                      | Purpose                                          |
|----: |--------|------------------------------------------------------|--------------------------------------------------|
| 1    | Client | **Connect** `wss://.../magic/ws?type=…&token=…`        | Opens socket; server validates parameters.       |
| 2    | Client | `{ project_uuid, version_number, file_uuid }`         | Declares which file/version to operate on.       |
| 3    | Server | `llm_response` frames                                 | Streams natural‑language reasoning.              |
| 4    | Server | `tool_result` frames                                  | Instructs front‑end to mutate agent JSON.        |
| 5    | Server | `error` frames (optional)                             | Communicates problems; may close connection.     |

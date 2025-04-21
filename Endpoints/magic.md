# Magic WebSocket API Documentation

---

## Table of Contents
1. [Magic WebSocket (`/magic/ws`)](#magic-websocket-endpoint)

---

## Magic WebSocket Endpoint
*(Endpoint path `/magic/ws` — upgrades to WebSocket)*  

> The sections below (“Connect to `/magic/ws`”, “Client → Server Message”, etc.) describe how to use this endpoint in detail.

---

### Connect to `/magic/ws`

| Field / Attribute | Value / Notes                                                                                                        |
|-------------------|----------------------------------------------------------------------------------------------------------------------|
| **Endpoint**      | `GET ws://<host>/magic/ws`  → WebSocket upgrade                                                                      |
| **Authentication**| **Query Parameter** → `token=<access_token>` — same JWT/Bearer token used for REST endpoints.                         |
| **Required Query**| `type`  — `nl_to_spl` \| `gen_alt_exc` \| `data`                                                                     |
| **Description**   | Opens a persistent WebSocket channel. After validating the token, the server streams incremental updates while LLM‑driven “magic” tools manipulate your agent file. |

**Connection example**

```text
ws://localhost:5000/magic/ws?type=nl_to_spl&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

### Client → Server Message

Immediately after `accept`, send a single JSON object describing the target project/file.

```json
{
  "project_uuid": "project_bXBOgtA0QeucarLjCE8QCg",
  "version_number": 1,
  "file_uuid": "file_HuvSxlGCS0-hZlxMwOXwvg"
}
```

| Field            | Type    | Required | Description                                      |
|------------------|---------|----------|--------------------------------------------------|
| `project_uuid`   | string  | **Yes**  | Project owning the file.                         |
| `version_number` | integer | **Yes**  | Project version to operate on.                   |
| `file_uuid`      | string  | **Yes**  | UUID of the agent / SPL JSON file to be updated. |

*(Additional fields may be needed for future `type` values.)*

---

### Server → Client Events

Every frame sent by the server is a JSON envelope:

```json
{
  "type": "llm_response" | "tool_result" | "error",
  "data": { ... }
}
```

| `type` value      | How the front‑end should react                                                                                                                    |
|-------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| `llm_response`    | `data` is a **string** from the LLM. Display it in the assistant chat/log UI.                                                                     |
| `tool_result`     | `data.tool` indicates which tool ran:<br/>  • `create_item` – update the agent JSON exactly as in the **[Create Item]** spec.<br/>  • `update_file_content_partial` – apply each `updates` entry per **[Update File Content (Partial)]** rules. |
| `error`           | `data` is a string describing the error. Show to the user and stop any progress indicators.                                                       |

#### `tool_result` payload shape

```json
{
  "type": "tool_result",
  "data": {
    "tool": "create_item" | "update_file_content_partial",
    "result": { ... }   // schema depends on tool
  }
}
```

*Reference specs*  
- **Create Item:** <https://github.com/UGAIForge/Documents/blob/main/Endpoints/edit.md#create-item>  
- **Update File Content (Partial):** <https://github.com/UGAIForge/Documents/blob/main/Endpoints/file.md#update-file-content-partial>

---

### Errors & Close Codes

| Situation                               | Behaviour / Close Code |
|-----------------------------------------|-------------------------|
| Missing / invalid `token`               | Connection closed with **1008 (POLICY_VIOLATION)**. |
| `type` query not in allowed list        | Server sends `{ "type": "error", "data": "Invalid type…" }` then closes (1000). |
| Unhandled internal exception            | Server sends `"error"` event (if possible) then closes with **1011 (INTERNAL_ERROR)**. |
| Client initiates close                  | Server logs and ends processing gracefully. |

---

### Summary of WebSocket Interaction

| Step | Actor  | Payload / Event                                     | Purpose                                             |
|-----:|--------|------------------------------------------------------|-----------------------------------------------------|
| 1    | Client | **Connect** `ws://.../magic/ws?type=…&token=…`       | Opens channel; server validates token and type.     |
| 2    | Client | `{ project_uuid, version_number, file_uuid }`        | Specifies which file/version to operate on.         |
| 3    | Server | `llm_response` events                                | Streams natural‑language reasoning from the model.  |
| 4    | Server | `tool_result` events                                 | Instructs front‑end to mutate agent JSON accordingly.|
| 5    | Server | `error` events (optional)                            | Communicates problems; may close connection.        |

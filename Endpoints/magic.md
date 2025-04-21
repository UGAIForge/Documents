## Magic WebSocket Endpoint

---

### Table of Contents
1. [Connect to `/magic/ws`](#connect-to-magicws)
2. [Client → Server Message Format](#client--server-message-format)
3. [Server → Client Events](#server--client-events)
4. [Errors & Close Codes](#errors--close-codes)

---

## Connect to `/magic/ws`

| Attribute        | Value / Notes                                                                                                     |
|------------------|-------------------------------------------------------------------------------------------------------------------|
| **URL**          | `ws://<host>/magic/ws?type={operation}&token={access_token}`                                                      |
| **Authentication** | **Query Parameter** → `token` — a valid **access‑token** (same one used for Bearer‑auth REST calls).            |
| **Required Query** | `type` &nbsp;— one of:<br/>  • `nl_to_spl` – natural‑language → SPL assistant<br/>  • `gen_alt_exc` – *reserved* (TBD)<br/>  • `data` – *reserved* (TBD) |
| **Description**  | Opens a persistent WebSocket channel. After validation, the server streams incremental updates back to the client while it executes tool‑driven LLM actions. |

**Connection example**

```text
ws://localhost:5000/magic/ws?type=nl_to_spl&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## Client → Server Message Format

Once connected, send a **single JSON object** describing the target file and version to process.

```json
{
  "project_uuid": "project_bXBOgtA0QeucarLjCE8QCg",
  "version_number": 1,
  "file_uuid": "file_HuvSxlGCS0-hZlxMwOXwvg"
}
```

| Field            | Type            | Required | Description                                           |
|------------------|-----------------|----------|-------------------------------------------------------|
| `project_uuid`   | string (UUID)   | **Yes**  | Project that owns the file.                           |
| `version_number` | integer         | **Yes**  | Version of the project to operate on.                 |
| `file_uuid`      | string (UUID)   | **Yes**  | The agent / SPL JSON file that will be modified.      |

*For future operation types (`gen_alt_exc`, `data`) additional or different parameters may be required.*

---

## Server → Client Events

The server streams **text frames** containing a JSON envelope:

```json
{
  "type": "llm_response" | "tool_result" | "error",
  "data": { ... }
}
```

| `type`        | Description & Front‑end Handling                                                                                                                           |
|---------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `llm_response`| `data` is a **string** (plain text from the LLM). Display it in the chat/console as assistant output.                                                                                            |
| `tool_result` | `data` describes the outcome of a tool call executed by the model. See handling rules below.                                                                                                      |
| `error`       | `data` is an error message string. Surface to the user and stop any spinner/progress indicators.                                                                                                  |

### Tool Result Handling

```json
{
  "type": "tool_result",
  "data": {
    "tool": "create_item" | "update_file_content_partial",
    "result": { ... }
  }
}
```

| `tool` value                        | What to do                                                                                                                                                                                                                                    |
|------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `create_item`                      | The `result` object matches the body returned by **Create Item** (`path`, `value`). Apply the modification to the in‑memory agent file exactly as described in the [Create Item](https://github.com/UGAIForge/Documents/blob/main/Endpoints/edit.md#create-item) spec. |
| `update_file_content_partial`      | The `result` object contains an `updates` array (list of path/value pairs). Apply each update to the agent file as documented in [Update File Content (Partial)](https://github.com/UGAIForge/Documents/blob/main/Endpoints/file.md#update-file-content-partial). |

---

## Errors & Close Codes

| Condition                                   | Action / Close code |
|---------------------------------------------|---------------------|
| Invalid or missing `token`                  | Connection is closed immediately with **1008 (POLICY_VIOLATION)**. |
| Unsupported `type` parameter                | Sends an `"error"` event, then closes with normal code 1000.        |
| Any server‑side unhandled exception         | Sends an `"error"` event (if possible) then closes with **1011 (INTERNAL_ERROR)**. |
| Client disconnects                          | The server logs the event and stops processing.                      |

---

Once the connection is open and the initial payload is sent, the front end should:

1. Render incoming `llm_response` text as assistant messages.
2. Parse `tool_result` messages and **mutate the agent JSON in memory** according to the referenced specifications (`create_item`, `update_file_content_partial`).
3. Handle `error` messages gracefully by alerting the user and halting any ongoing progress UI.

# System‑Settings Endpoints Documentation

---

## Table of Contents
1. [Get System Settings](#get-system-settings)
2. [Update System Settings](#update-system-settings)
3. [Summary of System‑Settings Endpoints](#summary-of-system-settings-endpoints)

---

## Get System Settings
- **Endpoint**: `GET /system`
- **Method**: `GET`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Retrieves the current user’s system settings.  
  If no settings exist yet, the backend **creates a default configuration** (see the default JSON below) and returns it.

### Response Body (JSON)
```json
{
  "id": 1,
  "user_uuid": "user_uUCcwC7yThyXQf8eVr3Ypg",
  "model": {
    "providers": [
      {
        "name": "openai",
        "api_key": "",
        "base_url": "https://api.openai.com/v1",
        "default_model": "gpt-4o",
        "models": [
          { "name": "gpt-4o",   "active": true },
          { "name": "o1",       "active": true },
          { "name": "o3-mini",  "active": true }
        ]
      },
      {
        "name": "anthropic",
        "api_key": "",
        "base_url": "https://api.anthropic.com",
        "default_model": "claude-3-5-sonnet-20241022",
        "models": [
          { "name": "claude-3-7-sonnet-20250219", "active": true },
          { "name": "claude-3-5-sonnet-20241022", "active": true },
          { "name": "claude-3-5-haiku-20240307",  "active": true }
        ]
      }
    ]
  },
  "rules": null,
  "created_at": "2025-04-03T05:27:01.000Z",
  "updated_at": "2025-04-03T05:27:01.000Z"
}
```
| Field          | Type  | Description                                                               |
|----------------|-------|---------------------------------------------------------------------------|
| `model`        | object| Provider & model configuration (see **Model JSON Structure** below).      |
| `rules`        | string&#124;null | Optional free‑text rules that apply globally to the user’s agents. |
| `created_at` / `updated_at` | string (ISO‑8601) | Timestamps.                                      |

### Errors
- `401 Unauthorized` – Missing / invalid bearer token.  
- `400 Bad Request` – Other internal errors (rare).  
- `500 Internal Server Error` – Database or unexpected failure.

---

## Update System Settings
- **Endpoint**: `PATCH /system`
- **Method**: `PATCH`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Partially or fully updates the current user’s system settings.  
  The request **must** include a valid `model` object (providers + models) and may include `rules`.

### Request Body (JSON)
```json
{
  "model": {
    "providers": [
      {
        "name": "openai",
        "api_key": "sk‑***",
        "base_url": "https://api.openai.com/v1",
        "default_model": "gpt-4o",
        "models": [
          { "name": "gpt-4o",  "active": true },
          { "name": "o1",      "active": false },
          { "name": "o3-mini", "active": true }
        ]
      }
    ]
  },
  "rules": "• Never reveal the user’s API keys.\n• Always obey system constraints."
}
```

| Field   | Type   | Required | Validation Notes                                                                                              |
|---------|--------|----------|----------------------------------------------------------------------------------------------------------------|
| `model` | object | **Yes**  | Must contain **≥ 1** provider. Each provider needs a unique `name`, a `default_model` that exists in `models`, and ≥ 1 model entry. |
| `rules` | string | No       | If supplied, must be non‑empty (whitespace‑only strings are rejected).                                         |

### Response Body (JSON)
Same schema as **Get System Settings** (reflecting updated values).

### Errors
| Status | Reason                                                                                   |
|-------:|------------------------------------------------------------------------------------------|
| `422 Unprocessable Entity` | Validation error (e.g., empty provider name, duplicate provider names, `default_model` not in list, empty `rules`). |
| `404 Not Found`           | Settings record somehow missing for the user.                         |
| `401 Unauthorized`        | Missing / invalid bearer token.                                       |
| `400 Bad Request`         | Other internal error.                                                |
| `500 Internal Server Error` | Database or unexpected failure.                                     |

---

### Model JSON Structure

`model.providers` is an **array** where each entry follows:

| Key            | Type                | Required | Notes                                                             |
|----------------|---------------------|----------|-------------------------------------------------------------------|
| `name`         | string              | **Yes**  | Unique provider identifier (e.g., `"openai"`).                    |
| `api_key`      | string&#124;null    | No       | Stored encrypted server‑side; empty string allowed.               |
| `base_url`     | string&#124;null    | No       | Must start with `http://` or `https://` if provided.              |
| `default_model`| string              | **Yes**  | Must match one of the `models[].name` values.                     |
| `models`       | array&lt;object&gt; | **Yes**  | Each object has `name` (string) and `active` (boolean).           |

Validation rules enforced server‑side:
- Provider names must be unique.
- Each provider must list at least one model.
- `default_model` must exist in its `models` array.
- Empty or whitespace‑only strings are rejected where values are required.

---

## Summary of System‑Settings Endpoints

| Method | Endpoint | Description                                |
|-------:|----------|--------------------------------------------|
| **GET**    | `/system` | **Get System Settings** – retrieve (or auto‑create) the current user’s settings |
| **PATCH**  | `/system` | **Update System Settings** – modify provider/model config or global rules |

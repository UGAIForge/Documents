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
        "type": "system",
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
        "type": "system",
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
- **Description**: Partially updates the current user’s system settings.  
  The request **must** include a valid `model` object (with updated provider details) and **may** include a `rules` string.

### Request Body (JSON)
```json
{
  "model": {
    "provider_name": "openai",
    "updated_provider": {
      "name": "openai",
      "api_key": "new-key",
      "base_url": "https://api.openai.com/v1",
      "default_model": "gpt-4o",
      "models": [
        { "name": "gpt-4o", "active": true },
        { "name": "o1", "active": true },
        { "name": "o3-mini", "active": false }
      ]
    }
  },
  "rules": "• Never reveal the user’s API keys.\n• Always obey system constraints."
}
```

### Request Fields

| Field                  | Type            | Required | Description                                                                                        |
|------------------------|-----------------|----------|----------------------------------------------------------------------------------------------------|
| `model`               | object          | **Yes**  | Contains `provider_name` and `updated_provider` with the new provider config to verify.            |
| └─ `provider_name`     | string          | **Yes**  | The name of the provider (e.g., `"openai"`, `"anthropic"`). Must match the provider you want to update/verify. |
| └─ `updated_provider`  | ProviderConfig  | **Yes**  | A valid `ProviderConfig` object describing the updated provider settings (API key, models, etc.).  |
| `rules`               | string          | No       | A string containing any system-level rules. It cannot be an empty string if provided.              |

### Response Body (JSON)
Same schema as **Get System Settings**, reflecting the updated values.

### Errors
- `422 Unprocessable Entity` – Validation error (e.g., empty provider name, duplicate provider names, `default_model` not in list, empty `rules`).  
- `404 Not Found` – Settings record not found for the user.  
- `401 Unauthorized` – Missing / invalid bearer token.  
- `400 Bad Request` – Other internal errors (rare).  
- `500 Internal Server Error` – Database or unexpected failure.
---

### Model JSON Structure

`model.providers` is an **array** where each entry follows:

| Key             | Type                | Required | Notes                                                                                   |
|-----------------|---------------------|----------|-----------------------------------------------------------------------------------------|
| `name`          | string              | **Yes**  | Unique provider identifier (e.g., `"openai"`).                                          |
| `type`          | string              | **Yes**  | Must be either `"system"` or `"user"`.                                                  |
| `api_key`       | string&#124;null    | No       | Stored encrypted server‑side; empty string allowed.                                     |
| `base_url`      | string&#124;null    | No       | Must start with `http://` or `https://` if provided.                                    |
| `default_model` | string              | **Yes**  | Must match one of the `models[].name` values.                                           |
| `models`        | array&lt;object&gt; | **Yes**  | Each object must include `name` (string) and `active` (boolean).                        |

Validation rules enforced server‑side:
- Provider names must be unique.
- Each provider must list at least one model.
- `default_model` must exist in its `models` array.
- Empty or whitespace‑only strings are rejected where values are required.
- For providers with `type: "system"`:
  - `name` and `base_url` **cannot be changed**.
- For providers with `type: "user"`:
  - All fields are editable.

---

### Verify Provider Settings
- **Endpoint**: `POST /system/verify`
- **Method**: `POST`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Verifies the API key. If verification fails (e.g., invalid API key or unsupported provider), the request will return an error.


#### Request Body (JSON)

Expects the same structure as **SystemSettingUpdate**, with an updated provider configuration:

```json
{
  "model": {
    "provider_name": "openai", 
    "updated_provider": {
      "name": "openai",
      "type": "some-provider-type",
      "api_key": "sk-xxxxxx",
      "base_url": "https://api.openai.com/v1",
      "default_model": "gpt-4o",
      "models": [
        {
          "name": "gpt-4o",
          "active": true
        }
      ]
    }
  },
  "rules": "Optional rules"
}
```

| Field                  | Type            | Required | Description                                                                                        |
|------------------------|-----------------|----------|----------------------------------------------------------------------------------------------------|
| `model`               | object          | **Yes**  | Contains `provider_name` and `updated_provider` with the new provider config to verify.            |
| └─ `provider_name`     | string          | **Yes**  | The name of the provider (e.g., `"openai"`, `"anthropic"`). Must match the provider you want to update/verify. |
| └─ `updated_provider`  | ProviderConfig  | **Yes**  | A valid `ProviderConfig` object describing the updated provider settings (API key, models, etc.).  |
| `rules`               | string          | No       | A string containing any system-level rules. It cannot be an empty string if provided.              |

**Example**:
```json
{
  "model": {
    "provider_name": "openai",
    "updated_provider": {
      "name": "openai",
      "type": "text-generation",
      "api_key": "sk-xxxxxx",
      "base_url": "https://api.openai.com/v1",
      "default_model": "gpt-4o",
      "models": [
        {
          "name": "gpt-4o",
          "active": true
        }
      ]
    }
  },
  "rules": "Some optional rules"
}
```

#### Response
If the provider settings are successfully updated and verified, the endpoint returns:

```json
{
  "status": "success",
  "message": "API key verified successfully"
}
```

#### Errors
- `422 Unprocessable Entity`:
  - Invalid provider configuration, e.g., empty `name` or missing `models`.
  - Missing `model` block in the request.
  - `rules` provided but is an empty string.
  - Unsupported `provider_name`.
- `404 Not Found`:  
  - The user’s system settings do not exist (though normally a default is created automatically).
- `401 Unauthorized`: Missing or invalid bearer token.
- `400 Bad Request`: Other internal errors, e.g., failure to create or test the provider service instance.
- `500 Internal Server Error`: Unexpected database or server error.

## Summary of System‑Settings Endpoints

| Method | Endpoint | Description                                |
|-------:|----------|--------------------------------------------|
| **GET**    | `/system` | **Get System Settings** – retrieve (or auto‑create) the current user’s settings |
| **PATCH**  | `/system` | **Update System Settings** – modify provider/model config or global rules |

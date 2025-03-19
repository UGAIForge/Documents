## User Endpoints Documentation

---

### Table of Contents
1. [Sign Up](#sign-up)
2. [Get User](#get-user)
3. [Update User](#update-user)
4. [Change Password](#change-password)
5. [Delete User](#delete-user)
6. [Summary of User Endpoints](#summary-of-user-endpoints)

---

### Sign Up
- **Endpoint**: `POST /users`
- **Method**: `POST`
- **Authentication**: None
- **Description**: Creates a new user account.
- **Request Body** (JSON):
  ```json
  {
    "name": "John Doe",
    "email": "john.doe@example.com",
    "password": "string",
    "invitation_code": "string"
  }
  ```
  | Field            | Type   | Required | Description                                     |
  |------------------|--------|----------|-------------------------------------------------|
  | `name`           | string | Optional | User's display name.                            |
  | `email`          | string | Required | User's email address. Must be unique.           |
  | `password`       | string | Required | Plaintext password (will be hashed).            |
  | `invitation_code`| string | Required | Invitation code required for new sign-ups.      |

- **Response**: `201 Created`  
  Returns a **UserResponse** object:
  ```json
  {
    "uuid": "string",
    "name": "John Doe",
    "email": "john.doe@example.com",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-05T12:34:56.789Z",
    "last_login": null
  }
  ```
- **Errors**:
  - `403 Forbidden`: Invalid invitation code.
  - `400 Bad Request`: Email already exists.
  - `500 Internal Server Error`: Database or other internal error.

---

### Get User
- **Endpoint**: `GET /users/{user_uuid}`
- **Method**: `GET`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Fetch a specific user by their UUID.
- **URL Params**:
  - `user_uuid` (string): The UUID of the user to retrieve.
- **Response**: `200 OK`  
  Returns a **UserResponse** object:
  ```json
  {
    "uuid": "string",
    "name": "John Doe",
    "email": "john.doe@example.com",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-05T12:34:56.789Z",
    "last_login": "2023-10-10T10:20:30.456Z"
  }
  ```
- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `404 Not Found`: User not found.
  - `400 Bad Request`: Other errors (e.g., malformed UUID).
  - `500 Internal Server Error`: Database or other internal error.

---

### Update User
- **Endpoint**: `PATCH /users/{user_uuid}`
- **Method**: `PATCH`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Update an existing user’s information (currently only the `name` field is supported).
- **URL Params**:
  - `user_uuid` (string): The UUID of the user to update.
- **Request Body** (JSON):
  ```json
  {
    "name": "New Name",
    "email": "john.doe@example.com"
  }
  ```
  > Typically, you only pass the fields you want to update; unsupported or unset fields will be ignored.

- **Response**: `200 OK`  
  Returns the updated **UserResponse** object:
  ```json
  {
    "uuid": "string",
    "name": "New Name",
    "email": "john.doe@example.com",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-12T11:22:33.789Z",
    "last_login": "2023-10-10T10:20:30.456Z"
  }
  ```
- **Errors**:
  - `403 Forbidden`: Attempting to update someone else’s profile.
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `404 Not Found`: User not found.
  - `500 Internal Server Error`: Database or other internal error.

---

### Change Password
- **Endpoint**: `PATCH /users/{user_uuid}/password`
- **Method**: `PATCH`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Allows the current user to change their own password, given the old password is correct.
- **URL Params**:
  - `user_uuid` (string): The UUID of the user whose password is being changed.
- **Request Body** (JSON):
  ```json
  {
    "old_password": "string",
    "new_password": "string"
  }
  ```
  | Field          | Type   | Required | Description                        |
  |----------------|--------|----------|------------------------------------|
  | `old_password` | string | Required | The current password for the user. |
  | `new_password` | string | Required | The new desired password.          |

- **Response**: `200 OK`  
  Returns the updated **UserResponse** object:
  ```json
  {
    "uuid": "string",
    "name": "John Doe",
    "email": "john.doe@example.com",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-12T11:22:33.789Z",
    "last_login": "2023-10-10T10:20:30.456Z"
  }
  ```
- **Errors**:
  - `403 Forbidden`: Attempting to change someone else’s password.
  - `401 Unauthorized`: Missing or invalid bearer token, or old password is incorrect.
  - `404 Not Found`: User not found.
  - `500 Internal Server Error`: Database or other internal error.

---

### Delete User
- **Endpoint**: `DELETE /users/{user_uuid}`
- **Method**: `DELETE`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Delete a user’s account after validating the current password.
- **URL Params**:
  - `user_uuid` (string): The UUID of the user to delete.
- **Request Body** (JSON):
  ```json
  {
    "password": "string"
  }
  ```
  | Field      | Type   | Required | Description            |
  |------------|--------|----------|------------------------|
  | `password` | string | Required | The current password.  |

- **Response**: `204 No Content`  
  Returns no data if the deletion is successful.

- **Errors**:
  - `403 Forbidden`: Attempting to delete someone else’s account.
  - `401 Unauthorized`: Missing or invalid bearer token, or incorrect password.
  - `404 Not Found`: User not found.
  - `500 Internal Server Error`: Database or other internal error.

---

### Summary of User Endpoints

| Method | Endpoint                        | Description                                      |
|-------:|---------------------------------|--------------------------------------------------|
| **POST**   | `/users`                          | **Sign Up** – Create a new user                   |
| **GET**    | `/users/{user_uuid}`              | **Get User** – Retrieve details of a specific user|
| **PATCH**  | `/users/{user_uuid}`              | **Update User** – Update user’s info (name)       |
| **PATCH**  | `/users/{user_uuid}/password`     | **Change Password** – Change user’s password      |
| **DELETE** | `/users/{user_uuid}`              | **Delete User** – Remove user’s account           |

---

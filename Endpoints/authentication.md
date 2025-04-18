## Authentication Endpoints Documentation

---

### Table of Contents
1. [Login](#login)
2. [Refresh Token](#refresh-token)
3. [Logout](#logout)
4. [Summary of Authentication Endpoints](#summary-of-authentication-endpoints)

---

### Login
- **Endpoint**: `POST /auth/login`
- **Method**: `POST`
- **Authentication**: None (credentials provided in the request body)
- **Description**: Logs a user in with email and password, returns an access token and sets a refresh token as an HTTP-only cookie.
- **Request Body** (Form Data – via `OAuth2PasswordRequestForm`):
  ```plaintext
  username: john.doe@example.com
  password: mysecretpassword
  ```
- **Response**: `200 OK`  
  Returns a **UserWithToken** object, which includes user details and an access token:
  ```json
  {
    "user": {
      "uuid": "string",
      "name": "John Doe",
      "email": "john.doe@example.com",
      "created_at": "2023-10-05T12:34:56.789Z",
      "updated_at": "2023-10-05T12:34:56.789Z",
      "last_login": "2023-10-10T10:20:30.456Z"
    },
    "token": {
      "access_token": "string",
      "token_type": "bearer"
    }
  }
  ```
  > Additionally, a `refresh_token` cookie is set (HTTP-only, `samesite=None`, `secure`) for session management.

- **Errors**:
  - `401 Unauthorized`: Incorrect username or password.
  - `500 Internal Server Error`: Database or other internal error.

---

### Refresh Token
- **Endpoint**: `POST /auth/refreshtokens`
- **Method**: `POST`
- **Authentication**: **Refresh Token** (provided via HTTP-only cookie)
- **Description**: Obtain a new access token using the `refresh_token` cookie.
- **Cookies**:
  - `refresh_token`: The refresh token set by the login endpoint.
- **Response**: `200 OK`  
  Returns a **Token** object:
  ```json
  {
    "access_token": "string",
    "token_type": "bearer",
    "refresh_token": "string"
  }
  ```
  > The `refresh_token` in the response body will be the same if it was valid.

- **Errors**:
  - `401 Unauthorized`: Invalid or missing `refresh_token` cookie; results in redirection to `/auth/login`.
  - `500 Internal Server Error`: Database or other internal error.

---

### Logout
- **Endpoint**: `DELETE /auth/logout`
- **Method**: `DELETE`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Logs the user out by clearing the `refresh_token` cookie.
- **Response**: `204 No Content`  
  The response will delete the cookie if present.

- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.

---

### Summary of Authentication Endpoints

| Method | Endpoint             | Description                              |
|-------:|----------------------|------------------------------------------|
| **POST**   | `/auth/login`         | **Login** – Authenticate & get access token  |
| **POST**   | `/auth/refreshtokens` | **Refresh Token** – Obtain a new access token |
| **DELETE** | `/auth/logout`        | **Logout** – Invalidate refresh token cookie  |

---

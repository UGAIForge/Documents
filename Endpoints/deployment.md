# Deployments API Documentation

---

## Table of Contents
1. [Create Deployment](#create-deployment)
2. [Get Paginated Deployments](#get-paginated-deployments)
3. [Get Deployment with Versions](#get-deployment-with-versions)
4. [Update Deployment](#update-deployment)
5. [Delete Deployment](#delete-deployment)
6. [Summary of Deployment Endpoints](#summary-of-deployment-endpoints)

---

## Create Deployment
- **Endpoint**: `POST /deployments`
- **Method**: `POST`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Creates a new deployment for a specific project version, and also creates an initial deployment version (version 1).
- **Query Parameters**:  
  - `project_uuid` (string, required): The UUID of the project.  
  - `version_number` (integer, required): The version number of the project.
  
- **Request Body** (JSON):
  ```json
  {
    "name": "MyDeployment",
    "visibility": "private"
  }
  ```
  | Field   | Type   | Required | Description                                                |
  |---------|--------|----------|------------------------------------------------------------|
  | `name`  | string | Required  | A short, descriptive name for the deployment.             |
  | `visibility`  | "private"\|"public" | Required | Defaults to `private` if not provided.                       |


- **Response**: `201 Created`  
  Returns a **DeploymentWithVersionsResponse** object, including the initial version:
  ```json
  {
    "uuid": "abcdef1234567890abcdef1234567890",
    "name": "MyDeployment",
    "user_uuid": "userabcdef1234567890abcdef1234",
    "visibility": "private",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-05T12:34:56.789Z",
    "versions": [
      {
        "version_number": 1,
        "tag": "Initial version",
        "url": null,
        "settings": null,
        "created_at": "2023-10-05T12:34:56.789Z",
        "updated_at": "2023-10-05T12:34:56.789Z"
      }
    ]
  }
  ```
- **Errors**:
  - `404 Not Found`:  
    - Current user not found.  
    - Specified project version not found for the given `project_uuid` and `version_number`.
  - `400 Bad Request`: General error or invalid request data.
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `500 Internal Server Error`: Database or other internal error.

---

## Get Paginated Deployments
- **Endpoint**: `GET /deployments`
- **Method**: `GET`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Returns a paginated list of all deployments owned by the current user.
- **Query Parameters**:  
  - `page` (integer, default `1`, min `1`): Page number.
  - `page_size` (integer, default `10`, min `1`, max `100`): Items per page.

- **Response**: `200 OK`  
  Returns a **DeploymentPaginationResponse** object:
  ```json
  {
    "items": [
      {
        "uuid": "abcdef1234567890abcdef1234567890",
        "name": "MyDeployment",
        "user_uuid": "userabcdef1234567890abcdef1234",
        "visibility": "private",
        "created_at": "2023-10-05T12:34:56.789Z",
        "updated_at": "2023-10-05T12:34:56.789Z"
      },
      ...
    ],
    "total": 1,
    "page": 1,
    "page_size": 10,
    "total_pages": 1
  }
  ```
  Where `items` is a list of **DeploymentResponse** objects.

- **Errors**:
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `400 Bad Request`: Invalid query parameters or other general error.
  - `500 Internal Server Error`: Database or other internal error.

---

## Get Deployment with Versions
- **Endpoint**: `GET /deployments/{deployment_uuid}`
- **Method**: `GET`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Fetches a single deployment (owned by the current user) by its UUID, including all associated deployment versions.
- **URL Params**:
  - `deployment_uuid` (string, required): The UUID of the deployment to retrieve.

- **Response**: `200 OK`  
  Returns a **DeploymentWithVersionsResponse** object:
  ```json
  {
    "uuid": "abcdef1234567890abcdef1234567890",
    "name": "MyDeployment",
    "user_uuid": "userabcdef1234567890abcdef1234",
    "visibility": "private",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-05T12:34:56.789Z",
    "versions": [
      {
        "version_number": 1,
        "tag": "Initial version",
        "url": null,
        "settings": null,
        "created_at": "2023-10-05T12:34:56.789Z",
        "updated_at": "2023-10-05T12:34:56.789Z"
      },
      ...
    ]
  }
  ```
- **Errors**:
  - `403 Forbidden`: Trying to access a deployment that you do not own.
  - `404 Not Found`: Deployment does not exist or no access.
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `400 Bad Request`: Other errors (e.g., malformed UUID).
  - `500 Internal Server Error`: Database or other internal error.

---

## Update Deployment
- **Endpoint**: `PATCH /deployments/{deployment_uuid}`
- **Method**: `PATCH`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Partially updates the metadata of a deployment (e.g., its `name`). Only the deployment owner can perform this action.
- **URL Params**:
  - `deployment_uuid` (string, required): The UUID of the deployment to update.
- **Request Body** (JSON):
  ```json
  {
    "name": "NewDeploymentName"
  }
  ```
  | Field   | Type   | Required | Description            |
  |---------|--------|----------|------------------------|
  | `name`  | string | No       | New deployment name.   |

- **Response**: `200 OK`  
  Returns a **DeploymentResponse** object:
  ```json
  {
    "uuid": "abcdef1234567890abcdef1234567890",
    "name": "NewDeploymentName",
    "visibility": "private",
    "updated_at": "2023-10-06T08:22:33.123Z"
  }
  ```
- **Errors**:
  - `403 Forbidden`: Only the deployment owner can update it.
  - `404 Not Found`: Deployment not found.
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `400 Bad Request`: Invalid request data or other errors.
  - `500 Internal Server Error`: Database or other internal error.

---

## Delete Deployment
- **Endpoint**: `DELETE /deployments/{deployment_uuid}`
- **Method**: `DELETE`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Deletes a deployment (and all its versions). Only the deployment owner can perform this action.
- **URL Params**:
  - `deployment_uuid` (string, required): The UUID of the deployment to delete.

- **Response**: `204 No Content`  
  Returns no data if deletion is successful.

- **Errors**:
  - `403 Forbidden`: Only the deployment owner can delete it.
  - `404 Not Found`: Deployment not found.
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `400 Bad Request`: Other errors (e.g., malformed UUID).
  - `500 Internal Server Error`: Database or other internal error.

---

## Summary of Deployment Endpoints

| Method | Endpoint                              | Description                                                     |
|-------:|---------------------------------------|-----------------------------------------------------------------|
| **POST**   | `/deployments?project_uuid={}&version_number={}` | **Create Deployment** – with initial version                    |
| **GET**    | `/deployments`                            | **Get Paginated Deployments** – deployments for the current user |
| **GET**    | `/deployments/{deployment_uuid}`           | **Get Deployment with Versions** – single deployment + versions |
| **PATCH**  | `/deployments/{deployment_uuid}`           | **Update Deployment** – only owner                              |
| **DELETE** | `/deployments/{deployment_uuid}`           | **Delete Deployment** – only owner                              |

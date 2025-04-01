# Deployment Versions API Documentation

---

## Table of Contents
1. [Create Deployment Version](#create-deployment-version)
2. [Get Deployment Version](#get-deployment-version)
3. [Update Deployment Version](#update-deployment-version)
4. [Delete Deployment Version](#delete-deployment-version)
5. [Summary of Deployment Version Endpoints](#summary-of-deployment-version-endpoints)

---

## Create Deployment Version
- **Endpoint**: `POST /deployments/{deployment_uuid}/versions`
- **Method**: `POST`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Creates a new version of an existing deployment. Only the deployment owner can create new versions.
- **URL Params**:
  - `deployment_uuid` (string, required): The UUID of the deployment for which a new version will be created.
- **Request Body** (JSON):
  ```json
  {
    "tag": "My Tag",
    "settings": {
      "env": "production",
      "replicas": 3
    }
  }
  ```
  | Field         | Type                 | Required | Description                                                 |
  |---------------|----------------------|----------|-------------------------------------------------------------|
  | `tag`         | string (optional)   | No       | A descriptive tag for the new version.                      |
  | `settings`    | object (JSON)       | No       | Arbitrary JSON settings, e.g., environment config.          |

- **Response**: `201 Created`  
  Returns a **DeploymentVersionResponse** object:
  ```json
  {
    "version_number": 2,
    "tag": "My Tag",
    "url": "https://example.com/deploy",
    "settings": {
      "env": "production",
      "replicas": 3
    },
    "status": "deploying",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-05T12:34:56.789Z"
  }
  ```
  > Note that the `version_number` in the response is automatically assigned as one more than the latest version number for that deployment.

- **Errors**:
  - `403 Forbidden`: The user does not own the deployment or does not have permission to create versions.
  - `404 Not Found`: Deployment not found; or current user not found.
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `400 Bad Request`: General or validation error.
  - `500 Internal Server Error`: Database or other internal error.

---

## Get Deployment Version
- **Endpoint**: `GET /deployments/{deployment_uuid}/versions/{version_number}`
- **Method**: `GET`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Retrieves a specific version of a deployment, accessible only by its owner.
- **URL Params**:
  - `deployment_uuid` (string, required): The UUID of the deployment to query.
  - `version_number` (integer, required): The version number to retrieve.
- **Response**: `200 OK`  
  Returns a **DeploymentVersionResponse** object:
  ```json
  {
    "version_number": 2,
    "tag": "My Tag",
    "url": "https://example.com/deploy",
    "settings": {
      "env": "production",
      "replicas": 3
    },
    "status": "deploying",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-06T07:21:33.789Z"
  }
  ```
- **Errors**:
  - `403 Forbidden`: Attempting to view a deployment version you do not own.
  - `404 Not Found`: Deployment or specified version not found.
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `400 Bad Request`: Malformed parameters or other errors.
  - `500 Internal Server Error`: Database or other internal error.

---

## Update Deployment Version
- **Endpoint**: `PATCH /deployments/{deployment_uuid}/versions/{version_number}`
- **Method**: `PATCH`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Partially updates a deployment version’s metadata (e.g., `tag`, `url`, `settings`). Only the deployment owner can perform updates.
- **URL Params**:
  - `deployment_uuid` (string, required): The UUID of the deployment.
  - `version_number` (integer, required): The version number to update.
- **Request Body** (JSON):
  ```json
  {
    "tag": "New Tag",
    "settings": {
      "replicas": 5
    }
  }
  ```
  | Field         | Type                 | Required | Description                                     |
  |---------------|----------------------|----------|-------------------------------------------------|
  | `tag`         | string (optional)   | No       | Updated tag for this version.                   |
  | `settings`    | object (JSON)       | No       | Updated JSON settings.                          |

- **Response**: `200 OK`  
  Returns an updated **DeploymentVersionResponse** object:
  ```json
  {
    "version_number": 2,
    "tag": "New Tag",
    "url": "https://new.example.com",
    "settings": {
      "replicas": 5
    },
    "status": "deploying",
    "created_at": "2023-10-05T12:34:56.789Z",
    "updated_at": "2023-10-06T09:11:45.001Z"
  }
  ```
- **Errors**:
  - `403 Forbidden`: Only the owner can update the version.
  - `404 Not Found`: Deployment or specified version not found.
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `400 Bad Request`: Invalid update data or general errors.
  - `500 Internal Server Error`: Database or other internal error.

---

## Delete Deployment Version
- **Endpoint**: `DELETE /deployments/{deployment_uuid}/versions/{version_number}`
- **Method**: `DELETE`
- **Authentication**: **Bearer Token** (must include `Authorization: Bearer <access_token>`)
- **Description**: Deletes a specific version of a deployment. Only the deployment owner can delete it.
- **URL Params**:
  - `deployment_uuid` (string, required): The UUID of the deployment.
  - `version_number` (integer, required): The version number to delete.

- **Response**: `204 No Content`  
  Returns no data if the deletion is successful.

- **Errors**:
  - `403 Forbidden`: Only the deployment owner can delete the version.
  - `404 Not Found`: Deployment or version not found.
  - `401 Unauthorized`: Missing or invalid bearer token.
  - `400 Bad Request`: Other errors (e.g., malformed UUID).
  - `500 Internal Server Error`: Database or other internal error.

---

## Summary of Deployment Version Endpoints

| Method | Endpoint                                                         | Description                                            |
|-------:|------------------------------------------------------------------|--------------------------------------------------------|
| **POST**   | `/deployments/{deployment_uuid}/versions`                            | **Create Deployment Version**                          |
| **GET**    | `/deployments/{deployment_uuid}/versions/{version_number}`          | **Get Deployment Version**                             |
| **PATCH**  | `/deployments/{deployment_uuid}/versions/{version_number}`          | **Update Deployment Version**                          |
| **DELETE** | `/deployments/{deployment_uuid}/versions/{version_number}`          | **Delete Deployment Version**                          |

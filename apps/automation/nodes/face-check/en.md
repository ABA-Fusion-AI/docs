---
node_id: facecheck
title: FaceCheck
description: Search the Internet by face using the FaceCheck API, upload images for a search session, check search results, and delete uploaded pictures.
category: AI & Computer Vision
version: 1.0.0
language: TypeScript
last_updated: 2026-09-09
author: ABA Fusion AI
tags:
  - facecheck
  - face-search
  - facial-recognition
  - reverse-image-search
  - computer-vision
related_nodes: []
---

# FaceCheck

**Category:** AI & Computer Vision  
**Type:** Action Node

## Overview

The **FaceCheck** node integrates the FaceCheck API into Fusion workflows.

It supports four operations:

- **Get Info** — Retrieve FaceCheck API account/service information.
- **Search** — Submit or check a face-search request.
- **Upload Picture** — Upload one or more images to an existing search session.
- **Delete Picture** — Remove a previously uploaded picture from a search session.

The node uses the FaceCheck API at:

```text
https://facecheck.id
```

Authentication is required for all operations through the `apiKey` parameter.

---

## Operations

### 1. Get Info

Retrieves information from the FaceCheck API.

**Operation value:**

```text
getInfo
```

**Endpoint:**

```text
POST https://facecheck.id/api/info
```

**Required parameters:**

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `apiKey` | string | Yes | FaceCheck API authentication token. |

---

### 2. Search

Starts a FaceCheck search or checks the status/results of an existing search.

**Operation value:**

```text
search
```

**Endpoint:**

```text
POST https://facecheck.id/api/search
```

**Required parameters:**

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `apiKey` | string | Yes | FaceCheck API authentication token. |
| `id_search` | string | Yes | Search identifier. |

**Optional parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `demo` | boolean | — | When enabled, limits the search to the first 100,000 faces for testing. |
| `status_only` | boolean | — | When enabled, checks the current search status without submitting a new search. |
| `with_progress` | boolean | — | When enabled, returns immediately with progress information instead of waiting for completion. |

**Request body example:**

```json
{
  "id_search": "SEARCH_ID",
  "demo": true,
  "status_only": false,
  "with_progress": true
}
```

---

### 3. Upload Picture

Uploads one or more images to a FaceCheck search session.

**Operation value:**

```text
uploadPic
```

**Endpoint:**

```text
POST https://facecheck.id/api/upload_pic
```

**Required parameters:**

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `apiKey` | string | Yes | FaceCheck API authentication token. |
| `id_search` | string | Yes | Search identifier to associate the image with. |
| Image input | string | Yes | At least one of `images`, `imageUrls`, or `imageBase64`. |

**Image parameters:**

| Parameter | Type | Description |
|---|---|---|
| `images` | string | Legacy input. Accepts comma-separated image URLs or base64 values. |
| `imageUrls` | string | Comma-separated image URLs. The node downloads each image before uploading it to FaceCheck. |
| `imageBase64` | string | Base64 image data. Multiple values may be supplied as a comma-separated string. |

The node builds a `multipart/form-data` request and sends each processed image using the `images` form field.

---

### 4. Delete Picture

Deletes a previously uploaded picture from a FaceCheck search session.

**Operation value:**

```text
deletePic
```

**Endpoint:**

```text
POST https://facecheck.id/api/delete_pic
```

**Required parameters:**

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `apiKey` | string | Yes | FaceCheck API authentication token. |
| `id_search` | string | Yes | Search identifier. |
| `id_pic` | string | Yes | Picture identifier to delete. |

The node sends `id_search` and `id_pic` as query parameters.

**Request format:**

```text
POST https://facecheck.id/api/delete_pic?id_search=SEARCH_ID&id_pic=PICTURE_ID
```

---

## Parameters

### Common Parameters

| Parameter | Type | Required | Default | Used By | Description |
|---|---|---:|---|---|---|
| `operation` | enum | No | `getInfo` | All | Operation to execute: `getInfo`, `search`, `uploadPic`, or `deletePic`. |
| `apiKey` | string | Yes | — | All | FaceCheck API authentication token. |

### Search Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `id_search` | string | Yes | — | Search ID used by search, upload, and delete operations. |
| `demo` | boolean | No | — | Limits search scope for testing. |
| `status_only` | boolean | No | — | Only checks search status. |
| `with_progress` | boolean | No | — | Returns progress instead of waiting for the completed search. |

### Upload Parameters

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `images` | string | Conditional | Legacy comma-separated URL/base64 input. |
| `imageUrls` | string | Conditional | Comma-separated image URLs. |
| `imageBase64` | string | Conditional | Base64 encoded image data. |

At least one image source is required for `uploadPic`.

### Delete Parameters

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `id_pic` | string | Yes | Picture ID to remove from the search session. |

---

## Dynamic Input Resolution

The node can also resolve several parameters from incoming workflow data when they are not configured directly.

### API Key aliases

```text
apiKey
token
api_token
```

### Search ID aliases

```text
id_search
searchId
search_id
```

### Picture ID aliases

```text
id_pic
picId
pic_id
```

### Image URL aliases

```text
imageUrls
image_urls
urls
```

### Base64 aliases

```text
imageBase64
image_base64
base64
```

### Generic image aliases

```text
images
image
```

Configured node values take priority over incoming workflow values.

---

## Outputs

The node returns the JSON response received from the FaceCheck API.

The exact response structure depends on the selected operation.

Typical output categories may include:

- API/account information for `getInfo`.
- Search status, progress, or results for `search`.
- Uploaded picture/search information for `uploadPic`.
- Deletion confirmation for `deletePic`.

The node does not transform the successful API response into a custom output schema.

---

## Examples

### Example 1 — Get Info

```json
{
  "operation": "getInfo",
  "apiKey": "YOUR_API_KEY"
}
```

### Example 2 — Start or Continue a Search

```json
{
  "operation": "search",
  "apiKey": "YOUR_API_KEY",
  "id_search": "SEARCH_ID",
  "demo": true,
  "status_only": false,
  "with_progress": true
}
```

### Example 3 — Check Search Status Only

```json
{
  "operation": "search",
  "apiKey": "YOUR_API_KEY",
  "id_search": "SEARCH_ID",
  "status_only": true,
  "with_progress": true
}
```

### Example 4 — Upload Image from URL

```json
{
  "operation": "uploadPic",
  "apiKey": "YOUR_API_KEY",
  "id_search": "SEARCH_ID",
  "imageUrls": "https://example.com/person.jpg"
}
```

### Example 5 — Upload Base64 Image

```json
{
  "operation": "uploadPic",
  "apiKey": "YOUR_API_KEY",
  "id_search": "SEARCH_ID",
  "imageBase64": "BASE64_IMAGE_DATA"
}
```

### Example 6 — Delete Picture

```json
{
  "operation": "deletePic",
  "apiKey": "YOUR_API_KEY",
  "id_search": "SEARCH_ID",
  "id_pic": "PICTURE_ID"
}
```

---

## cURL Tests

> Replace placeholder values before testing.

### Get Info

```bash
curl.exe -L -X POST "https://facecheck.id/api/info" -H "Accept: application/json" -H "Authorization: YOUR_API_KEY"
```

### Search

```bash
curl.exe -L -X POST "https://facecheck.id/api/search" -H "Accept: application/json" -H "Content-Type: application/json" -H "Authorization: YOUR_API_KEY" --data "{\"id_search\":\"SEARCH_ID\",\"demo\":true,\"status_only\":false,\"with_progress\":true}"
```

### Check Search Status

```bash
curl.exe -L -X POST "https://facecheck.id/api/search" -H "Accept: application/json" -H "Content-Type: application/json" -H "Authorization: YOUR_API_KEY" --data "{\"id_search\":\"SEARCH_ID\",\"status_only\":true,\"with_progress\":true}"
```

### Upload Picture

```bash
curl.exe -L -X POST "https://facecheck.id/api/upload_pic" -H "Accept: application/json" -H "Authorization: YOUR_API_KEY" -F "id_search=SEARCH_ID" -F "images=@C:\path\to\image.jpg"
```

### Delete Picture

```bash
curl.exe -L -X POST "https://facecheck.id/api/delete_pic?id_search=SEARCH_ID&id_pic=PICTURE_ID" -H "Accept: application/json" -H "Authorization: YOUR_API_KEY"
```

---

## Validation Rules

### Missing API Key

All operations require `apiKey`.

```text
apiKey is required for FaceCheck API operations
```

### Missing Search ID

The following operations require `id_search`:

- `search`
- `uploadPic`
- `deletePic`

```text
id_search is required for search, uploadPic, and deletePic operations
```

### Missing Picture ID

`deletePic` requires `id_pic`.

```text
id_pic is required for deletePic operation
```

### Missing Upload Image

`uploadPic` requires at least one of:

```text
images
imageUrls
imageBase64
```

Otherwise the node throws:

```text
At least one of images, imageUrls, or imageBase64 is required for uploadPic operation
```

---

## Error Handling

The node wraps API and processing errors using the selected operation.

General format:

```text
FaceCheck <operation> failed: <error>
```

### Authentication Error

When the API returns HTTP `401`:

```text
FaceCheck API Authentication failed (401): Invalid or missing API key
```

### Resource Not Found

When the API returns HTTP `404`:

```text
FaceCheck API Resource not found (404): <URL>
```

### Other API Errors

For other non-success HTTP responses:

```text
FaceCheck API Error: <status> <statusText> - <response body>
```

### Image URL Error

When the node cannot download an image URL:

```text
Error processing image URL <URL>: <error>
```

### Base64 Error

When base64 data cannot be processed:

```text
Error processing base64 image: <error>
```

---

## Troubleshooting

### 401 Authentication Failed

**Cause:**  
The API key is missing or rejected by FaceCheck.

**Check:**

- `apiKey` is configured.
- The token is still valid.
- The value is sent in the format expected by the FaceCheck API.

The implementation sends the configured `apiKey` value directly in the `Authorization` header.

### Search Fails Because `id_search` Is Missing

Provide `id_search` directly or pass one of:

```text
id_search
searchId
search_id
```

### Upload Fails Before Reaching FaceCheck

Check that:

- The image URL is publicly accessible from the Fusion runtime.
- The URL returns a successful HTTP response.
- The remote server does not block automated downloads.

### Upload Reports That No Image Was Provided

Provide at least one of:

```text
images
imageUrls
imageBase64
```

### Delete Picture Fails

Verify that both `id_search` and `id_pic` are present.

---

## Implementation Notes

- Base URL: `https://facecheck.id`
- HTTP client: native `fetch`
- Successful responses are parsed as JSON.
- Uploads use `FormData`.
- Image URLs are downloaded by the node before being sent to FaceCheck.
- Base64 image values are converted into binary blobs before upload.
- The node uses the `Authorization` header for authentication.
- The node sends the user agent `FusionV2/1.0`.

---

## Version History

### 1.0.0 — 2026-09-09

- Initial FaceCheck node documentation.
- Documented `getInfo`, `search`, `uploadPic`, and `deletePic`.
- Added parameter reference.
- Added dynamic input aliases.
- Added JSON configuration examples.
- Added Windows `curl.exe` test commands.
- Added validation, error handling, and troubleshooting guidance.

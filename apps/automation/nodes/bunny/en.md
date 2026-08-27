---
node_id: "bunny-net"
title: "Bunny.net"
description: "Manage Storage Zones and CDN Cache."
category: "Storage / CDN"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:

- bunny-net
- bunnycdn
- cdn
- storage
- file-upload
- cache-purge
- edge-storage

related_nodes:
- function
- if
- http-request

---

# Bunny.net

> **Category:** storage-cdn-nodes | **Type:** Action Node

Manage **Bunny.net Storage Zones** and **purge CDN cache**, and upload/delete files in Bunny's edge storage.

The **Bunny.net** node exposes four operations spanning **two different Bunny.net APIs**: the account-level Management API (list storage zones, purge cache) and the per-zone Storage API (upload/delete files) — both authenticated with the same `accessKey` field, but expecting **different kinds of keys** depending on the operation.

## ⚠️ Important: Two Different Key Types Share One Field

`accessKey` serves double duty:
- For `list_storage_zones` and `purge_cache` (Management API), it must be your **Bunny.net Account API Key**.
- For `upload_file` and `delete_file` (Storage API), it must be the **Storage Zone Password** for the specific zone being accessed.

These are **different credentials** in Bunny.net's own dashboard. Using the wrong one for a given operation will fail authentication.

### Supported Features

- List all storage zones on the account
- Purge a specific URL from Bunny's CDN cache
- Upload a file to a storage zone, as plain text or Base64-encoded binary content
- Delete a file from a storage zone
- Automatic leading-slash normalization for file paths

---

## Configuration

### Base Parameter

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `accessKey` | `string` | ✅ Yes | — | Storage Zone Password (for uploads/deletes) or Account API Key (for management/purge) — see the warning above for which one applies. |
| `operation` | `enum` | ❌ No | `"list_storage_zones"` | Operation: `list_storage_zones`, `upload_file`, `delete_file`, or `purge_cache`. |

### Storage Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `storageZoneName` | `string` | ✅ Yes (for `upload_file`, `delete_file`) | — | Name of the storage zone. |
| `path` | `string` | ❌ No | `"/"` | File path within the zone, e.g. `/images/logo.png`. |
| `fileContent` | `string` | ✅ Yes (for `upload_file`) | — | File content to upload — plain text, or Base64 if `contentEncoding` is `base64`. |
| `contentEncoding` | `enum` | ❌ No | `"text"` | `"text"` or `"base64"` — how to interpret `fileContent` before uploading. |

### Purge Parameter

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `url` | `string` | ✅ Yes (for `purge_cache`) | — | Full URL to purge from cache, e.g. `https://cdn.example.com/image.png`. |

---

## Operations

| Operation | API | Endpoint | Method |
| --------- | --- | -------- | ------ |
| `list_storage_zones` | Management | `https://api.bunny.net/storagezone` | `GET` |
| `purge_cache` | Management | `https://api.bunny.net/purge?url={url}` | `POST` |
| `upload_file` | Storage | `https://storage.bunnycdn.com/{storageZoneName}{path}` | `PUT` |
| `delete_file` | Storage | `https://storage.bunnycdn.com/{storageZoneName}{path}` | `DELETE` |

## ⚠️ Important: Fixed Storage Region

The Storage API base URL is hardcoded to `https://storage.bunnycdn.com` — **Germany's default storage region endpoint**. Bunny.net storage zones can be created in other regions (e.g. New York, Singapore, Los Angeles), each with its own regional endpoint (e.g. `ny.storage.bunnycdn.com`). If a storage zone was created in a non-default region, `upload_file`/`delete_file` calls through this node will fail or silently hit the wrong endpoint, since there is no region-selection parameter.

---

## Path Handling

For both `upload_file` and `delete_file`, `path` is normalized to ensure a leading slash:

```text
cleanPath = path.startsWith("/") ? path : "/" + path
```

So `"images/logo.png"` and `"/images/logo.png"` both resolve to the same request path.

---

## File Content Encoding

For `upload_file`, `fileContent` is sent as the request body depending on `contentEncoding`:

| `contentEncoding` | Handling |
| -------------------- | -------- |
| `"text"` (default) | `fileContent` is sent as-is (raw string body). |
| `"base64"` | `fileContent` is decoded via `Buffer.from(fileContent, "base64")` into binary before sending. |

The request always sets `Content-Type: application/octet-stream`, regardless of the actual file type being uploaded.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

Output shape varies by operation:

| Operation | Output |
| --------- | ------ |
| `list_storage_zones` | Raw JSON array of storage zone objects, as returned by Bunny's Management API. |
| `purge_cache` | `{ success: boolean }` — `true` if the HTTP response was OK. |
| `upload_file` | `{ success: true }` on success (throws on failure — see [Error Handling](#error-handling)). |
| `delete_file` | `{ success: boolean }` — `true` if the HTTP response was OK. |

Note the inconsistency: `upload_file` **throws** on failure rather than returning `{ success: false }`, while `purge_cache` and `delete_file` return `{ success: false }` without throwing.

---

## Output Example

### `list_storage_zones` (abbreviated)

```json
[
  {
    "Id": 12345,
    "Name": "my-assets-zone",
    "Region": "DE",
    "StorageUsed": 104857600
  }
]
```

### `purge_cache`

```json
{ "success": true }
```

### `upload_file`

```json
{ "success": true }
```

### `delete_file`

```json
{ "success": false }
```

---

## Configuration Examples

### List Storage Zones

```json
{
  "operation": "list_storage_zones",
  "accessKey": "your-account-api-key"
}
```

### Upload a Text File

```json
{
  "operation": "upload_file",
  "accessKey": "your-storage-zone-password",
  "storageZoneName": "my-assets-zone",
  "path": "/notes/readme.txt",
  "fileContent": "Hello from the workflow!",
  "contentEncoding": "text"
}
```

### Upload a Base64-Encoded Image

```json
{
  "operation": "upload_file",
  "accessKey": "your-storage-zone-password",
  "storageZoneName": "my-assets-zone",
  "path": "/images/logo.png",
  "fileContent": "iVBORw0KGgoAAAANSUhEUgAA...",
  "contentEncoding": "base64"
}
```

### Delete a File

```json
{
  "operation": "delete_file",
  "accessKey": "your-storage-zone-password",
  "storageZoneName": "my-assets-zone",
  "path": "/images/old-logo.png"
}
```

### Purge a Cached URL

```json
{
  "operation": "purge_cache",
  "accessKey": "your-account-api-key",
  "url": "https://cdn.example.com/images/logo.png"
}
```

---

## Workflow Integration

### Sample Workflow: Function (generate content) → Upload → Purge

```json
{
  "nodes": [
    {
      "id": "generate-report",
      "type": "function"
    },
    {
      "id": "bunny-upload",
      "type": "bunny-net",
      "config": {
        "operation": "upload_file",
        "accessKey": "your-storage-zone-password",
        "storageZoneName": "my-assets-zone",
        "path": "/reports/latest.json"
      }
    },
    {
      "id": "bunny-purge",
      "type": "bunny-net",
      "config": {
        "operation": "purge_cache",
        "accessKey": "your-account-api-key",
        "url": "https://cdn.example.com/reports/latest.json"
      }
    }
  ]
}
```

### Sample Workflow: List Zones → If → Notification

```json
{
  "nodes": [
    {
      "id": "bunny-list-zones",
      "type": "bunny-net",
      "config": {
        "operation": "list_storage_zones",
        "accessKey": "your-account-api-key"
      }
    },
    {
      "id": "check-storage-usage",
      "type": "if"
    },
    {
      "id": "notify-storage-alert",
      "type": "notification"
    }
  ]
}
```

### Sample Workflow: Cleanup → Delete File

```json
{
  "nodes": [
    {
      "id": "bunny-delete",
      "type": "bunny-net",
      "config": {
        "operation": "delete_file",
        "accessKey": "your-storage-zone-password",
        "storageZoneName": "my-assets-zone",
        "path": "/temp/old-export.csv"
      }
    }
  ]
}
```

### Common Patterns

- Function (build content) → Bunny.net (`upload_file`) → Bunny.net (`purge_cache`) — publish and invalidate CDN cache in one flow
- Schedule → Bunny.net (`list_storage_zones`) → If (usage threshold) → Notification — storage usage monitoring
- Function (cleanup logic) → Bunny.net (`delete_file`) — scheduled cleanup of stale files

---

## Error Handling

### Missing Purge URL

```text
URL is required for purge.
```

Raised when `operation` is `purge_cache` and `url` is empty.

### Missing Upload Parameters

```text
Zone name and content required.
```

Raised when `operation` is `upload_file` and either `storageZoneName` or `fileContent` is empty.

### Missing Delete Parameter

```text
Zone name required.
```

Raised when `operation` is `delete_file` and `storageZoneName` is empty.

### List Zones API Error

```text
Bunny API Error: <statusText>
```

Raised when `list_storage_zones` receives a non-OK HTTP response.

### Upload Failure

```text
Upload Failed: <statusText>
```

Raised when `upload_file` receives a non-OK HTTP response — this operation **throws** rather than returning `{ success: false }`.

### Unknown Operation

```text
Unknown operation: <operation>
```

---

## Troubleshooting

### "URL is required for purge."

**Cause**

`url` was left empty while `operation` is `purge_cache`.

**Solution**

Provide the full CDN URL to purge, e.g. `https://cdn.example.com/path/to/file`.

---

### "Zone name and content required." / "Zone name required."

**Cause**

`storageZoneName` (and/or `fileContent` for uploads) was left empty.

**Solution**

Provide the storage zone name; it must match the name shown in the Bunny.net dashboard exactly (case-sensitive).

---

### "Upload Failed: Unauthorized" or Similar Auth Error

**Cause**

The most common cause: `accessKey` is set to the **Account API Key** instead of the **Storage Zone Password** — these are different credentials, and this error means the wrong one was used for `upload_file`/`delete_file`. See [Two Different Key Types Share One Field](#️-important-two-different-key-types-share-one-field).

**Solution**

Use the Storage Zone Password from the specific zone's FTP & API Access page in the Bunny.net dashboard, not the account-level API key.

---

### `delete_file`/`purge_cache` Return `{ success: false }` With No Further Detail

**Cause**

Both operations only check `response.ok` and return a boolean — unlike `upload_file` and `list_storage_zones`, they don't throw or surface the underlying status/error text.

**Solution**

If more detail is needed on why a delete or purge failed, use an [HTTP Request](./http-request.md) node directly against the same endpoint to inspect the full response.

---

### Upload/Delete Fails Despite Correct Credentials

**Cause**

The storage zone may be hosted in a **non-default region** (not Germany) — this node always targets `storage.bunnycdn.com`, with no way to select a regional endpoint (e.g. `ny.storage.bunnycdn.com`, `sg.storage.bunnycdn.com`).

**Solution**

Check the zone's region in the Bunny.net dashboard; if it's not the default (Germany) region, this node's hardcoded endpoint won't work — an [HTTP Request](./http-request.md) node targeting the correct regional endpoint would be needed instead.

---

### Uploaded File Has Wrong Content-Type When Served

**Cause**

The upload request always sends `Content-Type: application/octet-stream`, regardless of the actual file type — Bunny.net's CDN may serve the file with this generic content type rather than inferring it from the file extension, depending on zone configuration.

**Solution**

Check the storage zone's content-type detection settings in the Bunny.net dashboard if served files need a specific `Content-Type` (e.g. `image/png`, `application/json`).

---

## Security

The node sends `accessKey` as an `AccessKey` HTTP header on every request, as required by both Bunny.net APIs.

Since `list_storage_zones`/`purge_cache` require a full **account-level** API key, that key should be treated as highly sensitive — it grants broader access than a single storage zone's password. Use the more narrowly-scoped Storage Zone Password wherever only upload/delete access is needed.

---

## Notes

The node's error-handling behavior is **inconsistent across operations**: `upload_file` and `list_storage_zones` throw on failure, while `purge_cache` and `delete_file` return `{ success: false }` silently — this is a property of the underlying implementation, not a configurable option.

The node does not:

- Support a region-selection parameter for the Storage API (always uses the Germany default endpoint)
- Support listing or browsing files within a storage zone (only upload/delete of a known path)
- Support batch upload/delete of multiple files in one call
- Set a specific `Content-Type` based on the uploaded file's extension (always `application/octet-stream`)
- Support Bunny.net's Pull Zone or DNS Zone management APIs — only storage zones and cache purge

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-27 | Initial release |
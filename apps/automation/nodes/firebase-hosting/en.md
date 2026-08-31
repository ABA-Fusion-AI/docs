---
node_id: "firebase-hosting"
title: "Firebase Hosting"
description: "Manage Hosting Sites and Releases."
category: "Cloud / Deployment"
version: "1.0.0"
language: "en"
last_updated: "2026-08-31"
author: "Fusion Team"
tags:

- firebase
- firebase-hosting
- google-cloud
- deployment
- oauth2
- sites
- releases

related_nodes:
- function
- if
- http-request

---

# Firebase Hosting

> **Category:** cloud-deployment-nodes | **Type:** Action Node

List **Firebase Hosting sites** for a project and list **releases** for a specific hosting site, via the Firebase Hosting REST API.

The **Firebase Hosting** node exposes two read-only operations: `list_sites`, which lists all Hosting sites under a given GCP/Firebase project, and `list_releases`, which lists deployment releases for a given site.

## ⚠️ Important: Known Implementation Issue — `stop()` Always Throws

This node's `stop()` method is implemented as `throw new Error("Method not implemented.")`. Any workflow engine behavior that calls `stop()` on this node (e.g. during workflow cancellation, cleanup, or a hot-reload) will encounter this error. This is worth being aware of when using this node in a workflow that may be stopped mid-execution.

### Supported Features

- List all Firebase Hosting sites for a given project
- List all releases (deployment history) for a given site
- OAuth2 Bearer token authentication (user or service account token)

### Use Cases

- Audit which Hosting sites exist under a Firebase project
- Check the deployment/release history of a specific Hosting site
- Build a deployment dashboard or monitoring workflow
- Verify a recent deployment succeeded by checking the latest release
- Feed site/release data into a `Function` node for reporting

---

## Configuration

### Base Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `accessToken` | `string` | ✅ Yes | — | Google OAuth2 access token (user or service account). Must be at least 1 character. |
| `operation` | `enum` | ❌ No | `"list_sites"` | Operation: `list_sites` or `list_releases`. |

### Operation-Specific Parameters

| Parameter | Type | Required For | Description |
| --------- | ---- | ------------- | ----------- |
| `parentParams` | `string` | `list_sites` | Parent project resource string, e.g. `projects/my-project`. |
| `siteId` | `string` | `list_releases` | Hosting site ID, e.g. `my-site`. |

---

## Operations

| Operation | Endpoint | Description |
| --------- | -------- | ----------- |
| `list_sites` | `GET /v1beta1/{parentParams}/sites` | List all Hosting sites under the given project resource. |
| `list_releases` | `GET /v1beta1/sites/{siteId}/releases` | List all releases for the given site. |

Note that `parentParams` is used **as-is** in the URL path (e.g. the caller must supply the full `projects/my-project` prefix), while `siteId` is used as a bare ID appended to a fixed `sites/` path segment.

---

## Authentication

The node authenticates every request with a Bearer token built from `accessToken`:

```text
Authorization: Bearer <accessToken>
```

`accessToken` must be a valid Google OAuth2 access token with the appropriate Firebase Hosting API scope — this node does not perform the OAuth2 flow itself, obtain, or refresh the token. It must be supplied already-valid, typically via a secrets/credential mechanism or an upstream OAuth2 node.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

The node returns the **raw JSON response** from the Firebase Hosting API for the selected operation — there is no reshaping, and notably **no `response.ok` check** before parsing the body as JSON (see [Notes](#notes)).

| Operation | Output |
| --------- | ------ |
| `list_sites` | `{ sites: [...] }` — array of Hosting site objects (`name`, `defaultUrl`, `appId`, etc.), possibly with a `nextPageToken` for pagination. |
| `list_releases` | `{ releases: [...] }` — array of release objects (`name`, `version`, `type`, `releaseTime`, `releaseUser`), possibly with a `nextPageToken`. |

---

## Output Example

### `list_sites`

```json
{
  "sites": [
    {
      "name": "projects/my-project/sites/my-site",
      "defaultUrl": "https://my-site.web.app",
      "type": "DEFAULT_SITE"
    }
  ]
}
```

### `list_releases`

```json
{
  "releases": [
    {
      "name": "sites/my-site/releases/abc123",
      "version": {
        "name": "sites/my-site/versions/xyz789",
        "status": "FINALIZED"
      },
      "type": "DEPLOY",
      "releaseTime": "2026-08-27T09:15:00Z",
      "releaseUser": {
        "email": "deployer@example.com"
      }
    }
  ]
}
```

---

## Configuration Examples

### List Sites for a Project

```json
{
  "operation": "list_sites",
  "accessToken": "ya29.your-oauth2-access-token",
  "parentParams": "projects/my-project"
}
```

### List Releases for a Site

```json
{
  "operation": "list_releases",
  "accessToken": "ya29.your-oauth2-access-token",
  "siteId": "my-site"
}
```

---

## Workflow Integration

### Sample Workflow: List Sites → Function

```json
{
  "nodes": [
    {
      "id": "firebase-sites",
      "type": "firebase-hosting",
      "config": {
        "operation": "list_sites",
        "accessToken": "ya29.your-oauth2-access-token",
        "parentParams": "projects/my-project"
      }
    },
    {
      "id": "build-site-report",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: List Releases → If → Notification

```json
{
  "nodes": [
    {
      "id": "firebase-releases",
      "type": "firebase-hosting",
      "config": {
        "operation": "list_releases",
        "accessToken": "ya29.your-oauth2-access-token",
        "siteId": "my-site"
      }
    },
    {
      "id": "check-latest-release",
      "type": "if"
    },
    {
      "id": "notify-deploy-status",
      "type": "notification"
    }
  ]
}
```

### Sample Workflow: Schedule → List Releases → Database

```json
{
  "nodes": [
    {
      "id": "firebase-releases",
      "type": "firebase-hosting",
      "config": {
        "operation": "list_releases",
        "accessToken": "ya29.your-oauth2-access-token",
        "siteId": "my-site"
      }
    },
    {
      "id": "log-deployment-history",
      "type": "database"
    }
  ]
}
```

### Common Patterns

- Schedule (post-deploy) → Firebase Hosting (`list_releases`) → If (latest release check) → Notification — deployment verification
- Firebase Hosting (`list_sites`) → Function → Dashboard rendering — Hosting site inventory
- Firebase Hosting (`list_releases`) → Database — deployment history logging

---

## Error Handling

### Missing Parent Parameter

```text
Parent (projects/...) is required.
```

Raised for `list_sites` when `parentParams` is empty.

### Missing Site ID

```text
Site ID is required.
```

Raised for `list_releases` when `siteId` is empty.

The node does **not** check `response.ok` before returning `response.json()` — an authentication failure, permission error, or malformed request from the Firebase Hosting API will be returned as the API's own error JSON body (e.g. containing an `error` field with `code`/`message`), rather than being thrown as a JavaScript error by this node. See [Notes](#notes).

---

## Troubleshooting

### "Parent (projects/...) is required."

**Cause**

`parentParams` was left empty while `operation` is `list_sites`.

**Solution**

Provide the full parent resource string, e.g. `projects/my-project` (not just the project ID alone).

---

### "Site ID is required."

**Cause**

`siteId` was left empty while `operation` is `list_releases`.

**Solution**

Provide the bare site ID, e.g. `my-site` (not the full `sites/my-site` resource path — the node adds the `sites/` prefix itself).

---

### Output Contains an `error` Field Instead of `sites`/`releases`

**Cause**

Since this node does not check `response.ok`, an authentication or permission failure (expired `accessToken`, insufficient IAM role, wrong project) results in the Firebase Hosting API's error response being returned as normal output data — for example `{ "error": { "code": 401, "message": "Request had invalid authentication credentials." } }` — rather than the node throwing.

**Solution**

Always check the response shape downstream (e.g. in a `Function` or `If` node) for an `error` field before assuming `sites`/`releases` is present. Refresh `accessToken` if the error indicates expired or invalid credentials.

---

### Empty `sites`/`releases` Array

**Cause**

The project or site genuinely has no Hosting sites/releases, or `parentParams`/`siteId` doesn't match an existing resource the authenticated identity has access to.

**Solution**

Verify the project ID and site ID in the Firebase console, and confirm the `accessToken`'s associated identity has at least viewer access to Firebase Hosting for that project.

---

### Pagination: Only a Subset of Sites/Releases Returned

**Cause**

The Firebase Hosting API paginates large result sets via a `nextPageToken` field, but this node does not expose a `pageToken` parameter or automatically follow pagination.

**Solution**

If more results are needed than a single page returns, an [HTTP Request](./http-request.md) node would be needed to manually pass `pageToken` on subsequent calls.

---

### `stop()` Throws "Method not implemented."

**Cause**

This is a known implementation gap in this node — `stop()` unconditionally throws rather than performing (or no-op-ing) a clean shutdown.

**Solution**

Be aware of this when using the node in workflows that may be stopped/cancelled mid-run; this is a property of the current node implementation, not a configuration issue to fix on the calling side.

---

## Security

The node authenticates using a Bearer OAuth2 `accessToken`, which must be obtained and kept fresh by the caller (e.g. via a secrets manager, a service account token exchange, or an upstream OAuth2 node) — this node does not manage token acquisition or refresh.

Since the node does not check `response.ok`, error responses (which could include details about the request or account) are returned as normal data rather than being filtered — treat the node's output as potentially containing error details that a calling workflow should not blindly forward to end users.

---

## Notes

Unlike most other nodes in this documentation series, this node performs **no HTTP status check** before parsing the response as JSON — both success and error responses from the Firebase Hosting API are returned identically as the node's output, differentiated only by the presence of an `error` field versus `sites`/`releases`.

The node does not:

- Check `response.ok` or throw on non-2xx HTTP responses (see above)
- Support pagination (`pageToken`) for large site/release lists
- Support creating, updating, or deleting sites, releases, or versions (list-only)
- Manage or refresh the OAuth2 `accessToken`
- Implement a working `stop()` method (always throws — see the warning above)

It is intended to provide simple, read-only visibility into Firebase Hosting sites and release history for downstream reporting and deployment-monitoring workflows.

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-31 | Initial release |
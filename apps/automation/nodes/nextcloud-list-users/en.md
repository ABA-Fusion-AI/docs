---
node_id: "nextcloud-list-users"
title: "Nextcloud - List Users"
description: "List users through the Nextcloud OCS Users API."
category: "Storage & Files"
subcategory: "Cloud Drives"
version: "1.0.0"
language: "en"
last_updated: "2026-09-18"
author: "Fusion Team"
tags:
  - nextcloud
  - user-management
  - ocs
  - users
  - cloud-storage
related_nodes:
  - nextcloud-get-user
  - nextcloud-create-user
  - nextcloud-update-user
  - nextcloud-delete-user
  - log
---

<!-- SECTION: header -->
# Nextcloud - List Users

> **Category:** Storage & Files | **Subcategory:** Cloud Drives | **Type:** Action Node

List user accounts through the Nextcloud OCS Users API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - List Users** node requests a list of users from the connected Nextcloud server. You can optionally narrow the request with a search term and pass a result limit. The node returns the server response as raw text rather than extracting user IDs.

### Use Cases

- Find an account before retrieving or updating its details.
- Review available user IDs during an administrative workflow.
- Search for accounts matching a name or identifier.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `connection` | Nextcloud connection | Yes | Server base URL, username, and password or app password used to authenticate the request. |
| `search` | string | No | Search text to pass as the `search` query parameter. Omitted when empty. |
| `limit` | number | No | Maximum result count to request through the `limit` query parameter. Omitted when zero or otherwise falsy. |

Set `connection.baseUrl` to the Nextcloud server root, such as `https://cloud.example.com`. The connected account must have permission to list users. Use an app password where supported, and protect the connection credentials.

The schema does not enforce a minimum, maximum, or integer value for `limit`. Use a positive whole number appropriate for your server. The node encodes the query parameters before sending the request.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node sends one HTTP `GET` request to `/ocs/v2.php/cloud/users`, adding `search` and `limit` to the URL when configured with truthy values. It authenticates with the connection's username and password using HTTP Basic authentication and includes the `OCS-APIREQUEST: true` header.

Incoming workflow data is ignored and does not change the request parameters. The node reads the response as text. It does not request a specific OCS response format, parse user records, paginate through additional results, or retry a failed request.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Starts the action; its data is not used to populate parameters. |
| `success` | object | Returns `{ "raw": "..." }`, containing the server response as text. |
| `error` | runtime error | Receives a network failure or a non-successful HTTP response. |

The node does not parse the OCS response body or verify an OCS-level status code inside a successful HTTP response. Inspect or parse `raw` when downstream steps need user IDs or must confirm the result reported by Nextcloud.

A network failure produces an error beginning `List users fetch failed` and includes the request URL. A non-successful HTTP response produces an error beginning `Nextcloud List Users error`, followed by the HTTP status and response text. Review these messages before forwarding them externally because they may reveal server or account details.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Example Configuration

```json
{
  "connection": {
    "baseUrl": "https://cloud.example.com",
    "username": "admin-user",
    "password": "your-admin-app-password"
  },
  "search": "alex",
  "limit": 10
}
```

Omit `search` to list users without a search term. These values are placeholders; do not publish real administrative credentials in shared workflow files.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

The supplied example contains two **Manual Trigger -> Nextcloud - List Users -> Log** flows. One requests up to 10 users without a search term; the other searches for `user1` with a limit of 5. Replace the connection values before running or sharing the workflows.

```fusion-workflow
src: example.worrkflow.json
title: List Nextcloud users
```

For production use, connect the error output to an error-handling step. Add a parsing step when the workflow needs individual user IDs from `raw`.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | What to check |
|-------|---------------|
| Authentication or permission error | Check the connection credentials and confirm that the account can list users. |
| Expected user is missing | Review `search`, `limit`, and the permissions of the connected account. |
| Limit appears to have no effect | Supply a positive whole number; zero is omitted from the request. |
| Response is raw text | Parse `raw` in a downstream step using the format returned by the server. |
| HTTP request succeeds but the response reports an error | Inspect the OCS status and message contained in `raw`. |
| Network request fails | Verify the base URL, server availability, and network connectivity. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - Get User** - Retrieve one user's details.
- **Nextcloud - Create User** - Add an account.
- **Nextcloud - Update User** - Change an account.
- **Nextcloud - Delete User** - Remove an account.
- **Log** - Inspect the raw OCS response.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-18 | Initial documentation for listing Nextcloud users through the OCS API. |

<!-- /SECTION: changelog -->

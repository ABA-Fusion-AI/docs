---
node_id: "nextcloud-get-user"
title: "Nextcloud - Get User"
description: "Get a Nextcloud user's details through the OCS Users API."
category: "Storage & Files"
subcategory: "Cloud Drives"
version: "1.0.0"
language: "en"
last_updated: "2026-09-17"
author: "Fusion Team"
tags:
  - nextcloud
  - user-management
  - ocs
  - user-details
  - cloud-storage
related_nodes:
  - nextcloud-list-users
  - nextcloud-create-user
  - nextcloud-update-user
  - nextcloud-delete-user
  - log
---

<!-- SECTION: header -->
# Nextcloud - Get User

> **Category:** Storage & Files | **Subcategory:** Cloud Drives | **Type:** Action Node

Retrieve a user's details from Nextcloud through its OCS Users API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - Get User** node requests information about one configured user ID. Use it to inspect an account before updating or deleting it, or to retrieve account information for an administrative workflow.

The connection account must have permission to view the target user's details. The node returns the OCS response as raw text and does not extract individual user fields.

### Use Cases

- Review an account before an administrative change.
- Verify that a user exists.
- Retrieve account details for auditing or workflow decisions.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `connection` | Nextcloud connection | Yes | Server base URL, username, and password or app password used to authenticate the API request. |
| `userId` | string | Yes | ID of the account to retrieve; must contain at least one character. |

Use the Nextcloud server root for `connection.baseUrl`, such as `https://cloud.example.com`. The connected account must have permission to access the requested user's information. Use an app password where supported.

The node URL-encodes `userId` before adding it to the endpoint. The schema only verifies that the value is non-empty; it does not confirm that the account exists.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node sends one HTTP `GET` request to `/ocs/v2.php/cloud/users/{userId}`. It authenticates with the connection's username and password using HTTP Basic authentication and includes the `OCS-APIREQUEST: true` header.

Incoming workflow data is ignored and does not change the configured user ID. The node reads the response as text. It does not request a specific OCS response format, parse the returned content, select individual fields, or retry a failed request.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Starts the action; its data is not used to populate parameters. |
| `success` | object | Returns `{ "raw": "..." }`, containing the server response as text. |
| `error` | runtime error | Receives a network failure or a non-successful HTTP response. |

The node does not parse the OCS response body or verify an OCS-level status code inside a successful HTTP response. Inspect or parse `raw` when downstream steps need specific fields or must confirm the result reported by Nextcloud.

A network failure produces an error beginning `Get user fetch failed` and includes the request URL. A non-successful HTTP response produces an error beginning `Nextcloud Get User error`, followed by the HTTP status and response text. Review these messages before forwarding them externally because they may reveal server or account details.

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
  "userId": "colleague-id"
}
```

These values are placeholders. Do not publish real administrative credentials in shared workflow files.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

The supplied example connects **Manual Trigger -> Nextcloud - Get User -> Log** and configures `userId` as `4`. Replace its connection values and user ID before running it. The Log node receives the raw OCS response.

```fusion-workflow
src: example.workflow.json
title: Get a Nextcloud user's details
```

For production use, connect the error output to an error-handling step. Add a parsing step after this node when the workflow needs individual fields from the response.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | What to check |
|-------|---------------|
| User not found | Confirm the exact `userId` and verify that the account exists. |
| Authentication or permission error | Check the connection credentials and confirm that the account can view user details. |
| Response is raw text | Parse `raw` in a downstream step using the format returned by the server. |
| HTTP request succeeds but the response reports an error | Inspect the OCS status and message contained in `raw`. |
| Network request fails | Verify the base URL, server availability, and network connectivity. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - List Users** - Find available user IDs.
- **Nextcloud - Create User** - Create an account.
- **Nextcloud - Update User** - Change an existing account.
- **Nextcloud - Delete User** - Remove an account.
- **Log** - Inspect the raw OCS response.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-17 | Initial documentation for retrieving Nextcloud user details through the OCS API. |

<!-- /SECTION: changelog -->

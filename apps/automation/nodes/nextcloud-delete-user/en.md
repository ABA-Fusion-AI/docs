---
node_id: "nextcloud-delete-user"
title: "Nextcloud - Delete User"
description: "Delete a Nextcloud user through the OCS Users API."
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
  - delete
  - cloud-storage
related_nodes:
  - nextcloud-create-user
  - nextcloud-get-user
  - nextcloud-update-user
  - nextcloud-list-users
  - log
---

<!-- SECTION: header -->
# Nextcloud - Delete User

> **Category:** Storage & Files | **Subcategory:** Cloud Drives | **Type:** Action Node

Delete a user account from Nextcloud through its OCS Users API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - Delete User** node sends a deletion request for one configured user ID. Use it in controlled offboarding and account-cleanup workflows. The connection account must have permission to manage users.

Deleting an account is destructive and may affect files, shares, group memberships, and other data associated with the user. Confirm the user ID and apply any required data-transfer or retention process before running the node. The node does not create a backup or ask for confirmation.

### Use Cases

- Remove an account after an approved offboarding process.
- Delete a temporary or test user.
- Clean up an obsolete account after transferring required data.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `connection` | Nextcloud connection | Yes | Server base URL, username, and password or app password used to authenticate the API request. |
| `userId` | string | Yes | ID of the account to delete; must contain at least one character. |

Use the Nextcloud server root for `connection.baseUrl`, such as `https://cloud.example.com`. The connection should use an administrative account with permission to delete the target user. An app password is recommended where supported.

The node URL-encodes `userId` before adding it to the API endpoint. The schema only verifies that the value is non-empty; it does not check that the user exists or prevent deletion of a privileged account.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node sends one HTTP `DELETE` request to `/ocs/v2.php/cloud/users/{userId}`. It authenticates with the connection's username and password using HTTP Basic authentication and includes the `OCS-APIREQUEST: true` header.

Incoming workflow data is ignored, so it does not change the configured user ID. The node does not retrieve the user first, transfer owned data, disable the account, request confirmation, or retry a failed request.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Starts the action; its data is not used to populate parameters. |
| `success` | object | Returns `{ "raw": "..." }`, containing the server response as text. |
| `error` | runtime error | Receives a network failure or a non-successful HTTP response. |

The node does not parse the OCS response body or verify an OCS-level status code inside a successful HTTP response. Inspect `raw` when downstream steps need to confirm the result reported by Nextcloud.

A network failure produces an error beginning `Delete user DELETE failed` and includes the request URL. A non-successful HTTP response produces an error beginning `Nextcloud Delete User error`, followed by the HTTP status and response text. Review these messages before forwarding them externally because they may reveal server or account details.

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
  "userId": "departing-colleague"
}
```

These values are placeholders. Do not publish real administrative credentials in shared workflow files.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

The supplied example connects **Manual Trigger -> Nextcloud - Delete User -> Log** and configures `userId` as `1`. Replace its connection and user ID before running it because execution can permanently affect the target account and its data.

```fusion-workflow
src: example.workflow.json
title: Delete a Nextcloud user
```

For production use, connect the error output to a protected error-handling step and complete any required file transfer or retention process before deletion.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | What to check |
|-------|---------------|
| User not found | Confirm the exact `userId` and verify that the account still exists. |
| Authentication or permission error | Check the connection credentials and confirm that the account can manage users. |
| HTTP request succeeds but deletion is not confirmed | Inspect the `raw` OCS response for a server-level error or status. |
| Data must be preserved | Transfer or retain the user's required files and shares before running the node. |
| Network request fails | Verify the base URL, server availability, and network connectivity. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - Create User** - Create an account.
- **Nextcloud - Get User** - Review account details before deletion.
- **Nextcloud - Update User** - Change an existing account.
- **Nextcloud - List Users** - Find and verify user IDs.
- **Log** - Inspect the raw OCS response.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-17 | Initial documentation for deleting Nextcloud users through the OCS API. |

<!-- /SECTION: changelog -->

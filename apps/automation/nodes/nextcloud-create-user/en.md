---
node_id: "nextcloud-create-user"
title: "Nextcloud - Create User"
description: "Create a Nextcloud user through the OCS Users API."
category: "Storage & Files"
subcategory: "Cloud Drives"
version: "1.0.0"
language: "en"
last_updated: "2026-09-16"
author: "Fusion Team"
tags:
  - nextcloud
  - user-management
  - ocs
  - cloud-storage
related_nodes:
  - nextcloud-get-user
  - nextcloud-update-user
  - nextcloud-delete-user
  - nextcloud-list-users
  - log
---

<!-- SECTION: header -->
# Nextcloud - Create User

> **Category:** Storage & Files | **Subcategory:** Cloud Drives | **Type:** Action Node

Create a user account in Nextcloud through its OCS Users API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - Create User** node submits a new user ID and password to the connected Nextcloud server. You can optionally supply a display name and email address. The connection account must have permission to create users.

### Use Cases

- Provision a Nextcloud account during onboarding.
- Create accounts from an approved internal workflow.
- Set a user's initial display name and email when creating the account.

Treat user creation as a privileged action. Restrict who can run the workflow and protect both the connection password and the new user's password.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `connection` | Nextcloud connection | Yes | Server base URL, username, and password or app password used to authenticate the API request. |
| `userId` | string | Yes | ID for the new account; must contain at least one character. |
| `password` | string | Yes | Initial password for the new account; must contain at least eight characters. |
| `displayName` | string | No | Display name to set when creating the account. |
| `email` | string | No | Email address to set when creating the account. |

The node sends `displayName` and `email` only when their configured values are non-empty. The code does not perform additional email-format validation. The `password` parameter is the new user's password; it is separate from the connection password.

Use the Nextcloud server root for `connection.baseUrl`, such as `https://cloud.example.com`. The node removes one trailing slash before appending the OCS endpoint.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node makes one `POST` request to `/ocs/v2.php/cloud/users`. It authenticates with the connection's username and password using HTTP Basic authentication and sends form-encoded fields:

| Form field | Configured value |
|------------|------------------|
| `userid` | `userId` |
| `password` | `password` |
| `displayName` | `displayName`, when provided |
| `email` | `email`, when provided |

The request includes `OCS-APIREQUEST: true`. Incoming workflow data is ignored; configure all values explicitly. The node does not assign groups, quotas, or administrator privileges to the new user.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Starts the action; its data is not used to populate parameters. |
| `success` | object | Returns `{ "raw": "..." }`, containing the server response as text. |
| `error` | runtime error | Receives a network failure or a non-successful HTTP response. |

The node does not parse the OCS response body or verify an OCS-level status code within a successful HTTP response. Inspect `raw` when downstream steps need to confirm the server's reported outcome.

A network failure produces an error beginning `Create user POST failed`. A non-successful HTTP response produces an error beginning `Nextcloud Create User error`, followed by the HTTP status and response text. Review error messages before forwarding them externally because a server response may contain account details.

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
  "userId": "new-colleague",
  "password": "replace-with-a-secure-password",
  "displayName": "New Colleague",
  "email": "new-colleague@example.com"
}
```

These values are placeholders. Do not publish real connection credentials or new-user passwords in shared workflow files.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

The supplied example connects **Manual Trigger → Nextcloud - Create User → Log**. It creates a user from configured parameters and sends the raw response to Log. Replace and secure its connection and password values before using or sharing the workflow.

```fusion-workflow
src: example.workflow.json
title: Create a Nextcloud user
```

Connect the error output to a protected error-handling step for production use.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | What to check |
|-------|---------------|
| User already exists | Choose a unique `userId` or inspect the existing account. |
| Password validation fails | Supply a password with at least eight characters and meet any additional server policy. |
| Authentication or permission error | Confirm the connection credentials and account's user-management permissions. |
| Request succeeds but account is not created | Inspect the `raw` OCS response for a server-level error. |
| Network request fails | Verify the base URL, server availability, and connectivity. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - Get User** — Retrieve a user's details.
- **Nextcloud - Update User** — Change an existing account.
- **Nextcloud - Delete User** — Remove an account.
- **Nextcloud - List Users** — Review available accounts.
- **Log** — Inspect the raw OCS response.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-16 | Initial documentation for creating Nextcloud users through the OCS API. |

<!-- /SECTION: changelog -->

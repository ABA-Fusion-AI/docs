---
node_id: "google-workspace-admin"
title: "Google Workspace Admin"
description: "Manage Google Workspace users, groups, memberships, and domains through the Admin SDK."
category: "web-search"
subcategory: "search-reference"
version: "1.0.0"
language: "en"
last_updated: "2026-08-04"
author: "Fusion Team"
tags: [google, workspace, admin, directory]
related_nodes: [google-contacts-action, gmail]
---

<!-- SECTION: overview -->
# Google Workspace Admin

> **Category:** Web Search&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage Workspace directory users, groups, memberships, and domains. Authenticate with an access token or let the node refresh one using OAuth client credentials.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `accessToken` | string | Conditional | Existing OAuth access token. |
| `clientId`, `clientSecret`, `refreshToken` | string | Conditional | OAuth refresh credentials used when no access token is supplied. |
| `redirectUri` | string | No | OAuth redirect URI. |
| `operation` | enum | No | User, group, membership, or domain operation. |
| `userKey` | string | Conditional | User ID or email. |
| `primaryEmail`, `givenName`, `familyName`, `userPassword` | string | For createUser | New-user fields. |
| `suspended` | string | No | `true` or `false` for user updates. |
| `groupKey`, `groupEmail`, `groupName`, `groupDescription` | string | Conditional | Group fields. |
| `memberEmail`, `memberRole` | string | Conditional | Membership fields. |
| `limit` | string | No | Maximum list result count. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Optional upstream event that triggers the configured operation.
- **Success:** Admin SDK response. List operations also expose convenient first-item identifiers.
- **Error:** OAuth or Admin SDK error details.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: List Workspace users
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store OAuth tokens, client secrets, and user passwords as workflow secrets. Grant only the Admin SDK scopes required by the selected operation.
<!-- /SECTION: security -->

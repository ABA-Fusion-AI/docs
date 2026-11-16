---
node_id: "tailscale-users"
title: "Tailscale Users"
description: "Manage Tailscale users."
category: "actions"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-11-16"
author: "Fusion Team"
tags: [tailscale, networking, users]
related_nodes: [tailscale-users, tailscale-user-invites, tailscale-devices]
---

<!-- SECTION: header -->
# Tailscale Users

> **Category:** Security & Networking → Network Tools | **Type:** Action Node
<!-- /SECTION: header -->

<!-- SECTION: overview -->
## Overview

Manage users through the Tailscale v2 REST API using the shared Axios package.
Set `accessToken` to an API access token or supported, already-issued OAuth access token. A regular workflow variable works: `{{ variables.tailscale_api_key }}`. Device enrollment keys and OAuth client secrets are not access tokens.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

`operation` defaults to `listUsers`. `tailnet` defaults to `-`, the token's default tailnet. Resource IDs are required for operations on individual resources. Fields appear only for relevant operations.

Paths are relative to `https://api.tailscale.com/api/v2`.

| Operation | Method and path | Parameters |
| --- | --- | --- |
| `listUsers` | `GET /tailnet/{tailnet}/users` | Optional userType and role filters |
| `getUser` | `GET /users/{userId}` | userId |
| `updateUserRole` | `POST /users/{userId}/role` | userId, role |
| `approveUser` | `POST /users/{userId}/approve` | userId |
| `suspendUser` | `POST /users/{userId}/suspend` | userId |
| `restoreUser` | `POST /users/{userId}/restore` | userId |
| `deleteUser` | `POST /users/{userId}/delete` | userId |

Use `userId` from `listUsers`. `userType` accepts `member` or `shared` and maps to the API `type` query parameter. Roles: `owner`, `member`, `admin`, `it-admin`, `network-admin`, `billing-admin`, `auditor`. Updating a role requires an explicit role. Tailscale enforces ownership and plan restrictions. Approve handles pending users; restore reactivates suspended users. Deletion uses POST, removes the user and their devices, and is not reversed by restore. Scoped tokens require `users:read` for reads or `users` for writes.
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

One incoming tick performs one operation. JSON responses retain their original structure; empty responses emit `{"success":true,"statusCode":204}` with the actual status code. Errors exclude credentials. Requests time out after 30 seconds, with no automatic retries. HTTP 401 indicates invalid credentials; 403 indicates insufficient permissions; 404 indicates a missing resource; 429 indicates rate limiting.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

Manual Trigger → Tailscale Users → Log. Configure `accessToken` and select `listUsers`. The preview is credential-free; configure the node after adding it.

```fusion-workflow
src: example.workflow.json
title: List Tailscale Users
```
<!-- /SECTION: examples -->

<!-- SECTION: related -->
## References

- [Official API reference](https://tailscale.com/api)
- [Credentials and scopes](https://tailscale.com/docs/reference/trust-credentials)
- [User roles](https://tailscale.com/docs/reference/user-roles)
- [Invitations](https://tailscale.com/docs/features/sharing/how-to/invite-any-user)
<!-- /SECTION: related -->

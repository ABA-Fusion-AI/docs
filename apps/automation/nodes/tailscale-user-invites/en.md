---
node_id: "tailscale-user-invites"
title: "Tailscale User Invites"
description: "Manage Tailscale user invites."
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
# Tailscale User Invites

> **Category:** Security & Networking → Network Tools | **Type:** Action Node
<!-- /SECTION: header -->

<!-- SECTION: overview -->
## Overview

Manage user invites through the Tailscale v2 REST API using the shared Axios package.
Set `accessToken` to an API access token or supported, already-issued OAuth access token. A regular workflow variable works: `{{ variables.tailscale_api_key }}`. Device enrollment keys and OAuth client secrets are not access tokens.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

`operation` defaults to `listUserInvites`. `tailnet` defaults to `-`, the token's default tailnet. Resource IDs are required for operations on individual resources. Fields appear only for relevant operations.

Paths are relative to `https://api.tailscale.com/api/v2`.

| Operation | Method and path | Parameters |
| --- | --- | --- |
| `listUserInvites` | `GET /tailnet/{tailnet}/user-invites` | tailnet |
| `createUserInvites` | `POST /tailnet/{tailnet}/user-invites` | tailnet, invites |
| `getUserInvite` | `GET /user-invites/{userInviteId}` | userInviteId |
| `deleteUserInvite` | `DELETE /user-invites/{userInviteId}` | userInviteId |
| `resendUserInvite` | `POST /user-invites/{userInviteId}/resend` | userInviteId |

Use `userInviteId` from the list or create response. Creation requires a non-empty `invites` array, sent as the top-level JSON body. Each item accepts optional `email` and `role`. Omit email to create a link without sending email; omit role for member. Allowed invite roles: `member`, `admin`, `it-admin`, `network-admin`, `billing-admin`, `auditor`. Example: `[{"email":"user@example.com","role":"member"},{}]`. Listing returns open invitations. Resend applies to open invitations originally sent by email. Use an API access token with sufficient admin permissions; do not assume a users-scoped OAuth token supports invitation endpoints. Invite URLs grant access and should not be published.
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

One incoming tick performs one operation. JSON responses retain their original structure; empty responses emit `{"success":true,"statusCode":204}` with the actual status code. Errors exclude credentials. Requests time out after 30 seconds, with no automatic retries. HTTP 401 indicates invalid credentials; 403 indicates insufficient permissions; 404 indicates a missing resource; 429 indicates rate limiting.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

Manual Trigger → Tailscale User Invites → Log. Configure `accessToken` and select `listUserInvites`. The preview is credential-free; configure the node after adding it.

```fusion-workflow
src: example.workflow.json
title: List Tailscale User Invites
```
<!-- /SECTION: examples -->

<!-- SECTION: related -->
## References

- [Official API reference](https://tailscale.com/api)
- [Credentials and scopes](https://tailscale.com/docs/reference/trust-credentials)
- [User roles](https://tailscale.com/docs/reference/user-roles)
- [Invitations](https://tailscale.com/docs/features/sharing/how-to/invite-any-user)
<!-- /SECTION: related -->

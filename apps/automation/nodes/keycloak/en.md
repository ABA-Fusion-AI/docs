---
node_id: "keycloak"
title: "Keycloak"
description: "List, read, create, update and delete Keycloak users in a realm through the Keycloak Admin REST API."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-11-24"
author: "Fusion Team"
tags:
  - keycloak
  - identity
  - iam
  - users
  - sso
related_nodes:
  - okta
  - ms-entra-id
---

<!-- SECTION: overview -->
# Keycloak

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage the users of a **Keycloak** realm from a workflow through the **Admin REST API** (`<baseUrl>/admin/realms/{realm}`). The node offers **5 operations**:

| Operation | Description |
|-----------|-------------|
| `listUsers` | List the users of a realm, with optional search and paging. |
| `getUser` | Get one user by ID. |
| `createUser` | Create a user and return its ID. |
| `updateUser` | Update the fields of a user. |
| `deleteUser` | Delete a user. |

### Use Cases

- **Onboarding:** Create a Keycloak account when a new employee or customer is added in an HR tool, a CRM or a form.
- **Offboarding:** Disable or delete accounts when someone leaves.
- **Profile sync:** Keep names and emails in Keycloak in sync with another system.
- **Audits:** List the users of a realm and send them to a sheet or a report.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Authentication

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `baseUrl` | `string` | Yes | Keycloak server URL, e.g. `https://auth.example.com` (no trailing `/admin`). |
| `accessToken` | `string` | Yes | Admin access token, sent as `Authorization: Bearer <token>`. |

Get a token from the token endpoint of the realm that holds the admin account or the service account:

```bash
curl -X POST "https://auth.example.com/realms/master/protocol/openid-connect/token" \
  -d grant_type=client_credentials \
  -d client_id=fusion-admin \
  -d client_secret=YOUR_CLIENT_SECRET
```

- **Service account (recommended):** Create a confidential client with **Client authentication** and **Service accounts roles** enabled, then give its service account the `realm-management` roles it needs (`view-users`, `query-users`, `manage-users`).
- **Admin user:** `grant_type=password`, `client_id=admin-cli`, `username`, `password` on the `master` realm.

> Access tokens are short-lived (1 minute by default on `master`). Raise **Realm settings → Tokens → Access Token Lifespan**, or refresh the token before each run.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | Yes | One of the 5 operations. Default: `listUsers`. |
| `realm` | `string` | Yes | Realm name, e.g. `master` or `my-company`. |
| `userId` | `string` | For `getUser`, `updateUser`, `deleteUser` | User ID (UUID), from `listUsers` or `createUser`. |
| `userData.username` | `string` | For `createUser` | Username. |
| `userData.email` | `string` | No | Email address. |
| `userData.firstName` | `string` | No | First name. |
| `userData.lastName` | `string` | No | Last name. |
| `userData.enabled` | `boolean` | No | Whether the account is enabled. |
| `userFilters.first` | `number` | No (`listUsers`) | Index of the first result (paging offset). |
| `userFilters.max` | `number` | No (`listUsers`) | Maximum number of results. |
| `userFilters.search` | `string` | No (`listUsers`) | Text searched in username, email, first and last name. |

> Fields are shown only for the operations that use them.

### Operations

| Operation | Request | Required | Returns |
|-----------|---------|----------|---------|
| `listUsers` | `GET /admin/realms/{realm}/users?first=&max=&search=` | `realm` | Array of users |
| `getUser` | `GET /admin/realms/{realm}/users/{userId}` | `realm`, `userId` | User |
| `createUser` | `POST /admin/realms/{realm}/users` | `realm`, `userData.username` | `{ "success": true, "id": "<new user id>" }` |
| `updateUser` | `PUT /admin/realms/{realm}/users/{userId}` | `realm`, `userId` | `{ "success": true }` |
| `deleteUser` | `DELETE /admin/realms/{realm}/users/{userId}` | `realm`, `userId` | `{ "success": true }` |

> `updateUser` only changes the fields you fill in; empty fields keep their current value.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream values (e.g. a new employee's email) in the parameters through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` / `array` | The Keycloak response, or `{ success: true }` for update and delete. |
| `error` | `Error` | Missing fields (`Realm is required`, `Realm and user ID are required`, `Realm and username are required`) or API errors `Keycloak API error (<status>): <message>`. |

### Output Examples

#### `listUsers`

```json
[
  {
    "id": "ee7174fe-af2d-488a-82ad-e05b69b6e3ba",
    "username": "admin",
    "emailVerified": false,
    "createdTimestamp": 1795534011598,
    "enabled": true,
    "totp": false,
    "disableableCredentialTypes": [],
    "requiredActions": [],
    "notBefore": 0
  }
]
```

#### `createUser`

```json
{
  "success": true,
  "id": "2f687ce6-8e9a-47a1-aaa7-f13844801493"
}
```

#### `getUser`

```json
{
  "id": "2f687ce6-8e9a-47a1-aaa7-f13844801493",
  "username": "fusion.tester",
  "firstName": "Fusion",
  "lastName": "Tester",
  "email": "fusion.tester@example.com",
  "emailVerified": false,
  "createdTimestamp": 1795534293512,
  "enabled": true,
  "totp": false,
  "disableableCredentialTypes": [],
  "requiredActions": [],
  "notBefore": 0,
  "access": {
    "manageGroupMembership": true,
    "view": true,
    "mapRoles": true,
    "impersonate": true,
    "manage": true
  }
}
```

#### `updateUser` / `deleteUser`

```json
{
  "success": true
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Keycloak in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Keycloak Node:** Runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### Search users

```json
{
  "baseUrl": "https://auth.example.com",
  "accessToken": "{{ secrets.KEYCLOAK_TOKEN }}",
  "operation": "listUsers",
  "realm": "my-company",
  "userFilters": { "search": "doe", "max": 50 }
}
```

#### Create a user from a form

```json
{
  "baseUrl": "https://auth.example.com",
  "accessToken": "{{ secrets.KEYCLOAK_TOKEN }}",
  "operation": "createUser",
  "realm": "my-company",
  "userData": {
    "username": "{{ input.username }}",
    "email": "{{ input.email }}",
    "firstName": "{{ input.firstName }}",
    "lastName": "{{ input.lastName }}",
    "enabled": true
  }
}
```

#### Delete a user

```json
{
  "baseUrl": "https://auth.example.com",
  "accessToken": "{{ secrets.KEYCLOAK_TOKEN }}",
  "operation": "deleteUser",
  "realm": "my-company",
  "userId": "2f687ce6-8e9a-47a1-aaa7-f13844801493"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the access token (or the client secret used to get it) in Fusion's **Secrets** vault. Do not commit it into version control.

- Prefer a service account with only `view-users`, `query-users` and `manage-users` over a full admin account.
- `deleteUser` cannot be undone: check the `userId` before running it.
- Use `https` for the base URL so the token is not sent in clear text.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Keycloak API error (401): HTTP 401 Unauthorized`
- **Cause:** The token is expired, or it was issued for another URL than `baseUrl` (e.g. requested on `http://localhost:8080` but used through a public URL). Keycloak checks that the token issuer matches the server URL.
- **Solution:** Get a new token from the same URL as `baseUrl`. Behind a proxy or tunnel, start Keycloak with `KC_HOSTNAME` set to its public URL (and `KC_PROXY_HEADERS=xforwarded`).

#### `Keycloak API error (403): HTTP 403 Forbidden`
- **Cause:** The account or service account lacks the `realm-management` roles.
- **Solution:** Give it `view-users`, `query-users` and `manage-users`.

#### `Keycloak API error (404): Realm not found.`
- **Cause:** Wrong realm name.
- **Solution:** Use the realm name exactly as shown in the admin console (case-sensitive).

#### `Keycloak API error (404): User not found`
- **Cause:** Wrong `userId`, or the user was deleted.
- **Solution:** Get the ID with `listUsers` (the ID is a UUID, not the username).

#### `Keycloak API error (409): User exists with same username`
- **Cause:** The username (or email) is already used in the realm.
- **Solution:** Use another username, or update the existing user.

#### `Realm and username are required`
- **Cause:** `realm` or `userData.username` is empty on `createUser`.
- **Solution:** Fill in both fields.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-24 | Full documentation: 5 operations, authentication, output examples, troubleshooting |

<!-- /SECTION: changelog -->

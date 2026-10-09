---
node_id: "ms-entra-id"
title: "Microsoft Entra ID"
description: "Microsoft Entra ID (Azure AD) - Manage users, groups, and applications via Microsoft Graph."
category: "peer-only"
subcategory: "Integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-09"
author: "Fusion Team"
tags:
  - ms-entra-id
  - azure-ad
  - microsoft-graph
  - identity
  - directory
  - integration
  - peer-only
related_nodes:
  - microsoft-todo
  - http-request
  - function
---

<!-- SECTION: overview -->
# Microsoft Entra ID

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Read the directory of a [Microsoft Entra ID](https://www.microsoft.com/security/business/identity-access/microsoft-entra-id) (formerly Azure AD) tenant through the Microsoft Graph API: list users, get one user, list groups, get one group, and list the app registrations of the tenant. All operations are read-only and support the OData options `$top`, `$filter` and `$select`.

### Use Cases

- **User lookup:** Get a user's name, e-mail and job title from their ID or user principal name before sending a notification.
- **Directory export:** List users or groups on a schedule and store them in a database or a spreadsheet.
- **Onboarding checks:** Check that a user exists (or belongs to the expected department with a `$filter`) before continuing a workflow.
- **App inventory:** List the app registrations of the tenant for audits.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | No | `getUsers` | `getUsers`, `getUser`, `getGroups`, `getGroup`, or `listApplications`. |
| `accessToken` | `string` | Yes | — | Microsoft Graph OAuth 2.0 access token (Bearer). Supports expressions. |
| `userId` | `string` | For `getUser` | — | User object ID or user principal name (e.g. `john@contoso.com`). Supports expressions. |
| `groupId` | `string` | For `getGroup` | — | Group object ID. Supports expressions. |
| `top` | `number` | No | — | Maximum number of records to return (`$top`). |
| `filter` | `string` | No | — | OData `$filter` expression, e.g. `startswith(displayName,'A')`. Supports expressions. |
| `select` | `string` | No | — | Comma-separated fields to return (`$select`), e.g. `id,displayName,mail`. Supports expressions. |

### Available Operations

| Operation | Description | Graph request | Typical permission |
|-----------|-------------|---------------|--------------------|
| `getUsers` | List the users of the tenant. | `GET /v1.0/users` | `User.ReadBasic.All` or `User.Read.All` |
| `getUser` | Get one user by ID or UPN. | `GET /v1.0/users/{userId}` | `User.ReadBasic.All` or `User.Read.All` |
| `getGroups` | List the groups of the tenant. | `GET /v1.0/groups` | `GroupMember.Read.All` or `Group.Read.All` |
| `getGroup` | Get one group by ID. | `GET /v1.0/groups/{groupId}` | `GroupMember.Read.All` or `Group.Read.All` |
| `listApplications` | List the app registrations. | `GET /v1.0/applications` | `Application.Read.All` |

`top`, `filter` and `select` are added to every request when they are set. The node does not check `userId` and `groupId` before calling the API: if one is empty, Microsoft Graph returns `404 Not Found`.

### Authentication

Every request sends `Authorization: Bearer <accessToken>`. The token must come from a **Microsoft 365 / Entra work or school account**:

- Personal Microsoft accounts (Outlook.com, Hotmail) have no directory: `getUsers`, `getUser`, `getGroups` and `getGroup` fail (`400 This API is not supported for MSA accounts` or an invalid response). Only `listApplications` answers, with an empty list.
- Group and application permissions usually require **admin consent** in the tenant. Without it, Graph returns `403 Authorization_RequestDenied`.
- The node does not refresh the token: when it expires (usually after about 1 hour), provide a new one.

### Getting an Access Token

**Quick test with Graph Explorer:**

1. Open [Graph Explorer](https://developer.microsoft.com/graph/graph-explorer) and sign in with your **work** account.
2. Open **Modify permissions** and consent to the permissions of the operations you need (see the table above). Permissions marked "Admin consent required" must be approved by a tenant administrator.
3. Run `GET https://graph.microsoft.com/v1.0/users` to check access.
4. Copy the token from the **Access token** tab and paste it into `accessToken`.

**Production:** register an application in **Microsoft Entra ID → App registrations**, add the permissions, have an administrator grant consent, and obtain tokens with the OAuth 2.0 authorization code or client credentials flow (refresh the token before each run, e.g. with an HTTP Request node).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. The incoming data is not used directly; use expressions in the parameters to pass upstream values. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The Microsoft Graph response (see examples). |
| `error` | `Error` | `Microsoft Graph API Error: <status> <statusText> <response>`, e.g. `Microsoft Graph API Error: 403 Forbidden {"error":{"code":"Authorization_RequestDenied",...}}`. |

### Output Examples

#### `getUsers`

```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users",
  "value": [
    {
      "businessPhones": [],
      "displayName": "John Doe",
      "givenName": "John",
      "jobTitle": null,
      "mail": "john@contoso.com",
      "mobilePhone": null,
      "officeLocation": null,
      "preferredLanguage": null,
      "surname": "Doe",
      "userPrincipalName": "john@contoso.com",
      "id": "fd5f2e97-0000-0000-0000-000000000000"
    }
  ]
}
```

#### `getUser`

```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
  "displayName": "John Doe",
  "mail": "john@contoso.com",
  "userPrincipalName": "john@contoso.com",
  "id": "fd5f2e97-0000-0000-0000-000000000000"
}
```

#### `getGroups`

```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#groups",
  "value": [
    {
      "id": "60767db6-0000-0000-0000-000000000000",
      "displayName": "Sales Team",
      "mail": "sales@contoso.com",
      "groupTypes": ["Unified"],
      "securityEnabled": false
    }
  ]
}
```

#### `listApplications`

```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#applications",
  "value": [
    {
      "id": "1b2c3d4e-0000-0000-0000-000000000000",
      "appId": "9f8e7d6c-0000-0000-0000-000000000000",
      "displayName": "Fusion Integration",
      "signInAudience": "AzureADMyOrg"
    }
  ]
}
```

Lists return at most 100 items per page by default; use `top` to change the page size. The node returns the first page only (`@odata.nextLink` is included when more results exist).

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Microsoft Entra ID in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **Microsoft Entra ID Node:** Runs the configured operation (e.g. `getUsers`).
3. **Log Node:** Displays the Microsoft Graph response.

### Example Configurations

#### List users (selected fields only)

```json
{
  "operation": "getUsers",
  "accessToken": "{{ secrets.MS_GRAPH_TOKEN }}",
  "top": 50,
  "select": "id,displayName,mail,jobTitle"
}
```

#### Get a user by e-mail

```json
{
  "operation": "getUser",
  "accessToken": "{{ secrets.MS_GRAPH_TOKEN }}",
  "userId": "{{ input.email }}"
}
```

#### Find groups by name

```json
{
  "operation": "getGroups",
  "accessToken": "{{ secrets.MS_GRAPH_TOKEN }}",
  "filter": "startswith(displayName,'Sales')"
}
```

### Common Patterns

- **Webhook → Microsoft Entra ID (`getUser`) → Notification:** Enrich an incoming request with the user's name and e-mail.
- **Cron → Microsoft Entra ID (`getUsers`) → Database:** Keep a daily copy of the directory.
- **Microsoft Entra ID (`listApplications`) → Function:** Report app registrations for a security review.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `accessToken` in Fusion's **Secrets** system. Never paste it directly into workflow parameters or commit it to version control.

- Request only the permissions the workflow needs (`User.ReadBasic.All` is enough to read basic user profiles).
- Directory data contains personal information (names, e-mails, phone numbers); use `select` to return only the fields you need and avoid logging full responses in shared workflows.
- Tokens copied from Graph Explorer are meant for testing only and expire after about 1 hour.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Microsoft Graph API Error: 401 Unauthorized ... InvalidAuthenticationToken`
- **Cause:** The access token is missing, malformed, or expired.
- **Solution:** Get a new token and update `accessToken`.

#### `Microsoft Graph API Error: 403 Forbidden ... Authorization_RequestDenied`
- **Cause:** The token does not include the permission needed by the operation (common for `getGroups`, `getGroup`, `listApplications`), or the permission was not approved by an administrator.
- **Solution:** Add the permission listed in **Available Operations** and ask a tenant administrator to grant admin consent.

#### `Microsoft Graph API Error: 400 Bad Request ... not supported for MSA accounts`
- **Cause:** The token comes from a personal Microsoft account, which has no directory.
- **Solution:** Use a token from a Microsoft 365 / Entra work or school account.

#### `Microsoft Graph API Error: 404 Not Found ... Request_ResourceNotFound`
- **Cause:** `userId` or `groupId` is empty or does not exist in the tenant.
- **Solution:** Copy the ID from `getUsers` / `getGroups`, or use the user's principal name for `getUser`.

#### `Microsoft Graph API Error: 400 Bad Request` with `filter`
- **Cause:** The `$filter` expression is invalid or uses an advanced query that Graph does not support without extra headers.
- **Solution:** Use simple filters such as `startswith(displayName,'A')` or `mail eq 'john@contoso.com'`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-09 | Full documentation: parameters, operations, permissions, authentication, output examples, troubleshooting |
| 1.0.0 | 2026-08-04 | Initial documentation |

<!-- /SECTION: changelog -->
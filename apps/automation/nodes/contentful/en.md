---
node_id: "contentful"

title: "Contentful"

description: "Retrieve entries, content types, assets, spaces, and individual entries from Contentful."

category: "Content Management / CMS"

version: "1.0.0"

language: "en"

last_updated: "2026-09-17"

author: "Fusion Team"

tags:

- contentful
- cms
- content-management
- entries
- assets
- content-types

related_nodes:

- http-request
- function
- if

---

**# Contentful**

> **\*\*Category:\*\*** content-management-nodes | **\*\*Type:\*\*** Action Node

Connect workflows to Contentful and retrieve content through its CDN API.

The **\*\*Contentful\*\*** node supports retrieving entries, individual entries, content types, assets, and space information using a configured access token, space ID, and environment ID.

**### Supported Features**

\- Retrieve entries

\- Retrieve a specific entry

\- Filter entries by content type

\- Retrieve content types

\- Retrieve assets

\- Retrieve space information

\- Configure the Contentful environment

\- Authenticate using a Bearer access token

\- Return the parsed Contentful response directly

**### Use Cases**

\- Read Contentful entries inside workflows

\- Retrieve structured CMS content

\- Filter entries by content type

\- Retrieve Contentful assets

\- Inspect available content types

\- Retrieve information about a configured space

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `accessToken` | `string` | ✅ Yes | — | Contentful access token used for Bearer authentication. |
| `spaceId` | `string` | ✅ Yes | — | Contentful space identifier. |
| `environmentId` | `string` | ❌ No | `"master"` | Contentful environment identifier. |
| `operation` | `enum` | ❌ No | `"getEntries"` | Operation to execute. |
| `entryId` | `string` | ❌ No | — | Entry identifier used by `getEntry`. |
| `contentType` | `string` | ❌ No | — | Optional content-type filter used by `getEntries`. |

`accessToken`, `spaceId`, `environmentId`, `entryId`, and `contentType` support expressions.

**### Supported Operations**

```text
getEntries
getEntry
getContentTypes
getAssets
getSpaces
```

Default operation:

```text
getEntries
```

**---**

**## Operations**

**### getEntries**

Retrieves entries from the configured space and environment.

Request:

```text
GET /spaces/<spaceId>/environments/<environmentId>/entries
```

If `contentType` is configured, the node appends:

```text
?content_type=<contentType>
```

Example:

```text
https://cdn.contentful.com/spaces/SPACE_ID/environments/master/entries?content_type=blogPost
```

---

**### getEntry**

Retrieves one entry.

Request:

```text
GET /spaces/<spaceId>/environments/<environmentId>/entries/<entryId>
```

The implementation inserts `entryId` directly into the URL.

---

**### getContentTypes**

Retrieves content types from the configured environment.

Request:

```text
GET /spaces/<spaceId>/environments/<environmentId>/content_types
```

---

**### getAssets**

Retrieves assets from the configured environment.

Request:

```text
GET /spaces/<spaceId>/environments/<environmentId>/assets
```

---

**### getSpaces**

Retrieves the configured space.

Request:

```text
GET https://cdn.contentful.com/spaces/<spaceId>
```

Unlike the other operations, this URL does not include `environmentId`.

**---**

**## Request Construction**

The environment-specific base URL is constructed as:

```ts
const baseUrl =
  `https://cdn.contentful.com/spaces/${spaceId}/environments/${environmentId}`;
```

Authentication uses:

```text
Authorization: Bearer <accessToken>
```

The node then performs:

```ts
fetch(url, { headers })
```

No HTTP method is explicitly supplied, so the request uses the default `GET` method.

**### Entry Filtering**

For `getEntries`:

```ts
const params = contentType ? `?content_type=${contentType}` : "";
```

When `contentType` is empty or undefined, no query parameter is added.

**---**

**## Inputs & Outputs**

**### Inputs**

`handleTick()` receives:

```text
incomingData
```

The implementation does not use incoming workflow data.

All request values come from node configuration.

**### Outputs**

The response is parsed using:

```ts
const data = await res.json();
```

On success, the parsed Contentful response is returned directly:

```ts
return data;
```

The node does not add a custom output wrapper.

**### Output Example**

The exact output depends on the selected operation and Contentful response.

Conceptually:

```json
{
  "...": "raw Contentful API response"
}
```

**---**

**## Configuration Examples**

**### Get Entries**

```json
{
  "accessToken": "YOUR_CONTENTFUL_ACCESS_TOKEN",
  "spaceId": "YOUR_SPACE_ID",
  "environmentId": "master",
  "operation": "getEntries"
}
```

**### Get Entries by Content Type**

```json
{
  "accessToken": "YOUR_CONTENTFUL_ACCESS_TOKEN",
  "spaceId": "YOUR_SPACE_ID",
  "environmentId": "master",
  "operation": "getEntries",
  "contentType": "blogPost"
}
```

**### Get Entry**

```json
{
  "accessToken": "YOUR_CONTENTFUL_ACCESS_TOKEN",
  "spaceId": "YOUR_SPACE_ID",
  "environmentId": "master",
  "operation": "getEntry",
  "entryId": "ENTRY_ID"
}
```

**### Get Content Types**

```json
{
  "accessToken": "YOUR_CONTENTFUL_ACCESS_TOKEN",
  "spaceId": "YOUR_SPACE_ID",
  "environmentId": "master",
  "operation": "getContentTypes"
}
```

**### Get Assets**

```json
{
  "accessToken": "YOUR_CONTENTFUL_ACCESS_TOKEN",
  "spaceId": "YOUR_SPACE_ID",
  "environmentId": "master",
  "operation": "getAssets"
}
```

**### Get Space**

```json
{
  "accessToken": "YOUR_CONTENTFUL_ACCESS_TOKEN",
  "spaceId": "YOUR_SPACE_ID",
  "operation": "getSpaces"
}
```

**---**

**## Workflow Integration**

**### Common Patterns**

\- Trigger → Contentful (`getEntries`)

\- Contentful (`getEntries`) → Function

\- Contentful (`getEntry`) → AI Node

\- Contentful (`getAssets`) → Data Processing

\- Contentful (`getContentTypes`) → Function

**---**

**## Error Handling**

The node parses the response before checking the HTTP status:

```ts
const data = await res.json();
```

If the HTTP response is unsuccessful:

```ts
if (!res.ok)
  throw new Error(`Contentful error: ${JSON.stringify(data)}`);
```

The resulting error format is:

```text
Contentful error: <Contentful JSON response>
```

**### Missing Entry ID**

The schema makes `entryId` optional and the implementation does not explicitly validate it for `getEntry`.

If it is missing, the URL is constructed using its current value.

**### Unknown Operation**

The schema restricts `operation` to the supported enum values.

The switch still contains a default branch that falls back to:

```text
<baseUrl>/entries
```

**---**

**## Troubleshooting**

**### Contentful Authentication Error**

Verify:

```text
accessToken
spaceId
```

Authentication is sent as:

```text
Authorization: Bearer <accessToken>
```

---

**### Wrong Environment**

The default environment is:

```text
master
```

Configure `environmentId` when content belongs to another environment.

---

**### getEntry Does Not Return the Expected Entry**

Verify that `entryId` is configured.

The current implementation does not contain an explicit:

```text
entryId required
```

validation check.

---

**### Content Type Filter Does Not Work**

The node builds the filter as:

```text
?content_type=<contentType>
```

The implementation inserts `contentType` directly into the URL and does not explicitly URL-encode it.

---

**### Incoming Data Is Ignored**

`incomingData` is accepted by `handleTick()` but is not referenced by the implementation.

Configure values through node parameters or expressions.

**---**

**## Security**

The Contentful access token is sent using:

```text
Authorization: Bearer <accessToken>
```

For production workflows:

\- Store access tokens using protected secrets or expressions

\- Never commit real Contentful tokens to Git

\- Do not include real credentials in workflow examples

\- Avoid logging Authorization headers

\- Rotate exposed credentials

**---**

**## Notes**

Metadata label:

```text
Contentful
```

Metadata description:

```text
Content management with Contentful
```

Contentful CDN base:

```text
https://cdn.contentful.com
```

Default environment:

```text
master
```

Default operation:

```text
getEntries
```

Supported operations:

```text
getEntries
getEntry
getContentTypes
getAssets
getSpaces
```

The node does not:

\- Use incoming workflow data

\- Create entries

\- Update entries

\- Delete entries

\- Upload assets

\- Implement pagination

\- Implement retries

\- Cache responses

\- Explicitly URL-encode `contentType`

\- Explicitly URL-encode `entryId`

\- Explicitly URL-encode `spaceId`

\- Explicitly URL-encode `environmentId`

The node expects Contentful responses to be JSON because it calls `res.json()` before checking `res.ok`.

The `stop()` method performs no cleanup logic.

**---**

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Contentful in a workflow
```
<!-- /SECTION: examples -->

**---**

**## Changelog**

| Version | Date | Changes |
| --- | --- | --- |
| `1.0.0` | `2026-09-17` | Initial release |

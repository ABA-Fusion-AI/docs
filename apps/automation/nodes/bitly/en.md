---
node_id: "bitly"

title: "Bitly"

description: "Shorten URLs, expand Bitlinks, retrieve click summaries, and manage Bitly groups and links."

category: "URL Management / Bitly"

version: "1.0.0"

language: "en"

last_updated: "2026-09-15"

author: "Fusion Team"

tags:

- bitly
- url-shortener
- links
- analytics
- groups
- bitlinks

related_nodes:

- http-request
- function
- if

---

**# Bitly**

> **\*\*Category:\*\*** utility-nodes | **\*\*Type:\*\*** Action Node

Integrate workflows with the **\*\*Bitly API v4\*\*** to shorten URLs, expand Bitlinks, retrieve click summaries, list groups, and retrieve Bitlinks belonging to a group.

The node authenticates with a Bitly access token and sends requests to the fixed Bitly API base URL.

**### Supported Features**

\- Shorten long URLs

\- Select a Bitly domain when shortening

\- Add an optional title

\- Expand a Bitlink

\- Retrieve Bitlink click summaries

\- Retrieve Bitly groups

\- Retrieve Bitlinks for a group

\- Bearer-token authentication

\- Return raw parsed Bitly API responses

\- Surface Bitly API errors

**### Use Cases**

\- Generate shortened URLs

\- Expand Bitlinks

\- Retrieve click statistics

\- List Bitly groups

\- Retrieve links associated with a group

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `accessToken` | `string` | ✅ Yes | — | Bitly access token used for Bearer authentication. |
| `operation` | `enum` | ❌ No | `"shorten"` | Operation to execute. |
| `longUrl` | `string` | ❌ No* | — | Long URL. Required by `shorten`. |
| `domain` | `string` | ❌ No | `"bit.ly"` | Domain used by `shorten`. |
| `title` | `string` | ❌ No | — | Optional title used by `shorten`. |
| `bitlink` | `string` | ❌ No* | — | Required by `expand` and `getClicks`. |
| `groupGuid` | `string` | ❌ No* | — | Required by `getBitlinks`. |

`accessToken`, `longUrl`, `title`, `bitlink`, and `groupGuid` support expressions.

**### Supported Operations**

```text
shorten
expand
getClicks
getGroups
getBitlinks
```

Default:

```text
shorten
```

**---**

**## Operations**

**### shorten**

Requires:

```text
longUrl
```

Request:

```text
POST /shorten
```

Body:

```json
{
  "long_url": "https://example.com/long-url",
  "domain": "bit.ly"
}
```

If `title` is truthy, the node adds:

```json
{
  "title": "Example Link"
}
```

The domain expression is:

```ts
domain || "bit.ly"
```

Missing `longUrl` throws:

```text
longUrl required
```

---

**### expand**

Requires:

```text
bitlink
```

Request:

```text
POST /expand
```

Body:

```json
{
  "bitlink_id": "bit.ly/example"
}
```

Missing `bitlink` throws:

```text
bitlink required
```

---

**### getClicks**

Requires:

```text
bitlink
```

Request:

```text
GET /bitlinks/<bitlink>/clicks/summary
```

Missing `bitlink` throws:

```text
bitlink required
```

---

**### getGroups**

Request:

```text
GET /groups
```

No operation-specific parameter is required.

---

**### getBitlinks**

Requires:

```text
groupGuid
```

Request:

```text
GET /groups/<groupGuid>/bitlinks
```

Missing `groupGuid` throws:

```text
groupGuid required
```

**---**

**## Request Body Construction**

Base URL:

```text
https://api-ssl.bitly.com/v4
```

Every request includes:

```text
Authorization: Bearer <accessToken>
Content-Type: application/json
```

The helper sends a JSON body only when `body` is truthy:

```ts
body: body ? JSON.stringify(body) : undefined
```

**### Shorten Body**

```ts
const b = {
  long_url: longUrl,
  domain: domain || "bit.ly"
};
```

If `title` is truthy:

```ts
b.title = title;
```

**### Expand Body**

```json
{
  "bitlink_id": "<bitlink>"
}
```

**---**

**## Inputs & Outputs**

**### Inputs**

`handleTick()` receives:

```text
incomingData
```

The implementation does not use it. Request values come from node configuration.

**### Outputs**

Successful responses are returned directly:

```ts
return d;
```

The node does not add a custom wrapper or transform the Bitly response.

**### Output Example**

The exact output depends on Bitly and the selected operation:

```json
{
  "...": "raw Bitly API response"
}
```

**---**

**## Configuration Examples**

**### Shorten URL**

```json
{
  "accessToken": "YOUR_BITLY_ACCESS_TOKEN",
  "operation": "shorten",
  "longUrl": "https://example.com/long-url",
  "domain": "bit.ly",
  "title": "Example Link"
}
```

**### Expand Bitlink**

```json
{
  "accessToken": "YOUR_BITLY_ACCESS_TOKEN",
  "operation": "expand",
  "bitlink": "bit.ly/example"
}
```

**### Get Click Summary**

```json
{
  "accessToken": "YOUR_BITLY_ACCESS_TOKEN",
  "operation": "getClicks",
  "bitlink": "bit.ly/example"
}
```

**### Get Groups**

```json
{
  "accessToken": "YOUR_BITLY_ACCESS_TOKEN",
  "operation": "getGroups"
}
```

**### Get Group Bitlinks**

```json
{
  "accessToken": "YOUR_BITLY_ACCESS_TOKEN",
  "operation": "getBitlinks",
  "groupGuid": "YOUR_GROUP_GUID"
}
```

**---**

**## Workflow Integration**

**### Common Patterns**

\- Trigger → Bitly (`shorten`)

\- Bitly (`shorten`) → Notification

\- Bitly (`getClicks`) → Function

\- Bitly (`getGroups`) → Function

\- Bitly (`getBitlinks`) → Data Processing

**---**

**## Error Handling**

The request helper first parses JSON:

```ts
const d = await r.json();
```

If `r.ok` is false:

```text
Bitly error (<status>): <JSON response>
```

The response is included using:

```ts
JSON.stringify(d)
```

Operation validation errors are:

```text
longUrl required
bitlink required
groupGuid required
Unknown operation: <operation>
```

**---**

**## Troubleshooting**

**### "longUrl required"**

Provide `longUrl` when using `shorten`.

---

**### "bitlink required"**

Provide `bitlink` when using `expand` or `getClicks`.

---

**### "groupGuid required"**

Provide `groupGuid` when using `getBitlinks`.

---

**### "Bitly error (...)"**

Bitly returned a non-success HTTP response. Check the access token and operation parameters.

---

**### Incoming Workflow Data Has No Effect**

`incomingData` is accepted by `handleTick()` but is not used.

Configure request values through node parameters or expressions.

**---**

**## Security**

Authentication uses:

```text
Authorization: Bearer <accessToken>
```

For production workflows:

\- Store the access token using protected secrets or expressions

\- Never commit a real Bitly access token to Git

\- Do not include real tokens in workflow examples

\- Avoid logging Authorization headers

\- Rotate exposed credentials

**---**

**## Notes**

Metadata label:

```text
Bitly
```

Metadata description:

```text
Shorten URLs, track clicks and manage links with Bitly
```

Base URL:

```text
https://api-ssl.bitly.com/v4
```

Supported operations:

```text
shorten
expand
getClicks
getGroups
getBitlinks
```

The default domain is:

```text
bit.ly
```

The node returns raw parsed Bitly JSON.

The node does not:

\- Use incoming workflow data

\- Transform successful responses

\- Implement retries

\- Implement pagination

\- Cache responses

\- Generate or refresh access tokens

\- URL-encode `bitlink` in `getClicks`

\- URL-encode `groupGuid` in `getBitlinks`

The request helper expects JSON responses because it calls `await r.json()` before checking `r.ok`.

The `stop()` method performs no cleanup logic.

**---**

**## Changelog**

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-15 | Initial release |

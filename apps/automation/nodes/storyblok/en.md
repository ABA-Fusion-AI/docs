---
node_id: "storyblok"
title: "Storyblok"
description: "List, find, create, update, publish, and delete stories in Storyblok."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-09-29"
author: "Fusion Team"
tags: [integration, storyblok, cms, content, stories]
related_nodes: []
---

<!-- SECTION: overview -->
# Storyblok

> **Category:** Peer-only Integrations | **Type:** Action Node

Manage stories in a configured Storyblok space through six operations: list, find by full slug, create, update, publish, and delete. Each execution sends one HTTP request and displays its running status.

This page documents the supplied StoryblokNode implementation. The separately exported StoryblokActionNode is not covered because its implementation was not supplied. API availability and service-side validation rules are not independently verified here.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required by schema | Default | Description |
|-----------|------|--------------------|---------|-------------|
| `operation` | enum | No | `getStories` | getStories, getStory, createStory, updateStory, publishStory, or deleteStory. |
| `accessToken` | string | Yes | None | Management API access token. |
| `spaceId` | string | Yes | None | Space used for every request. |
| `storySlug` | string | No | Unset | Full slug filter used only by getStory. |
| `storyId` | string | No | Unset | Target ID used by updateStory, publishStory, and deleteStory. |
| `name` | string | No | Unset | Story name for creation or update. |
| `content` | string | No | Unset | JSON-encoded content for creation or update. |
| `parentId` | string | No | Unset | Parent ID for creation only; converted with Number(). |
| `fullSlug` | string | No | Unset | Value sent as full_slug on creation or update. |

All fields except operation support expressions. No conditional field visibility is declared. Required string fields have no nonempty check, and operation-specific fields have no local required checks.

### Endpoint and authentication

The intended base URL is:

```text
https://mapi.storyblok.com/v1/spaces/<spaceId>
```

**Source formatting caveat:** The pasted baseUrl template contains malformed Markdown link syntax around the URL and interpolation. If present in the actual source, correct it to a plain URL template using spaceId before running the node. There is no configurable base URL.

Every request includes Content-Type: application/json and Authorization containing accessToken exactly as configured. The node does not trim the token or add a Bearer prefix. Supply a management API token privately and exclude credentials from exported examples.

Space and story IDs are interpolated directly into paths without trimming or URL encoding. The storySlug query value is encoded with encodeURIComponent.
<!-- /SECTION: configuration -->

<!-- SECTION: operations -->
## Operations

Paths below are relative to the configured space URL.

| Operation | Method | Path | Behavior |
|-----------|--------|------|----------|
| `getStories` | GET | /stories | Lists stories using the service's default request behavior. |
| `getStory` | GET | /stories?with_slug=<encoded-slug> | Filters the list when storySlug is truthy; otherwise requests /stories. |
| `createStory` | POST | /stories | Sends a story object containing name and parsed content, plus optional parent_id and full_slug. |
| `updateStory` | PUT | /stories/<storyId> | Sends a story object containing only truthy name, content, and fullSlug settings. |
| `publishStory` | GET | /stories/<storyId>/publish | Sends the publication request; this GET operation changes content state. |
| `deleteStory` | DELETE | /stories/<storyId> | Sends the deletion request. |

### List and find

getStory uses the list endpoint with a slug filter; it does not fetch by storyId or extract a single record. If storySlug is absent or empty, it behaves like getStories. Both return the entire parsed API response.

No pagination, sorting, or additional list filters are exposed. The node does not automatically retrieve subsequent pages.

### Create

When content is absent or empty, the payload contains an empty content object. Missing name becomes an empty string. Truthy parentId is converted to a number; truthy fullSlug is sent as full_slug.

Example request body:

```json
{
  "story": {
    "name": "Welcome",
    "content": { "component": "page", "title": "Welcome" },
    "parent_id": 12345,
    "full_slug": "pages/welcome"
  }
}
```

The node does not validate whether the content matches your space's components. It also does not validate parentId as a finite number: invalid numeric text becomes NaN and is serialized as null.

### Update

Only truthy name, content, and fullSlug values are included. Empty strings do not clear existing values. With none of these configured, the body is {"story": {}}. parentId and storySlug are ignored for updates.

For both creation and update, a truthy content string is parsed with JSON.parse. Whitespace-only or malformed JSON fails before the request. Any valid JSON value is accepted locally, including arrays, primitives, and null; an object shape is not enforced. Content must be configured as a string, not a raw object.

### Publish and delete

Provide the target storyId. The handler does not check that it is present: a missing ID produces /stories//publish for publication or /stories/ for update/deletion. Publication and deletion are separate operations; creation and update do not issue an additional publish request.
<!-- /SECTION: operations -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Direction | Description |
|------|-----------|-------------|
| `input` | Input | Starts execution. The incoming payload is accepted but unused. |
| `success` | Output | Parsed response JSON, or {"success": true} for HTTP 204. |
| `error` | Output | Error path for failed execution. |

Set configuration directly or use expression-enabled fields. Incoming objects are not automatically merged into the request.

Any HTTP 204 response returns:

```json
{
  "success": true
}
```

For every other status, the node parses JSON first, then checks HTTP success. A non-2xx JSON response produces Storyblok error: followed by the JSON-stringified response. Non-JSON or empty responses fail parsing before that custom error can be raised.

Successful JSON responses are returned unchanged. The node neither extracts a story nor checks an application-level success flag. Network and parsing errors propagate without a custom wrapper.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Examples

Replace token, space, story, and parent placeholders with values appropriate to your account. Example component names and fields must match your space.

### List stories

```json
{
  "accessToken": "<management-api-token>",
  "spaceId": "12345",
  "operation": "getStories"
}
```

### Find by full slug

```json
{
  "accessToken": "<management-api-token>",
  "spaceId": "12345",
  "operation": "getStory",
  "storySlug": "pages/welcome"
}
```

This returns the filtered list response without selecting its first item.

### Create a story

```json
{
  "accessToken": "<management-api-token>",
  "spaceId": "12345",
  "operation": "createStory",
  "name": "Welcome",
  "content": "{\"component\":\"page\",\"title\":\"Welcome\"}",
  "parentId": "67890",
  "fullSlug": "pages/welcome"
}
```

parentId and fullSlug are optional. Each execution submits another create request; the node implements no deduplication.

### Update a story

```json
{
  "accessToken": "<management-api-token>",
  "spaceId": "12345",
  "operation": "updateStory",
  "storyId": "98765",
  "name": "Updated welcome",
  "content": "{\"component\":\"page\",\"title\":\"Updated welcome\"}"
}
```

The supplied content is sent as one value; the node does not merge it with existing story content.

### Publish a story

```json
{
  "accessToken": "<management-api-token>",
  "spaceId": "12345",
  "operation": "publishStory",
  "storyId": "98765"
}
```

### Delete a story

```json
{
  "accessToken": "<management-api-token>",
  "spaceId": "12345",
  "operation": "deleteStory",
  "storyId": "98765"
}
```

This sends a deletion request for the configured target; the node has no preview or confirmation step.
<!-- /SECTION: examples -->

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: List Storyblok stories and inspect the response
```

Connect **Manual Trigger -> Storyblok -> Log**. Set the private management token and space ID, select getStories, then run the trigger and inspect the complete response in Log. Use the returned fields to configure subsequent steps.

The existing preview contains wiring only. Configure parameters and credentials before execution. Connect error to an error-handling branch as needed.
<!-- /SECTION: workflow-example -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Symptom | Explanation and action |
|---------|------------------------|
| `content must be a valid JSON string` | Correct the content string's JSON syntax. Whitespace-only strings also fail. |
| `Storyblok error: ...` | The service returned a non-2xx JSON response. Review its details, token permissions, space, target ID, and payload. |
| JSON parsing failure | A non-204 response was empty or not JSON. Parsing precedes the HTTP status check. |
| Invalid URL or malformed source | Correct the pasted baseUrl template to a plain URL with the space ID interpolation. |
| getStory returns a collection | It uses a filtered list endpoint and returns the response unchanged. |
| getStory returns an unfiltered list | Supply a nonempty storySlug; storyId is ignored by this operation. |
| Missing or incorrect target | Provide storyId for update, publish, and delete; it is not checked locally. |
| Update ignores parent or empty values | parentId is creation-only; empty name, content, and fullSlug strings are omitted from updates. |
| Content rejected by the service | Valid JSON syntax does not guarantee that content matches your space's component schema. |
| Fewer stories than expected | The node sends one request and implements no pagination loop. |
| `Unknown operation: <operation>` | Choose one of the six supported operation names. |

No explicit timeout, automatic retry, or rate-limit handling is implemented. stop() is a no-op and does not cancel an in-flight fetch. Remote error JSON is included in the error message, so handle errors with awareness of any content the service returns.
<!-- /SECTION: troubleshooting -->

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-29 | Regenerated documentation covering six operations, payload rules, response handling, and examples. |
<!-- /SECTION: changelog -->

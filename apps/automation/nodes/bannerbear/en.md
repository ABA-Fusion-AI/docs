---
node_id: "bannerbear"
title: "Bannerbear"
description: "Generate images and animations from Bannerbear V5 templates and retrieve render status and files."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-09-30"
author: "Fusion Team"
tags: [integration, peer-only, bannerbear, images, animations, templates]
related_nodes: []
---

<!-- SECTION: overview -->
# Bannerbear

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Generate images and animations from templates using the Bannerbear V5 API. Browse templates, submit render requests, and retrieve generated resources in later workflow steps.

Rendering is asynchronous. The node returns the API response immediately, including successful HTTP `202` responses for pending or queued renders. It does not poll for completion or download generated files.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | string enum | No | `listTemplates` | One of the eight operations listed below. |
| `apiKey` | string | Yes | None | Bannerbear V5 API key, sent as a Bearer token. Must be non-empty after trimming. |
| `templateUid` | string | For `getTemplate`, `createImage` | None | Image template UID. |
| `imageUid` | string | For `getImage` | None | Generated image UID, not the template UID. |
| `animationTemplateUid` | string | For `createAnimation` | None | Animation template UID. |
| `animationUid` | string | For `getAnimation` | None | Generated animation UID. |
| `modifications` | string | No | Omitted means `[]` | JSON-encoded array of layer overrides for image or animation creation. |
| `page` | string | No | Omitted | Positive page number for list operations; V5 returns 25 items per page according to the supplied schema. |

All parameters except `operation` support expressions. Resolve values to strings, including `page` and `modifications`.

### Operations

All paths below are relative to `https://api.bannerbear.com/v5`.

| Operation | Method | Path | Required parameters in addition to `apiKey` |
|-----------|--------|------|--------------------------------------------|
| `listTemplates` | GET | `/image_templates` | None; optional `page` |
| `getTemplate` | GET | `/image_templates/{templateUid}` | `templateUid` |
| `createImage` | POST | `/images` | `templateUid`; optional `modifications` |
| `getImage` | GET | `/images/{imageUid}` | `imageUid` |
| `listImages` | GET | `/images` | None; optional `page` |
| `listAnimationTemplates` | GET | `/animation_templates` | None; optional `page` |
| `createAnimation` | POST | `/animations` | `animationTemplateUid`; optional `modifications` |
| `getAnimation` | GET | `/animations/{animationUid}` | `animationUid` |

Required UIDs are trimmed. UIDs used in GET paths are URL-encoded and cannot be `.` or `..`. List operations append `?page=...` when configured; they do not automatically fetch additional pages.

### Layer modifications

Enter `modifications` as a string containing a JSON array:

```json
[{"name":"title","text":"Hello"}]
```

Each entry must be an object with exactly one non-empty string identifier: `name` or `id`. Providing both identifiers, a blank identifier, a nested array, or a non-object entry fails validation. Other properties are passed through to Bannerbear.

Omit the parameter or use the string `[]` to render the unchanged template. An empty string is invalid JSON. The node parses the string and wraps the array in the V5 request body:

```json
{
  "template": "YOUR_TEMPLATE_UID",
  "modifications": {
    "objects": [{"name":"title","text":"Hello"}]
  }
}
```

Do not supply the `objects` wrapper in the `modifications` parameter itself.
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Incoming data triggers the action, but the handler does not read it directly. Configure parameters explicitly or use expressions to map upstream values into them.
- **Success:** The parsed JSON API response is returned unchanged. The node does not add a wrapper, normalize fields, or validate the response structure. Resource fields such as `uid`, `status`, and `files` are preserved when present.
- **Error:** Validation, authentication, network, or remote-service failures are emitted through the error output.

### Retrieving a completed render

1. Call `createImage` or `createAnimation` with the appropriate template UID.
2. Preserve the generated resource UID from the response.
3. Add a delay in the workflow, then call `getImage` with `imageUid` or `getAnimation` with `animationUid`.
4. Inspect the returned status. Repeat retrieval with a delay while rendering is pending, and handle completion or failure in the workflow. Use available file information in a downstream step.

A successful create request does not mean that rendering has completed. Polling limits, delays, retries, and file downloads must be handled by the surrounding workflow.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Bannerbear in a workflow
```

The preview connects a manual trigger to Bannerbear and then displays the response. Configure the Bannerbear node with one of the examples below; the preview does not include credentials or operation parameters.

### List image templates

```json
{
  "operation": "listTemplates",
  "apiKey": "{{secrets.bannerbearApiKey}}",
  "page": "1"
}
```

### Create an image with a text override

```json
{
  "operation": "createImage",
  "apiKey": "{{secrets.bannerbearApiKey}}",
  "templateUid": "YOUR_IMAGE_TEMPLATE_UID",
  "modifications": "[{\"name\":\"title\",\"text\":\"Hello from Fusion\"}]"
}
```

Replace `title` with a layer name from your template. To retrieve the resulting image later, use:

```json
{
  "operation": "getImage",
  "apiKey": "{{secrets.bannerbearApiKey}}",
  "imageUid": "UID_RETURNED_BY_CREATE_IMAGE"
}
```

### Create an animation

Use `listAnimationTemplates` to discover animation templates, then configure:

```json
{
  "operation": "createAnimation",
  "apiKey": "{{secrets.bannerbearApiKey}}",
  "animationTemplateUid": "YOUR_ANIMATION_TEMPLATE_UID",
  "modifications": "[]"
}
```

Retrieve the generated resource with `getAnimation` and its returned UID in `animationUid`.
<!-- /SECTION: examples -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error or symptom | Resolution |
|------------------|------------|
| `<field> is required` | Supply a non-empty string for `apiKey` and the UID required by the selected operation. |
| `<field> must be a resource UID` | Replace `.` or `..` with the actual resource UID. |
| `modifications must be a valid JSON array of objects; use [] for no overrides` | Supply a valid JSON string, or omit the parameter. Do not use an empty string. |
| `modifications must be a JSON array of objects` | Use an array containing only non-null objects. |
| `Each modifications object requires either a non-empty name or id, not both` | Give each override exactly one valid identifier. |
| `page must be a positive integer` | Use a string such as `"1"` or `"2"`. Zero, leading zeros, signs, decimals, and values outside the JavaScript safe-integer range are rejected. Surrounding whitespace is trimmed. |
| `Bannerbear HTTP <status>: <detail>` | Inspect the returned status and service error detail, then correct credentials or request parameters as appropriate. |
| `Bannerbear HTTP <status>: expected a JSON response` | The request succeeded at the HTTP level but the response was not valid JSON. |
| `Bannerbear network error while requesting or reading the response` | Check connectivity and retry as appropriate. The node suppresses underlying transport details. |
| Render remains pending or queued | Retrieve the generated resource later; this node does not wait for rendering. |

### Migrating legacy configurations

The supplied implementation exposes only the eight operations above. It includes defensive checks for legacy settings if they survive schema parsing:

- `limit` is rejected on list operations. Use `page`; page size is not configurable through this node.
- A truthy `webhookUrl` is rejected. Configure a V5 webhook separately; this node does not register one.
- `createVideo` and `getVideo` are unsupported. Migrate to `createAnimation` and `getAnimation`, using `animationTemplateUid` and `animationUid`. An animation template is required.
- `createCollection` and `getCollection` are unsupported. The error directs users to redesign with V5 batches or workflows; neither is exposed by this node.

Unsupported operation values may be rejected by schema validation before reaching these defensive checks.
<!-- /SECTION: troubleshooting -->

<!-- SECTION: security -->
## Security

Store credentials in Fusion's credential system. Do not place secrets directly in workflow parameters or exported examples.

The node sends the resolved API key in the `Authorization` header. HTTP error details redact the configured key and strings matching the `bb_ak_v5_` token pattern. Transport failures use a generic message to avoid exposing request headers.
<!-- /SECTION: security -->

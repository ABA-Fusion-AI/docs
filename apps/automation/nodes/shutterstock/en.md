---
node_id: "shutterstock"
title: "Shutterstock"
description: "Search and access Shutterstock images and videos with support for full metadata inspection, category filtering, and asset URL resolution."
category: "media"
subcategory: "stock-media"
version: "1.0.0"
language: "en"
last_updated: "2026-11-24"
author: "Fusion Team"
tags:
  - shutterstock
  - stock-media
  - images
  - videos
  - media
  - search
  - assets
related_nodes:
  - function
  - log
  - manual-trigger
---

<!-- SECTION: overview -->
# Shutterstock

> **Category:** Stock Media&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **Shutterstock** node integrates the official Shutterstock REST API into Fusion workflows. It enables automated querying, discovery, and retrieval of high-resolution stock photography, vector graphics, illustrations, and stock footage clips.

The node supports search queries with multi-parameter filtering (category, sort order, pagination), direct asset retrieval by numeric ID, and intelligent URL parsing that automatically extracts asset IDs from standard Shutterstock browser links.

### Use Cases

- **Automated Content Generation:** Fetch royalty-free stock imagery matching articles, blog posts, or marketing campaign briefs.
- **Dynamic Video Creation:** Search and retrieve stock video clips for video rendering pipelines.
- **Asset Metadata Enrichment:** Retrieve licensing resolutions, aspect ratios, keywords, and contributor data for catalog management.
- **URL-Based Ingestion:** Extract and inspect image or video metadata directly from Shutterstock web URLs submitted via forms or webhooks.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `accessToken` | `string` | ✅ Yes | — | Shutterstock API Bearer OAuth Access Token. |
| `operation` | `enum` | ❌ No | `searchImages` | The API operation to execute:<br>‑ `searchImages`: Search image library with filters.<br>‑ `searchVideos`: Search stock footage library.<br>‑ `getImage`: Retrieve full metadata for a specific image.<br>‑ `getVideo`: Retrieve full metadata and formats for a specific video. |

---

### Operation-Specific Parameters

#### `searchImages` & `searchVideos`

When `operation` is set to `searchImages` or `searchVideos`, the `searchParams` group becomes available:

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `searchParams.query` | `string` | ❌ No | — | Free-text search keyword (e.g. `technology`, `ocean`, `morocco`). |
| `searchParams.page` | `number` | ❌ No | `1` | Page number for result pagination. |
| `searchParams.per_page` | `number` | ❌ No | `20` | Number of results to return per page (typically 1–100). |
| `searchParams.sort` | `string` | ❌ No | `popular` | Sort order (e.g. `popular`, `newest`, `relevance`). |
| `searchParams.category` | `string` | ❌ No | — | Filter by Shutterstock category name (e.g. `Technology`, `Nature`, `People`, `Business`). |

#### `getImage`

When `operation` is set to `getImage`:

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `imageId` | `string` | ✅ Yes | — | Numeric Shutterstock Image ID (e.g. `2156438243`) or full Shutterstock image URL (e.g. `https://www.shutterstock.com/image-photo/...-2156438243`). |

#### `getVideo`

When `operation` is set to `getVideo`:

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `videoId` | `string` | ✅ Yes | — | Numeric Shutterstock Video ID (e.g. `4625780`) or full Shutterstock video URL (e.g. `https://www.shutterstock.com/video/clip-4625780`). |

---

### Intelligent Asset ID Extraction

The node features an automatic extraction utility for `imageId` and `videoId`. You can provide either:
1. **A bare numeric ID:** `2156438243`
2. **A full browser URL:** `https://www.shutterstock.com/image-photo/focused-developer-coder-wears-glasses-working-2156438243`

The node extracts the numeric ID from the end of the URL slug automatically before dispatching the request to the Shutterstock REST API.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming data payload from upstream nodes. Can supply dynamic search queries or asset IDs via expression bindings. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the Shutterstock API call succeeds. |
| `error` | `object` | Emitted when validation fails or the API responds with an error status. |

---

### Output Schemas

#### Search Results (`searchImages` / `searchVideos`)

```json
{
  "total_count": 7918,
  "search_id": "2dac7bee-fc7c-417a-a4e9-f3e881621c6d",
  "page": 2,
  "per_page": 2,
  "data": [
    {
      "id": "2175128595",
      "aspect": 1,
      "assets": {
        "preview": {
          "url": "https://image.shutterstock.com/image-vector/...-2175128595.jpg",
          "width": 450,
          "height": 450
        },
        "huge_thumb": {
          "url": "https://image.shutterstock.com/image-vector/...-2175128595.jpg",
          "width": 260,
          "height": 260
        }
      },
      "description": "VR in construction isolated cartoon vector illustrations.",
      "image_type": "vector",
      "media_type": "image",
      "url": "https://www.shutterstock.com/image-photo/vr-construction-2175128595"
    }
  ]
}
```

#### Asset Details (`getImage`)

```json
{
  "id": "2156438243",
  "added_date": "2022-05-15",
  "aspect": 1.8645,
  "description": "Focused developer coder wears glasses working on computer...",
  "categories": [
    { "id": "13", "name": "People" },
    { "id": "16", "name": "Technology" }
  ],
  "keywords": [
    "analysis system",
    "developer",
    "cyber security",
    "coding"
  ],
  "assets": {
    "small_jpg": { "width": 500, "height": 268 },
    "medium_jpg": { "width": 1000, "height": 536 },
    "huge_jpg": { "width": 6000, "height": 3218 }
  }
}
```

#### Video Details (`getVideo`)

```json
{
  "id": "4625780",
  "duration": 30,
  "aspect_ratio": "16:9",
  "description": "Sparkling fresh wavy water is shining on a sunny summer day...",
  "assets": {
    "web": { "width": 426, "height": 240, "fps": 23.98, "format": "mov" },
    "sd": { "width": 852, "height": 480, "fps": 23.98, "format": "mov" },
    "hd": { "width": 1920, "height": 1080, "fps": 23.98, "format": "mov" },
    "preview_mp4": { "url": "https://ak.picdn.net/shutterstock/videos/4625780/preview/...mp4" }
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
### Example Workflow

The following workflow demonstrates dynamically generating search keywords inside a **Function** node, searching Shutterstock's image library via **Shutterstock**, and outputting results to a **Log** node.

```fusion-workflow
src: example.workflow.json
title: Search and Access Shutterstock Media
```

### Step-by-Step Walkthrough

1. **Manual Trigger:** Initiates the workflow on demand.
2. **Function Node:** Produces a search configuration object dynamically:
   ```javascript
   return {
     searchTerm: "morocco",
     limit: 2
   };
   ```
3. **Shutterstock Node:**
   - Evaluates `searchParams.query` from `outputs.Function.success.searchTerm`.
   - Evaluates `searchParams.per_page` from `outputs.Function.success.limit`.
   - Returns matched images matching the query.
4. **Log Node:** Receives and logs the asset IDs, preview URLs, and metadata to the workflow console.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Shutterstock API error (401): Invalid access token`

**Cause:** The provided `accessToken` is missing, invalid, or has expired.

**Solution:** Generate a fresh OAuth Bearer token in your Shutterstock Developer account and verify that it possesses required scopes (`licenses.view`, `media.view`).

#### `Shutterstock API error (404): Not Found`

**Cause:** The requested `imageId` or `videoId` does not exist or has been removed from Shutterstock's catalogue.

**Solution:** Verify the numeric asset ID. Ensure you are looking for an image ID when using `getImage` and a video ID when using `getVideo`.

#### `Invalid Shutterstock image/video ID`

**Cause:** The value provided to `imageId` or `videoId` is neither a numeric string nor a valid URL ending in an asset identifier.

**Solution:** Provide only the numerical ID (e.g. `2156438243`) or a standard Shutterstock URL (e.g. `https://www.shutterstock.com/image-photo/...-2156438243`).

#### `Image ID is required` / `Video ID is required`

**Cause:** The `getImage` or `getVideo` operation was executed with an empty asset ID field.

**Solution:** Supply a non-empty string in the `imageId` or `videoId` parameter.

### Error Codes

| Error Message | Cause | Resolution |
|---------------|-------|------------|
| `Shutterstock API error (401)` | Expired or incorrect access token | Update `accessToken` with a valid Bearer token |
| `Shutterstock API error (404)` | Asset not found | Check if asset ID exists on Shutterstock |
| `Invalid Shutterstock image ID` | Non-numeric string provided in `imageId` | Provide numeric ID or valid URL ending in digits |
| `Invalid Shutterstock video ID` | Non-numeric string provided in `videoId` | Provide numeric ID or valid URL ending in digits |
| `Image ID is required` | Empty `imageId` on `getImage` operation | Specify target image ID |
| `Video ID is required` | Empty `videoId` on `getVideo` operation | Specify target video ID |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](/docs/function) – Prepare dynamic search terms and parameters
- [Log](/docs/log) – Log retrieved media metadata to the console
- [Manual Trigger](/docs/manual-trigger) – Manually trigger media workflows

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-24 | Initial release of Shutterstock action node documentation |

<!-- /SECTION: changelog -->

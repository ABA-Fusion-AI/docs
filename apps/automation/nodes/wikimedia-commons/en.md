---
node_id: "wikimedia-commons"
title: "Wikimedia Commons"
description: "Get image, media, audio, and document metadata from Wikimedia Commons API."
category: "Image Search"
subcategory: "media-assets"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:
  - wikimedia
  - commons
  - images
  - media
  - audio
  - svg
  - pdf
  - mediawiki
  - metadata
related_nodes:
  - wikipedia
  - function
  - manual-trigger
  - google-images-search
  - unsplash-image-search
  - pixabay-image-search
---

<!-- SECTION: header -->
# Wikimedia Commons

> **Category:** Image Search | **Type:** Action Node

Query and retrieve direct media URLs, dimensions, MIME types, file sizes, and attribution links from **Wikimedia Commons** without requiring an API key.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Wikimedia Commons** node connects workflows to the free media repository Wikimedia Commons. It enables automated retrieval of high-resolution image links, vector graphics (SVG), audio files (OGG), animated GIFs, and PDF documents hosted on Wikimedia servers.

By querying the official MediaWiki API (`commons.wikimedia.org/w/api.php`), the node returns direct CDN download links, full license/description page URLs, file sizes in bytes, pixel dimensions (`width`, `height`), and MIME types.

### Key Capabilities

- **No Authentication / API Key Required:** Connects directly to Wikimedia's public MediaWiki API endpoints.
- **Rich Media Support:** Retrieve metadata for images (`.jpg`, `.png`, `.webp`), vectors (`.svg`), audio tracks (`.ogg`, `.mp3`, `.wav`), video (`.webm`), and documents (`.pdf`).
- **Batch Query Support:** Query multiple media files simultaneously in a single API call using pipe separation (e.g. `File:Flag of Morocco.svg|File:Flag of France.svg`).
- **Customizable Property Extraction (`iiprop`):** Choose the exact properties to fetch (`url`, `size`, `mime`, `dimensions`, `thumburl`).
- **Dynamic File Name Resolution:** Seamlessly accepts file titles from node parameters, upstream plain strings, or JSON objects via expressions.
- **Structured Normalized Output:** Unifies the API's dictionary responses into a standardized array of page and image objects.

### Common Use Cases

- **Automated Content Generation:** Fetch royalty-free Wikimedia illustrations and icons for articles, blog posts, or social media cards.
- **Vector & Logo Asset Lookup:** Retrieve official SVG vectors for country flags, corporate logos, and historical symbols.
- **AI Multimodal Pipelines:** Supply direct image URLs to Vision LLMs (e.g. Gemini, OpenAI) for image analysis.
- **Multimedia Enrichment:** Look up national anthems, historical recordings, or astronomical images dynamically.
- **Media Asset Verification:** Confirm whether a specific file exists on Wikimedia Commons before embedding it into web apps.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## How to Use the Wikimedia Commons Node

The Wikimedia Commons node can be configured directly in the visual builder or driven dynamically through upstream workflow nodes.

![Wikimedia Commons Configuration](icon.svg)

### Step-by-Step Setup in the Visual Builder

1. **Add the Node:** Drag the **Wikimedia Commons** node from the **Image Search** category onto your canvas.
2. **Connect an Upstream Trigger or Action:** Connect a trigger (such as `Manual Trigger` or `Webhook`) or an action node (such as `Function` or `Wikipedia`) to the `input` port.
3. **Specify the File Title:**
   - Enter the target file name in the **Titles** field. Include the `File:` prefix (e.g. `File:Albert Einstein Head.jpg` or `File:Flag of Morocco.svg`).
   - For batch queries, separate multiple filenames with a vertical pipe `|`.
   - Leave empty if supplying the filename dynamically from incoming workflow data.
4. **Customize Image Properties (`iiprop`):**
   - Default is `url`.
   - Use `url|size|mime|dimensions` to retrieve file size in bytes, MIME type, and width/height.
5. **Connect Downstream Nodes:** Wire the `success` output port to nodes like `Log`, `AI Chat`, `HTTP Request`, or database storage.

---

### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|:-------:|-------------|
| `titles` | `string` | ❌ No* | — | File title on Wikimedia Commons with `File:` prefix (e.g. `File:Tesla.jpg`). *Required if not passed via input. |
| `action` | `string` | ❌ No | `query` | MediaWiki API action. Default is `query`. |
| `prop` | `string` | ❌ No | `imageinfo` | MediaWiki property type. Default is `imageinfo`. |
| `iiprop` | `string` | ❌ No | `url` | Pipe-separated list of image properties to fetch (e.g. `url`, `size`, `mime`, `dimensions`). |
| `format` | `string` | ❌ No | `json` | API output format. Default is `json`. |

---

### Supported `iiprop` Values

You can combine multiple properties using the pipe delimiter (`|`):

| Property Value | Information Extracted |
|----------------|-----------------------|
| `url` | Direct CDN URL (`url`), description page URL (`description_url`), and short URL (`description_short_url`). |
| `size` | File size in bytes (`size`). |
| `mime` | MIME type string (e.g. `image/jpeg`, `image/svg+xml`, `audio/ogg`, `application/pdf`). |
| `dimensions` | Pixel dimensions (`width` and `height`). |

**Example Combinations:**
- Basic: `url`
- Detailed: `url|size|mime`
- Complete: `url|size|mime|dimensions`

---

### Batch File Querying

You can query multiple files in a single request by separating filenames with `|`:

```text
File:Flag of Morocco.svg|File:Flag of France.svg|File:Flag of Japan.svg
```

The node will return an array containing metadata for each requested file in the `pages` array.

---

### Dynamic Input Resolution

If the `titles` parameter is left empty in the node configuration, the node resolves the file title dynamically from incoming workflow data:

```
                  ┌────────────────────────────────────────┐
                  │          Incoming Input Data           │
                  └──────────────────┬─────────────────────┘
                                     │
                  Is `titles` set in node configuration?
                                     │
                     ┌───────────────┴───────────────┐
                    YES                              NO
                     │                               │
             Use config `titles`         Use incoming `input`
                                         (string payload)
```

- **String input:** If the previous node outputs a string (e.g. `"File:Wikipedia-logo-v2.svg"`), the node automatically uses it.
- **Expression mapping:** In the builder, click **Expression** on the **Titles** field to map properties from upstream objects (e.g. `{{Function.success.imageName}}`).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` | Optional incoming string containing the file title (e.g. `"File:Albert Einstein Head.jpg"`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted on successful query. Contains query parameters and an array of pages with file metadata. |
| `error` | `Error` | Emitted when validation fails (e.g. missing title) or Wikimedia API encounters an error. |

---

### Output Data Schema

```json
{
  "success": true,
  "query": {
    "titles": "File:Albert Einstein Head.jpg",
    "action": "query",
    "prop": "imageinfo",
    "iiprop": "url|size|mime|dimensions"
  },
  "pages": [
    {
      "page_id": 358482,
      "title": "File:Albert Einstein Head.jpg",
      "namespace": 6,
      "imageinfo": [
        {
          "url": "https://upload.wikimedia.org/wikipedia/commons/d/d3/Albert_Einstein_Head.jpg",
          "description_url": "https://commons.wikimedia.org/wiki/File:Albert_Einstein_Head.jpg",
          "description_short_url": "https://commons.wikimedia.org/w/index.php?curid=358482",
          "size": 651234,
          "width": 2400,
          "height": 3000,
          "thumburl": null,
          "thumbwidth": null,
          "thumbheight": null,
          "mime": "image/jpeg"
        }
      ]
    }
  ],
  "total_results": 1,
  "note": "Returns image information from Wikimedia Commons. Example: File:Tesla.jpg"
}
```

---

### Output Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates whether the query completed successfully (`true`). |
| `query` | `object` | Echoes the query parameters sent to the API (`titles`, `action`, `prop`, `iiprop`). |
| `pages` | `array` | List of page entries corresponding to the queried file(s). |
| `pages[].page_id` | `number \| null` | Unique Wikimedia Commons MediaWiki page ID. |
| `pages[].title` | `string \| null` | Full title of the file on Wikimedia Commons. |
| `pages[].namespace` | `number \| null` | MediaWiki namespace index (Namespace 6 represents File pages). |
| `pages[].imageinfo` | `array` | List of file revisions (usually 1 containing the latest active revision). |
| `pages[].imageinfo[].url` | `string \| null` | Direct, publicly accessible CDN URL to the full-resolution file. |
| `pages[].imageinfo[].description_url` | `string \| null` | URL of the Wikimedia Commons file description and license page. |
| `pages[].imageinfo[].description_short_url` | `string \| null` | Short permalink to the file page. |
| `pages[].imageinfo[].size` | `number \| null` | File size in bytes (when `size` is in `iiprop`). |
| `pages[].imageinfo[].width` | `number \| null` | Image width in pixels (when `dimensions` is in `iiprop`). |
| `pages[].imageinfo[].height` | `number \| null` | Image height in pixels (when `dimensions` is in `iiprop`). |
| `pages[].imageinfo[].mime` | `string \| null` | File MIME content type (when `mime` is in `iiprop`). |
| `total_results` | `number` | Total number of page entries returned. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Practical Examples

### Example 1: High-Resolution Photo Metadata

Fetch the direct download URL for Albert Einstein's portrait.

**Configuration:**
- **Titles:** `File:Albert Einstein Head.jpg`
- **iiprop:** `url`

**Output:**
```json
{
  "success": true,
  "pages": [
    {
      "page_id": 358482,
      "title": "File:Albert Einstein Head.jpg",
      "imageinfo": [
        {
          "url": "https://upload.wikimedia.org/wikipedia/commons/d/d3/Albert_Einstein_Head.jpg",
          "description_url": "https://commons.wikimedia.org/wiki/File:Albert_Einstein_Head.jpg"
        }
      ]
    }
  ],
  "total_results": 1
}
```

---

### Example 2: Vector Graphic (SVG) with Dimensions & MIME Type

Retrieve an SVG vector flag with full size, dimensions, and MIME information.

**Configuration:**
- **Titles:** `File:Flag of Morocco.svg`
- **iiprop:** `url|size|mime|dimensions`

**Output:**
```json
{
  "success": true,
  "pages": [
    {
      "page_id": 432109,
      "title": "File:Flag of Morocco.svg",
      "imageinfo": [
        {
          "url": "https://upload.wikimedia.org/wikipedia/commons/2/2c/Flag_of_Morocco.svg",
          "size": 948,
          "width": 900,
          "height": 600,
          "mime": "image/svg+xml"
        }
      ]
    }
  ],
  "total_results": 1
}
```

---

### Example 3: Batch Query (Multiple Files in One Request)

Query metadata for multiple country flags at the same time.

**Configuration:**
- **Titles:** `File:Flag of Morocco.svg|File:Flag of France.svg`
- **iiprop:** `url|size|mime`

**Output:**
```json
{
  "success": true,
  "pages": [
    {
      "page_id": 432109,
      "title": "File:Flag of Morocco.svg",
      "imageinfo": [
        {
          "url": "https://upload.wikimedia.org/wikipedia/commons/2/2c/Flag_of_Morocco.svg",
          "mime": "image/svg+xml"
        }
      ]
    },
    {
      "page_id": 128456,
      "title": "File:Flag of France.svg",
      "imageinfo": [
        {
          "url": "https://upload.wikimedia.org/wikipedia/commons/c/c3/Flag_of_France.svg",
          "mime": "image/svg+xml"
        }
      ]
    }
  ],
  "total_results": 2
}
```

---

### Example 4: Audio File Metadata (`.ogg`)

Retrieve metadata for an audio recording.

**Configuration:**
- **Titles:** `File:Morocco national anthem.ogg`
- **iiprop:** `url|size|mime`

**Output:**
```json
{
  "success": true,
  "pages": [
    {
      "page_id": 564738,
      "title": "File:Morocco national anthem.ogg",
      "imageinfo": [
        {
          "url": "https://upload.wikimedia.org/wikipedia/commons/9/91/Morocco_national_anthem.ogg",
          "size": 1820490,
          "mime": "audio/ogg"
        }
      ]
    }
  ],
  "total_results": 1
}
```

---

### Example 5: Animated GIF & PDF Documents

Query animated GIFs or PDF documents hosted on Wikimedia Commons.

**Animated GIF:**
- **Titles:** `File:Rotating earth (large).gif`
- **iiprop:** `url|size|mime|dimensions`
- **MIME Output:** `image/gif`

**PDF Document:**
- **Titles:** `File:Sample.pdf`
- **iiprop:** `url|size|mime`
- **MIME Output:** `application/pdf`

---

### Example 6: Dynamic Expression from Function Node

Pass a dynamic image name calculated in an upstream JavaScript Function.

**Upstream Function Code:**
```javascript
return {
  "imageName": "File:Wikipedia-logo-v2.svg"
};
```

**Wikimedia Commons Configuration:**
- **Titles (Expression):** `{{Function.success.imageName}}`
- **iiprop:** `url|size|mime`

---

### Example 7: Handling Non-Existent Files

Querying a filename that does not exist on Wikimedia Commons.

**Configuration:**
- **Titles:** `File:random_non_existent_image_123456789.png`

**Output:**
```json
{
  "success": true,
  "pages": [
    {
      "page_id": -1,
      "title": "File:random_non_existent_image_123456789.png",
      "namespace": 6,
      "imageinfo": []
    }
  ],
  "total_results": 1
}
```

> **Tip:** If a file does not exist, `page_id` is `-1` and `imageinfo` is empty `[]`. You can easily filter missing files using an **If** node checking `imageinfo.length > 0`.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Interactive Workflow Preview

```fusion-workflow
src: example.workflow.json
title: Wikimedia Commons Media Retrieval Workflow
```

---

### Sample Workflows

#### 1. Direct Image Lookup: Trigger ➔ Wikimedia Commons ➔ Log

A simple test workflow to retrieve direct image URLs and inspect file metadata:

```json
{
  "nodes": [
    {
      "id": "trigger",
      "type": "manual-trigger",
      "label": "Fetch Media"
    },
    {
      "id": "wikimedia",
      "type": "wikimedia-commons",
      "config": {
        "titles": "File:Albert Einstein Head.jpg",
        "iiprop": "url|size|mime|dimensions"
      }
    },
    {
      "id": "log-output",
      "type": "log",
      "label": "Inspect Output"
    }
  ],
  "connections": [
    {
      "source": "trigger",
      "target": "wikimedia"
    },
    {
      "source": "wikimedia",
      "target": "log-output"
    }
  ]
}
```

---

#### 2. Wikipedia ➔ Wikimedia Commons Asset Enrichment Pipeline

A workflow where a Wikipedia article lookup extracts lead image names, and queries Wikimedia Commons for the high-res SVG or JPG download URL:

```
  ┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
  │  Manual Trigger  │ ────▶ │  Wikipedia Node  │ ────▶ │  Function Filter │ ────▶ │ Wikimedia Node   │
  │   (Search topic) │       │ (GetById: Rabat) │       │ (Extract image)  │       │ (Get high-res URL│
  └──────────────────┘       └──────────────────┘       └──────────────────┘       └──────────────────┘
```

---

### Architecture Patterns

- **Multimodal AI Pipeline:** `Trigger ➔ Wikimedia Commons (Fetch JPG URL) ➔ AI Chat / Vision Model (Analyze visual contents)`.
- **Dynamic Asset CDN:** `Database (Product / Entity) ➔ Function (Format File:... title) ➔ Wikimedia Commons ➔ Frontend Asset Cache`.
- **Media Archiver:** `Schedule ➔ Wikimedia Commons (Batch Flags query) ➔ Cloud Storage / S3 (Archive vector files)`.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues & Resolutions

#### `Wikimedia Commons lookup failed: Titles parameter is required`
- **Cause:** The `titles` field was not specified in the configuration, and no string input was provided by the upstream node.
- **Solution:** Enter a valid file name in the `titles` parameter or ensure the preceding node outputs a non-empty string.

---

#### `imageinfo array is empty ([]) and page_id is -1`
- **Cause:** The specified file was not found on Wikimedia Commons.
- **Solution:**
  - Verify that the filename includes the `File:` prefix (e.g. `File:Tesla.jpg`, not just `Tesla.jpg`).
  - Verify exact spelling, spaces, and case sensitivity on Wikimedia Commons.

---

#### `Missing dimensions or size fields in response`
- **Cause:** The `iiprop` parameter was set to `url` only.
- **Solution:** Set `iiprop` to `url|size|mime|dimensions` to request full metadata.

---

#### `Wikimedia Commons API error: 429 Too Many Requests`
- **Cause:** High volume of rapid automated requests hitting Wikimedia's rate limiters.
- **Solution:** Insert a `Delay` or `Throttle` node in batch workflows.

---

### Error Reference Table

| Error Message | Cause | Resolution |
|---------------|-------|------------|
| `Titles parameter is required` | Empty title input | Specify `titles` or pass string data from upstream. |
| `Wikimedia Commons API error: 404` | Endpoint not found | Verify network connectivity to `commons.wikimedia.org`. |
| `Wikimedia Commons API error: 429` | Rate limit exceeded | Space out requests using a Delay node. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Wikipedia](../wikipedia/en.md) — Search encyclopedia articles and summaries
- [Function](../function/en.md) — Format file names and process media payloads
- [Manual Trigger](../manual-trigger/en.md) — Manually trigger media retrieval workflows
- [Google Images Search](../google-images-search/en.md) — Search Google Images
- [Unsplash Image Search](../unsplash-image-search/en.md) — Query high-resolution stock photos

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-27 | Initial release of Wikimedia Commons Action Node with batch querying, dynamic inputs, and customizable `iiprop` properties |

<!-- /SECTION: changelog -->

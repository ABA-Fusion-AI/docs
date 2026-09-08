---
node_id: "bing-images-search"
title: "Bing Images Search"
description: "Search Bing Images by scraping results (official API retired). Returns image URLs, titles, and metadata."
category: "web-search-information"
subcategory: "image-search"
version: "1.0.0"
language: "en"
last_updated: "2026-09-08"
author: "Fusion Team"
tags:
  - bing
  - images
  - search
  - scraping
  - web
related_nodes:
  - bing-copilot-search
  - quick-chart
---

<!-- SECTION: header -->
# Bing Images Search

> **Category:** Web Search & Information | **Type:** Action Node

Search Bing Images by scraping public search result pages and return image URLs, titles, thumbnails, and basic result metadata.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Bing Images Search** node performs image searches on Bing without using the retired official Bing Images API.

It sends a standard HTTP request to Bing Images, parses the returned HTML with Cheerio, extracts image information, removes duplicate URLs, and returns a limited list of results.

### Key Features

- Searches Bing Images without an API key.
- Supports configurable result counts.
- Supports image size filters.
- Supports color filters.
- Supports image type filters.
- Supports license filters.
- Extracts image URLs, titles, and thumbnails.
- Removes duplicate image URLs.
- Limits returned results to the configured count.

### Processing Flow

```text
Receive workflow input
  ↓
Resolve search query
  ↓
Build Bing Images search URL
  ↓
Apply optional filters
  ↓
Send HTTP GET request
  ↓
Parse HTML with Cheerio
  ↓
Extract image results
  ↓
Remove duplicates
  ↓
Limit results
  ↓
Return search response
```

### Use Cases

- Searching for images from workflow automation.
- Collecting image URLs for downstream processing.
- Building image discovery workflows.
- Searching by size, color, type, or license.
- Prototyping visual-content pipelines without an API key.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | `string` | No* | — | Search query. Required in practice unless a string input is provided by the previous node. |
| `count` | `number` | No | `20` | Maximum number of results to return. The request count is capped at `150`. |
| `size` | `string` | No | `all` | Image size filter: `small`, `medium`, `large`, or `all`. |
| `color` | `string` | No | — | Optional color filter. |
| `imageType` | `string` | No | — | Optional image type filter. |
| `license` | `string` | No | — | Optional image license filter. |

### Query Resolution

The node resolves the search query in this order:

```text
Configured query
  ↓
Incoming string data
  ↓
String conversion of incoming data
```

If the resulting query is empty, the node throws:

```text
Search query is required
```

### Count

Example:

```text
count: 5
```

The node limits its output using:

```text
results.slice(0, count || 20)
```

The Bing request itself caps the count at:

```text
150
```

### Size

Supported values:

```text
small
medium
large
all
```

When `all` is selected, no size filter is added to the request.

### Color

Supported values include:

```text
color
monochrome
black
blue
green
orange
purple
red
teal
white
yellow
```

### Image Type

Supported values:

```text
AnimatedGif
Clipart
Line
Photo
Shopping
Transparent
```

### License

Supported values:

```text
All
Public
Share
ShareCommercially
Modify
ModifyCommercially
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node accepts workflow input from a previous node.

If `query` is configured directly, incoming data is not needed for query resolution.

If `query` is empty and the incoming data is a string, that string becomes the search query.

### Success Output

Example:

```json
{
  "query": "cat",
  "results": [
    {
      "url": "https://example.com/image.jpg",
      "title": "Example image title",
      "thumbnail": "https://example.com/thumb.jpg"
    }
  ],
  "total": 1
}
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `query` | `string` | Search query used for the request. |
| `results` | `array` | Extracted image results. |
| `results[].url` | `string` | Image URL. |
| `results[].title` | `string` | Image title or alt text when available. |
| `results[].thumbnail` | `string` | Thumbnail URL. |
| `total` | `number` | Number of returned results. |

### Error Output

If Bing returns a non-success HTTP response, the node throws an error similar to:

```text
Bing Images search failed: HTTP error! status: 403
```

Other failures are wrapped as:

```text
Bing Images search failed: <message>
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Basic Image Search

```text
Query: cat
Count: 5
Size: all
Color: [empty]
ImageType: [empty]
License: [empty]
```

This returns up to five image results.

### Example 2: Size Filter

```text
Query: car
Count: 3
Size: large
```

This searches for large images and returns up to three results.

### Example 3: Color Filter

```text
Query: flower
Count: 3
Size: all
Color: red
```

This applies Bing's red color filter.

### Example 4: Image Type Filter

```text
Query: cat
Count: 3
Size: all
ImageType: Photo
```

This applies the photo image type filter.

### Example 5: License Filter

```text
Query: nature
Count: 3
Size: all
License: Public
```

This applies Bing's public license filter.

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Bing Images Search Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Search Query Is Required

**Cause:** No configured query is available and the incoming workflow data does not provide a usable string query.

**Solution:** Configure the `query` field directly or pass a string from the previous node.

### No Results Returned

**Cause:** Bing may return HTML that does not contain selectors currently handled by the node, or the applied filters may be too restrictive.

**Solution:** Try a broader query and remove optional filters.

### Bing Returns an HTTP Error

**Cause:** Bing may block, rate-limit, or reject automated requests.

**Solution:** Retry later and avoid excessively frequent requests.

### Result Count Is Lower Than Requested

The configured `count` is a maximum, not a guarantee.

Bing may return fewer extractable results than requested.

### Filters Do Not Guarantee Semantic Accuracy

The node forwards size, color, image type, and license filters to Bing, but the returned results depend on Bing's interpretation of those parameters.

### Scraping Depends on Bing HTML

This node relies on HTML scraping rather than an official API.

Changes to Bing's page structure or selectors may affect extraction behavior.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Bing Copilot Search** — Search Bing/Copilot web results using scraping.
- **Quick Chart** — Generate charts from workflow data.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-08 | Initial documentation for Bing Images Search. |

<!-- /SECTION: changelog -->
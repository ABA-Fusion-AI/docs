---
node_id: "quick-chart"
title: "QuickChart"
description: "Generate charts using QuickChart API."
category: "devops-cloud-management"
subcategory: "developer-tools"
version: "1.0.0"
language: "en"
last_updated: "2026-08-11"
author: "Fusion Team"
tags:
  - charts
  - quickchart
  - visualization
  - image
  - api
  - developer-tools
related_nodes:
  - barcode-generator
  - json-placeholder
  - lorem-picsum
---

<!-- SECTION: header -->
# QuickChart

> **Category:** DevOps & Cloud Management | **Type:** Action Node

Generate chart images or SVG content using the QuickChart API from a Chart.js-compatible configuration.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **QuickChart** node generates charts by sending a chart configuration to the QuickChart API.

The node accepts a Chart.js-compatible configuration as a string, together with optional width, height, and output format settings.

Depending on the selected format, the node returns either:

- an image as a Base64 data URL;
- or the API response as text for non-image formats.

### Key Features

- **Chart Generation:** Generates charts using QuickChart.
- **Chart.js Configuration:** Accepts chart configuration as a string.
- **Configurable Dimensions:** Supports custom width and height.
- **Multiple Image Formats:** Supports PNG, JPG, GIF, and WebP image responses.
- **Text Output for Non-Image Formats:** For formats other than PNG, JPG, GIF, and WebP, the response is returned as text in `chart_svg`.
- **Base64 Image Output:** Encodes image responses as Base64 data URLs.
- **Direct Chart URL:** Returns the generated QuickChart request URL.
- **Workflow Input Fallback:** Can use incoming workflow data when `chartConfig` is not configured.

### Processing Flow

```text
Chart configuration
        ↓
Build QuickChart request
        ↓
Call QuickChart API
        ↓
Check HTTP response
        ↓
Image format?
   ┌────┴────┐
  Yes       No
   ↓         ↓
Base64     Text / SVG
   ↓         ↓
Return result
```

### Use Cases

- Generate dashboards and reports
- Create charts from workflow data
- Generate bar, line, pie, or other Chart.js charts
- Produce images for emails or HTML content
- Generate SVG charts
- Convert structured metrics into visual output

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `chartConfig` | `string` | ✅ Yes | — | Chart.js-compatible configuration sent to QuickChart. |
| `width` | `number` | ❌ No | `500` | Output chart width. |
| `height` | `number` | ❌ No | `300` | Output chart height. |
| `format` | `string` | ❌ No | `png` | Requested chart output format. |

### Chart Config

`chartConfig` is sent directly to QuickChart as the `c` query parameter.

Example:

```json
{
  "type": "bar",
  "data": {
    "labels": [
      "January",
      "February",
      "March"
    ],
    "datasets": [
      {
        "label": "Sales",
        "data": [
          12,
          19,
          8
        ]
      }
    ]
  }
}
```

In the node configuration, this value is stored as a string.

### Width

Default:

```text
500
```

The configured value is converted to a string and sent to QuickChart as the `width` query parameter.

### Height

Default:

```text
300
```

The configured value is converted to a string and sent to QuickChart as the `height` query parameter.

### Format

Default:

```text
png
```

The current implementation handles these formats as image responses:

```text
png
jpg
gif
webp
```

These formats are converted to Base64 data URLs.

Other formats, including SVG, are handled as text responses.

### Input Fallback

The node first uses the configured:

```text
chartConfig
```

> The runtime handler supports incoming workflow data as a fallback, although `chartConfig` is declared as required in the node schema.

If no configured value is available, it uses incoming workflow data.

If the incoming value is a string:

```ts
data
```

is used directly.

If the incoming value is not a string:

```ts
JSON.stringify(data)
```

is used.

If no usable chart configuration exists, the node throws:

```text
Chart configuration is required (JSON string)
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `chartConfig` | `string` | Configured chart definition. |
| `input` | `unknown` | Incoming workflow value used as fallback when `chartConfig` is empty. |

### Image Output

For:

```text
png
jpg
gif
webp
```

the node returns:

```json
{
  "success": true,
  "chart_url": "https://quickchart.io/chart?...",
  "chart_base64": "data:image/png;base64,...",
  "format": "png",
  "width": 500,
  "height": 300,
  "note": "Use chart_base64 for embedding in HTML or chart_url for direct image link"
}
```

### Image Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates successful generation. |
| `chart_url` | `string` | Final QuickChart request URL. |
| `chart_base64` | `string` | Base64 data URL containing the generated image. |
| `format` | `string` | Requested format. |
| `width` | `number` | Requested output width. |
| `height` | `number` | Requested output height. |
| `note` | `string` | Usage guidance returned by the node. |

### SVG / Text Output

For formats not handled as image responses, the node reads the API response as text.

For SVG, the returned structure is:

```json
{
  "success": true,
  "chart_url": "https://quickchart.io/chart?...",
  "chart_svg": "<svg>...</svg>",
  "format": "svg",
  "width": 500,
  "height": 300,
  "note": "Use chart_svg for SVG format or chart_url for direct link"
}
```

### SVG Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates successful generation. |
| `chart_url` | `string` | Final QuickChart request URL. |
| `chart_svg` | `string` | Text response returned by QuickChart. |
| `format` | `string` | Requested format. |
| `width` | `number` | Requested output width. |
| `height` | `number` | Requested output height. |
| `note` | `string` | Usage guidance returned by the node. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Bar Chart

The following configuration was successfully tested:

```json
{
  "type": "bar",
  "data": {
    "labels": [
      "January",
      "February",
      "March"
    ],
    "datasets": [
      {
        "label": "Sales",
        "data": [
          12,
          19,
          8
        ]
      }
    ]
  }
}
```

Configuration:

```text
Width  = 500
Height = 300
Format = png
```

The node successfully returns:

```text
success      = true
chart_url    = QuickChart URL
chart_base64 = data:image/png;base64,...
format       = png
width        = 500
height       = 300
```

---

### Example: PNG Output

With:

```text
Format = png
```

the node returns an image encoded as:

```text
data:image/png;base64,...
```

---

### Example: JPG Output

With:

```text
Format = jpg
```

the node returns:

```text
data:image/jpg;base64,...
```

---

### Example: SVG Output

With:

```text
Format = svg
```

the node reads the API response as text and returns it in:

```text
chart_svg
```

Example response structure:

```json
{
  "success": true,
  "chart_url": "https://quickchart.io/chart?...",
  "chart_svg": "<svg>...</svg>",
  "format": "svg",
  "width": 500,
  "height": 300
}
```

---

### Example: Incoming String

When `chartConfig` is not configured, an incoming string can be used directly:

```text
{"type":"line","data":{"labels":["A","B","C"],"datasets":[{"data":[1,2,3]}]}}
```

The string is sent directly to QuickChart.

---

### Example: Incoming Object

When incoming workflow data is an object, the node applies:

```text
JSON.stringify(data)
```

before sending it to QuickChart.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate a Chart with QuickChart
```

### Common Patterns

- **Generate Chart:** Manual Trigger → QuickChart → Log
- **Generate PNG:** Manual Trigger → QuickChart → Log
- **Generate SVG:** Manual Trigger → QuickChart → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### "Chart configuration is required (JSON string)"

**Cause**

No configured `chartConfig` and no usable incoming workflow input were available.

**Solution**

Provide a valid chart configuration either directly in the node or through incoming workflow data.

---

### "QuickChart API error"

**Cause**

QuickChart returned a non-successful HTTP status.

The node checks:

```text
response.ok
```

and throws an error containing the HTTP status.

Example:

```text
QuickChart generation failed: QuickChart API error: 400
```

**Solution**

Verify:

- the chart configuration;
- the output format;
- the width and height;
- QuickChart API availability.

---

### Invalid Chart Configuration

The node itself does not parse or validate the `chartConfig` string before sending it to QuickChart.

A malformed or unsupported configuration may therefore be rejected by the QuickChart API.

---

### Base64 Output

For:

```text
png
jpg
gif
webp
```

the API response is converted to:

```text
data:image/<format>;base64,...
```

Use:

```text
chart_base64
```

for embedding the image in HTML or compatible output.

---

### SVG Output

SVG is not converted to Base64 by the current implementation.

The response is read as text and returned through:

```text
chart_svg
```

---

### Default Dimensions

If no width or height is configured, the node uses:

```text
width  = 500
height = 300
```

The runtime also applies these fallback values before calling the API.

---

### Error Handling

Errors generated during the API request are wrapped as:

```text
QuickChart generation failed: <error message>
```

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- Barcode Generator
- JSON Placeholder
- Lorem Picsum

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-11 | Initial release of the QuickChart documentation. |

<!-- /SECTION: changelog -->
---
node_id: "shields-io"
title: "Shields.io"
description: "Generate SVG status badges using the Shields.io static badge API with customizable labels, messages, colors, styles, and logos."
category: "devops-cloud-management"
subcategory: "developer-tools"
version: "1.0.0"
language: "en"
last_updated: "2026-11-18"
author: "Fusion Team"
tags:
  - shields-io
  - badges
  - svg
  - status
  - ci-cd
  - developer-tools
  - documentation
related_nodes:
  - quick-chart
  - barcode-generator
  - github
  - function
---

<!-- SECTION: overview -->
# Shields.io

> **Category:** DevOps & Cloud Management &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Generate customizable **SVG status badges** using the public **Shields.io** static badge API (`https://img.shields.io/static/v1`). Configure the badge label, message, colors, visual style, and optional icon logo to receive both a ready-to-embed SVG string (`badge_svg`) and the direct image URL (`badge_url`).

No API key or authentication is required.

### Key Features

- **Static Badge Generation:** Builds and fetches badges directly from `https://img.shields.io/static/v1`.
- **Dual Output Formats:** Returns both the raw SVG markup (`badge_svg`) for inline embedding and the resolved Shields.io URL (`badge_url`) for Markdown or HTML image links.
- **5 Built-in Styles:** Supports `flat`, `flat-square`, `plastic`, `for-the-badge`, and `social`.
- **Flexible Color Input:** Accepts named colors (such as `brightgreen`, `blue`, `red`) or hex values with or without a leading `#` (automatically stripped).
- **Custom Logos:** Supports Simple Icons slugs (such as `github`, `docker`, `npm`) and optional logo tinting via `logoColor`.

### Use Cases

- **CI/CD & Build Status:** Generate live build, test, or deployment status badges after a pipeline completes.
- **Release & Version Tracking:** Produce version badges automatically whenever a new release is tagged.
- **Code Quality & Metrics:** Turn test coverage, latency, or uptime figures from upstream nodes into visual badges for dashboards and reports.
- **Automated Documentation:** Embed generated badge URLs or SVGs into README files, wikis, status pages, or release notes.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `label` | `string` | ✅ Yes | — | Left-side text displayed on the badge (e.g., `build`, `version`, `coverage`). Must be non-empty. |
| `message` | `string` | ✅ Yes | — | Right-side text displayed on the badge (e.g., `passing`, `v1.2.0`, `95%`). Must be non-empty. |
| `color` | `string` | ❌ No | `brightgreen` | Background color of the right (message) section. Accepts Shields.io named colors or hex codes (leading `#` is stripped automatically). |
| `labelColor` | `string` | ❌ No | `""` | Background color of the left (label) section. Accepts named colors or hex codes (leading `#` is stripped automatically). When empty, Shields.io uses its default dark grey (`#555`). |
| `style` | `enum` | ❌ No | `flat` | Visual badge style: `flat`, `flat-square`, `plastic`, `for-the-badge`, or `social`. |
| `logo` | `string` | ❌ No | `""` | Optional icon displayed on the left side of the badge. Accepts any Simple Icons slug (e.g., `github`, `npm`, `docker`, `typescript`) or a custom base64-encoded icon string supported by Shields.io. |
| `logoColor` | `string` | ❌ No | `""` | Optional color for the badge logo. Accepts named colors or hex codes (leading `#` is stripped automatically). Applies when `logo` is set. |

### Supported Styles

| Style | Description |
|-------|-------------|
| `flat` | Clean flat badge with slightly rounded corners (default). |
| `flat-square` | Flat badge with sharp square corners. |
| `plastic` | Glossy badge with subtle vertical gradient and rounded corners. |
| `for-the-badge` | Taller, uppercase badge designed for prominent headers and READMEs. |
| `social` | Light-themed social counter badge style (similar to GitHub stars/forks). |

### Colors & Hex Normalization

`color`, `labelColor`, and `logoColor` accept either:

- **Named colors:** `brightgreen`, `green`, `yellowgreen`, `yellow`, `orange`, `red`, `blue`, `lightgrey`, `blueviolet`, `success`, `important`, `critical`, `informational`, `inactive`.
- **Hex colors:** 3-digit or 6-digit hex values such as `4c1`, `#4c1`, `007ec6`, or `#007ec6`.

Any leading `#` character is automatically removed before sending the request to Shields.io, and the normalized value is returned in the node output.

### Authentication

No authentication is required. The node queries the public Shields.io static badge endpoint (`https://img.shields.io/static/v1`).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Triggers badge generation when an event arrives. Upstream values can be mapped into any configuration parameter using expressions (e.g., `{{ input.status }}`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Generated badge metadata, direct image URL, and raw SVG markup. |
| `error` | `Error` | Emitted when `label` or `message` is missing, or when the Shields.io API request fails. |

### Output Fields (`success`)

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Always `true` when badge generation succeeds. |
| `label` | `string` | Label text sent to Shields.io. |
| `message` | `string` | Message text sent to Shields.io. |
| `color` | `string` | Normalized message background color (leading `#` removed). |
| `labelColor` | `string` | Normalized label background color (leading `#` removed, or `""` if omitted). |
| `style` | `string` | Badge style used (`flat`, `flat-square`, `plastic`, `for-the-badge`, or `social`). |
| `logo` | `string` | Logo slug or value sent to Shields.io (or `""` if omitted). |
| `logoColor` | `string` | Normalized logo color (leading `#` removed, or `""` if omitted). |
| `badge_url` | `string` | Resolved Shields.io static badge URL (`response.url`). |
| `badge_svg` | `string` | Raw SVG markup returned by Shields.io, ready for direct embedding. |
| `note` | `string` | Usage hint: `"Use badge_svg for embedding SVG directly or badge_url for image link"`. |

### Output Example

```json
{
  "success": true,
  "label": "build",
  "message": "passing",
  "color": "brightgreen",
  "labelColor": "1e293b",
  "style": "flat",
  "logo": "github",
  "logoColor": "ffffff",
  "badge_url": "https://img.shields.io/static/v1?label=build&message=passing&color=brightgreen&style=flat&labelColor=1e293b&logo=github&logoColor=ffffff",
  "badge_svg": "<svg xmlns=\"http://www.w3.org/2000/svg\" width=\"94\" height=\"20\" role=\"img\" aria-label=\"build: passing\">...</svg>",
  "note": "Use badge_svg for embedding SVG directly or badge_url for image link"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Build Status Badge

Generate a standard green `flat` badge with default colors:

```json
{
  "label": "build",
  "message": "passing"
}
```

Uses default `color: "brightgreen"` and `style: "flat"`.

### Styled Release Badge with Logo and Hex Colors

Generate a `for-the-badge` release badge with custom label/message colors and a GitHub logo:

```json
{
  "label": "release",
  "message": "v2.4.0",
  "color": "#2563eb",
  "labelColor": "#1e293b",
  "style": "for-the-badge",
  "logo": "github",
  "logoColor": "#ffffff"
}
```

The leading `#` prefixes are automatically stripped, resulting in `color: "2563eb"`, `labelColor: "1e293b"`, and `logoColor: "ffffff"` in the request and output.

### Dynamic Status Badge from Upstream Data

Use Fusion expressions to map upstream workflow data into badge fields:

```json
{
  "label": "coverage",
  "message": "{{ input.coveragePercent }}%",
  "color": "{{ input.badgeColor }}",
  "style": "flat-square"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate a status badge with Shields.io
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Shields.io Node:** Calls the Shields.io static badge API with the configured label, message, color, style, and optional logo, returning both `badge_url` and `badge_svg`.
3. **Log Node:** Displays the generated badge metadata, URL, and SVG output.

### Example Configuration

```json
{
  "label": "build",
  "message": "passing",
  "color": "brightgreen",
  "style": "flat",
  "logo": "github",
  "logoColor": "ffffff"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Label and message are required`
- **Cause:** Either `label` or `message` is empty or missing when the node executes.
- **Solution:** Provide non-empty string values for both `label` and `message` (or verify that upstream expressions resolve to non-empty strings).

#### `Shields.io badge generation failed: Shields.io API error: <status>`
- **Cause:** The Shields.io API returned a non-2xx HTTP response (such as `400`, `429`, or `5xx`).
- **Solution:** Check that your parameters are valid (especially `style` and custom `logo` values). If you receive `429` or `5xx`, wait briefly and retry or reduce request frequency.

#### `Shields.io badge generation failed: <network error>`
- **Cause:** The worker could not reach `https://img.shields.io/static/v1` due to DNS, firewall, or connectivity issues.
- **Solution:** Ensure outbound HTTPS access to `img.shields.io` is allowed from the workflow runner environment.

#### Logo does not appear on the badge
- **Cause:** The `logo` slug does not match a valid Simple Icons slug, or `logoColor` was set without specifying `logo`.
- **Solution:** Verify the icon slug on [Simple Icons](https://simpleicons.org/) (use lowercase slugs such as `github`, `nodedotjs`, `docker`) and ensure `logo` is populated when using `logoColor`.

#### Unexpected badge colors
- **Cause:** An unrecognized color name or malformed hex string was provided in `color`, `labelColor`, or `logoColor`.
- **Solution:** Use standard Shields.io named colors (e.g., `brightgreen`, `blue`, `red`) or valid 3- or 6-character hex codes (e.g., `007ec6` or `#007ec6`).

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [QuickChart](/docs/quick-chart) – Generate chart images and SVGs from Chart.js configurations
- [Barcode/QR Generator](/docs/barcode-generator) – Generate 1D barcodes and QR codes as Base64 PNGs
- [GitHub](/docs/github) – Retrieve repository, release, or workflow status data to feed into badges
- [Function](/docs/function) – Compute dynamic labels, messages, or status colors before generating badges

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-18 | Initial documentation: parameter reference, badge styles, output schema, troubleshooting, and example workflow |

<!-- /SECTION: changelog -->

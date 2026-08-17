---
node_id: "qr-server"
title: "QR Server"
description: "Generate QR codes using QR Server API for product packaging or business cards."
category: "utilities"
subcategory: "developer-tools"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - qr-code
  - qr-server
  - barcode
  - generator
  - image
  - api
  - packaging
  - business-cards
related_nodes:
  - barcode-generator
  - quick-chart
  - media-log
  - http-request
  - function
  - log
---

<!-- SECTION: header -->
# QR Server

> **Category:** Utilities | **Type:** Action Node

Generate high-quality, scannable QR codes from text, URLs, contact vCards, or custom payloads using the [QR Server](https://goqr.me/api/) API for product packaging, marketing flyers, receipts, and digital business cards.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **QR Server** node connects to the QR Server cloud API (`api.qrserver.com`) to generate customizable QR (Quick Response) codes on demand. Whether you need scannable web links for retail packaging, digital contact cards (vCards) for event badges, or high-resolution vector assets for print media, this node automates the generation and delivery process within your workflows.

The node takes your text or URL payload and produces either a direct image URL, a Base64-encoded data string, or vector SVG content ready for immediate display in the **Media Viewer** (`media-log`), integration into dynamic PDF documents, or emailing to customers.

### Key Features

- **Versatile Data Encoding:** Encodes standard URLs, plain text, WiFi credentials, vCard contact cards, email addresses, SMS prompts, and payment payloads.
- **Configurable Output Dimensions:** Define custom pixel dimensions (such as `200x200`, `350x350`, `500x500`, or `1000x1000`) for clear readability across screens and physical print.
- **Multiple File Formats:** Supports raster image formats (`png`, `gif`, `jpg`) for web/email and scalable vector formats (`svg`, `eps`) for professional high-resolution printing.
- **Adjustable Error Correction (ECC):** Select between 4 error correction levels (`L`, `M`, `Q`, `H`) to maintain scannability even if a portion of the printed QR code is obscured, scratched, or decorated with a logo.
- **Zero Configuration / No API Key:** Queries the public QR Server API without requiring API keys or authentication headers.
- **Base64 & Direct URL Output:** Returns both the direct hosted image URL and Base64 data URLs for seamless offline embedding or media rendering.

### Generation Flow

```text
Incoming Payload / Parameter (e.g. "nike.com")
                   ↓
   Build QR Server Request URL
   (data, size, format, errorCorrection)
                   ↓
        Call QR Server API
   (https://api.qrserver.com/v1/create-qr-code/)
                   ↓
         Format Response
   ┌───────────────┴───────────────┐
   ↓                               ↓
Raster (PNG/JPG/GIF)         Vector (SVG/EPS)
   ↓                               ↓
Base64 Data URL + Image URL   SVG Text + Direct Link
   └───────────────┬───────────────┘
                   ↓
      Success Output (Media Viewer / Log)
```

### Use Cases

- **Product Packaging & Labels:** Generate unique serial or batch QR codes linking to online user manuals, warranty registrations, or authenticity verifications.
- **Digital & Printed Business Cards:** Create vCard QR codes for conference badges, resumes, and email signatures.
- **E-Commerce & Digital Receipts:** Automatically generate order tracking and review links embedded in confirmation emails or PDF invoices.
- **Restaurant & Hospitality Menus:** Produce table-specific QR codes directing patrons to contactless digital menus and payment portals.
- **Event Ticketing & Check-In:** Generate unique ticket confirmation codes for rapid smartphone check-in scanners.
- **Smart Office & WiFi Access:** Generate instant WiFi connection codes (`WIFI:S:...`) for guest access cards in office lobbies.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | ✅ Yes | — | The text, URL, or raw data string to encode in the QR code (e.g., `nike.com`, `https://example.com`). |
| `size` | `string` | ❌ No | `200x200` | The output dimensions in `<width>x<height>` pixels (e.g., `200x200`, `300x300`, `500x500`). |
| `format` | `string` | ❌ No | `png` | The desired file format (`png`, `svg`, `eps`, `gif`, `jpg`). |
| `errorCorrection` | `string` | ❌ No | `M` | Error correction code (ECC) recovery level (`L`, `M`, `Q`, `H`). |

---

### Parameter Details

#### `data`
The primary payload to be encoded into the 2D QR matrix.
- **Type:** `string`
- **Required:** Yes
- **Supported Content:** URLs (`https://example.com`), plain text, email links (`mailto:user@example.com`), telephone URIs (`tel:+123456789`), WiFi strings (`WIFI:S:MySSID;T:WPA;P:MyPassword;;`), and vCard records.
- **Example:** `"nike.com"`, `"https://fusion.ai/docs"`

#### `size`
Specifies the width and height of the generated QR code image in pixels.
- **Type:** `string`
- **Default:** `200x200`
- **Syntax:** `[width]x[height]` (e.g., `150x150`, `200x200`, `400x400`, `1000x1000`)
- **Note:** Square dimensions are strongly recommended for standard QR code compliance.

#### `format`
Defines the output file format of the generated QR code graphic.
- **Type:** `string`
- **Default:** `png`
- **Available Options:**
  - `png` — High-compatibility raster format with lossless compression (recommended for web, apps, and emails).
  - `svg` — Scalable Vector Graphics format (ideal for responsive web designs and crisp screen display at any scale).
  - `eps` — Encapsulated PostScript vector format (ideal for commercial offset printing and packaging workflows).
  - `jpg` / `jpeg` — Standard raster format with opaque background.
  - `gif` — Compact raster format.

#### `errorCorrection`
Controls the Reed-Solomon error correction level (ECC) embedded in the QR code. Higher levels increase redundancy, allowing the QR code to be scanned even if part of it is damaged, dirty, or obscured by graphics/logos.
- **Type:** `string`
- **Default:** `M`
- **Supported Levels:**

| Level | Name | Recovery Capability | Recommended For |
|:-----:|:-----|:-------------------:|:----------------|
| **`L`** | Low | ~7% | Clean digital screens where maximum data density is required. |
| **`M`** | Medium | ~15% | Standard default; great balance between density and scan reliability. |
| **`Q`** | Quartile | ~25% | Printed documents, brochures, and retail shelf tags prone to slight wear. |
| **`H`** | High | ~30% | Industrial packaging, outdoor signage, or when overlaying custom brand logos. |

---

### API Endpoint

The node constructs and submits requests to the QR Server API endpoint:

```text
https://api.qrserver.com/v1/create-qr-code/?data={data}&size={size}&format={format}&ecc={errorCorrection}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming trigger or upstream workflow payload. Can supply dynamic `data` values via expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the QR code is successfully generated. Contains the image URL, Base64 data string, and metadata. |
| `error` | `Error` | Emitted if the API request fails, required parameters are missing, or network issues occur. |

---

### Output Data Structure

#### Raster Output (`png`, `jpg`, `gif`)

When a raster image format like `png` is generated, the `success` output payload provides:

```json
{
  "success": true,
  "data": "nike.com",
  "url": "https://api.qrserver.com/v1/create-qr-code/?data=nike.com&size=200x200&format=png&ecc=M",
  "qr_url": "https://api.qrserver.com/v1/create-qr-code/?data=nike.com&size=200x200&format=png&ecc=M",
  "image": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAADIAQAAAACFIImAAAAAzklEQVR42u3YQRKCMAwAQPIb/v9b60VjG8WWhkI73E2bA4eEkG2t447r...",
  "size": "200x200",
  "format": "png",
  "errorCorrection": "M",
  "mime_type": "image/png"
}
```

#### Vector Output (`svg`)

When `format` is configured to `svg`, the payload includes the raw SVG markup:

```json
{
  "success": true,
  "data": "nike.com",
  "url": "https://api.qrserver.com/v1/create-qr-code/?data=nike.com&size=200x200&format=svg&ecc=M",
  "qr_url": "https://api.qrserver.com/v1/create-qr-code/?data=nike.com&size=200x200&format=svg&ecc=M",
  "svg": "<svg xmlns=\"http://www.w3.org/2000/svg\" viewBox=\"0 0 200 200\">...</svg>",
  "size": "200x200",
  "format": "svg",
  "errorCorrection": "M",
  "mime_type": "image/svg+xml"
}
```

---

### Output Fields Reference

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates whether the QR code generation was successful. |
| `data` | `string` | The original string or URL encoded in the QR code. |
| `url` | `string` | Direct public URL to fetch the generated QR code image. |
| `qr_url` | `string` | Alias for the direct QR code image URL. |
| `image` | `string` | Base64-encoded Data URL (e.g. `data:image/png;base64,...`) suitable for inline HTML/CSS or direct viewing in the **Media Viewer**. |
| `svg` | `string` | Raw SVG XML string (available when format is `svg`). |
| `size` | `string` | Dimensions configured for the generated QR code (e.g., `200x200`). |
| `format` | `string` | Output file format (`png`, `svg`, `eps`, `gif`, `jpg`). |
| `errorCorrection` | `string` | Error correction level applied (`L`, `M`, `Q`, `H`). |
| `mime_type` | `string` | Corresponding MIME type of the generated asset (e.g. `image/png`, `image/svg+xml`). |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Website URL for Product Packaging

Generate a standard PNG QR code directing customers to a brand product page (`nike.com`).

**Parameter Configuration:**

```text
Data: nike.com
Size: 200x200
Format: png
ErrorCorrection: M
```

**Result:**
Generates a 200×200 pixel PNG QR code with 15% error correction capacity, optimal for printing on apparel tags, packaging boxes, or product user guides.

---

### Example 2: High-Resolution Print with High Error Correction

Generate a high-density QR code for outdoor packaging or business cards subject to physical wear.

**Parameter Configuration:**

```text
Data: https://example.com/warranty/register?sn=98234-AX
Size: 600x600
Format: png
ErrorCorrection: H
```

**Result:**
Generates a large 600×600 pixel image with maximum 30% error recovery, ensuring fast scan rates even if the label is scratched or smudged.

---

### Example 3: Scalable Vector Graphics (SVG) for Web Applications

Create a responsive vector QR code that scales smoothly across mobile devices and retina screens.

**Parameter Configuration:**

```text
Data: https://app.fusion.ai/invite/team-123
Size: 300x300
Format: svg
ErrorCorrection: Q
```

**Result:**
Returns clean `<svg>` markup that can be directly embedded into web pages or email newsletters without losing resolution.

---

### Example 4: vCard Contact QR Code for Business Cards

Encode complete contact information into a scannable business card format.

**Parameter Configuration:**

```text
Data: BEGIN:VCARD\nVERSION:3.0\nN:Doe;John\nFN:John Doe\nORG:Fusion AI\nTITLE:Lead Engineer\nTEL:+1-555-0199\nEMAIL:john@fusion.ai\nURL:https://fusion.ai\nEND:VCARD
Size: 350x350
Format: png
ErrorCorrection: M
```

**Result:**
When scanned by a smartphone camera, prompts the user to instantly save John Doe's contact information to their address book.

---

### Example 5: WiFi Network Auto-Connect

Generate a QR code allowing guests to connect to an office or event WiFi network automatically.

**Parameter Configuration:**

```text
Data: WIFI:S:FusionGuest;T:WPA;P:Welcome2026!;;
Size: 250x250
Format: png
ErrorCorrection: M
```

**Result:**
Scanning the QR code automatically connects supported mobile devices to the `FusionGuest` wireless network.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate QR codes using QR Server API for product packaging or business cards
```

### Common Workflow Patterns

- **Live QR Preview Pipeline:**
  `Manual Trigger` → `QR Server` (`data: "nike.com"`, `size: "200x200"`) → `Media Viewer` (`media-log`) + `Log` console.
- **Dynamic Order Packaging Generator:**
  `Order Webhook Trigger` → `Function` (Format tracking URL) → `QR Server` (`size: "300x300"`, `errorCorrection: "H"`) → `PDF Generator` (Embed QR on Packing Slip) → `AWS S3 Upload`.
- **Automated Event Badge Generator:**
  `New Attendee Webhook` → `Function` (Build vCard payload) → `QR Server` (`format: "png"`) → `Email Send` (Send printable badge with QR).
- **Contactless Restaurant Table Setup:**
  `Airtable / Database Read` (Table list) → `For Each` → `QR Server` (`format: "svg"`) → `Google Drive` (Save vector print assets).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Missing or Empty `data` Parameter
- **Symptom:** The node throws an error or returns a blank QR code.
- **Cause:** The `data` field was left empty or resolved to an `undefined` value from upstream node expressions.
- **Solution:** Verify that the `data` parameter contains a non-empty string or that upstream variables are properly populated before triggering the node.

#### Invalid Size Format
- **Symptom:** API returns an error or image does not render.
- **Cause:** The `size` parameter was formatted improperly (e.g. `200` instead of `200x200` or using spaces like `200 x 200`).
- **Solution:** Always format `size` as `<width>x<height>` (e.g. `200x200`, `400x400`).

#### Scanning Failures on Printed Packaging
- **Symptom:** Smartphone scanners struggle to read the printed QR code.
- **Cause:** The error correction level is too low for the print material, or the code contains too much data for a small pixel size.
- **Solution:** Increase `errorCorrection` to `Q` or `H`, and increase `size` (e.g., to `400x400` or `500x500`) to increase module dot size on print media.

#### Special Characters & Encoding
- **Symptom:** Scanned QR code displays garbled characters.
- **Cause:** Unescaped newline characters or special symbols in complex payloads (like vCards or JSON strings).
- **Solution:** Ensure standard line breaks (`\n`) and RFC-compliant vCard formatting are used in expressions.

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Data parameter is required` | `data` field was empty or null | Supply a valid URL or text payload in `data` |
| `Invalid size format` | Size string does not follow `WxH` format | Set size as `[width]x[height]` (e.g. `200x200`) |
| `HTTP 400 Bad Request` | Unsupported format or invalid ECC value | Verify `format` is one of `png`, `svg`, `eps`, `gif`, `jpg` and ECC is `L`, `M`, `Q`, or `H` |
| `Network error / Timeout` | Connectivity issue reaching `api.qrserver.com` | Check outbound internet access and firewall rules |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Barcode/QR Generator](../barcode-generator/en.md) — Generate local offline 1D and 2D barcodes using bwip-js
- [QuickChart](../quick-chart/en.md) — Render dynamic charts and graphs via API
- [Media Viewer / Media Log](../media-log/en.md) — Display rich media, images, and visual outputs directly on the canvas
- [HTTP Request](../http-request/en.md) — Send custom HTTP requests to external REST APIs
- [Function](../function/en.md) — Transform payloads and generate dynamic QR input strings
- [Log](../log/en.md) — Inspect workflow outputs in the debug console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial release with support for custom sizes, raster/vector formats, and configurable error correction |

<!-- /SECTION: changelog -->

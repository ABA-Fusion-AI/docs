---
node_id: "brandfetch"
title: "Brandfetch"
description: "Retrieve brand data including logos, colors, and fonts from Brandfetch."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-08-13"
author: "Fusion Team"
tags:
  - brandfetch
  - brand
  - logo
  - colors
  - fonts
  - asset-fetching
  - branding
  - peer-only
  - action
related_nodes:
  - http-request
  - function
  - if
---

<!-- SECTION: overview -->
# Brandfetch

> **Category:** Peer-only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Retrieve brand data including logos, color palettes, fonts, and company metadata from the **Brandfetch API**.

The **Brandfetch** node enables workflows to look up official corporate brand identity details by domain name or search query. It extracts high-resolution logos, vector icons, brand colors, typography details, company descriptions, social media profiles, and brand assets to streamline design automation, CRM enrichment, and customized portal onboarding.

### Key Capabilities

- **Get Full Brand Profile (`getBrand`):** Fetch comprehensive brand identity records (logos, colors, fonts, company summary, social links, and cover images) for any company domain.
- **Search Brands (`searchBrands`):** Search the Brandfetch database by company name query or domain keyword.
- **Get Logos (`getLogos`):** Retrieve official brand logo variants, icons, and SVG/PNG image formats.
- **Get Color Palettes (`getColors`):** Extract brand color hex codes, RGB values, and color roles (`primary`, `secondary`, `accent`).
- **Get Brand Typography (`getFonts`):** Extract official font families, font weights, and typography assets.

### Use Cases

- **Automated Design & Asset Generation:** Pull partner logos and brand colors into automated email templates, PDF generators, or graphic design tools.
- **CRM & Account Profile Enrichment:** Enrich company accounts with official brand logos, industry descriptions, and social media handles.
- **SaaS Customer Onboarding:** Automatically theme client portals with their official logos and brand colors upon registration.
- **Brand Consistency & Asset Auditing:** Monitor and retrieve current brand guidelines and media assets automatically.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | `"getBrand"` | The Brandfetch API operation to execute (see operations table below). |
| `apiKey` | `string` | ❌ No | — | Brandfetch API Bearer Key / Client Token. Supports expressions. |
| `domain` | `string` | ❌ No | — | Target company domain name (e.g., `"nike.com"`). Required for domain-based operations (`getBrand`, `getLogos`, `getColors`, `getFonts`). Supports expressions. |
| `query` | `string` | ❌ No | — | Search query string (e.g., `"Nike"` or `"Apple"`). Used when searching brands with `searchBrands`. Supports expressions. |

### Operations (`operation`)

| Operation | Value | Description |
|-----------|-------|-------------|
| **Get Brand** | `getBrand` | Retrieves complete brand information including logos, colors, fonts, images, description, and social links for a given domain. |
| **Search Brands** | `searchBrands` | Searches the Brandfetch database by brand name query or domain keyword. |
| **Get Logos** | `getLogos` | Retrieves logo assets, icons, and SVG/PNG image variants for a domain. |
| **Get Colors** | `getColors` | Retrieves the official color palette, hex codes, and color types for a domain. |
| **Get Fonts** | `getFonts` | Retrieves official font families, font weights, and typography assets for a domain. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` or `string` | Workflow input data or trigger payload containing target domain or query parameters. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Returned when brand data or assets are successfully retrieved from Brandfetch. |
| `error` | `object` | Returned if the API request fails, domain is invalid, API key is missing/unauthorized, or rate limit is reached. |

### Output Data Fields (`getBrand`)

| Field | Type | Description |
|-------|------|-------------|
| `name` | `string` | Official brand or company name. |
| `domain` | `string` | Verified company domain name. |
| `claimed` | `boolean` | Indicates whether the brand profile has been claimed on Brandfetch. |
| `description` | `string` | Short brand description or company summary. |
| `longDescription` | `string` | Detailed extended company overview, when available. |
| `links` | `array` | Social media profiles and official company URLs (e.g., Twitter, Instagram, LinkedIn). |
| `logos` | `array` | List of logo objects including SVG/PNG URLs, dimensions, theme (light/dark), and format types. |
| `colors` | `array` | List of brand color objects including hex code, color type (`primary`, `secondary`, `accent`), and brightness score. |
| `fonts` | `array` | List of font family objects including name, type (`title`, `body`), and font origin. |
| `images` | `array` | Associated brand banners, cover images, or promotional graphics. |

---

### Configuration & Output Examples

#### Example 1: Get Complete Brand Profile (`getBrand`)

**Configuration**
```json
{
  "operation": "getBrand",
  "domain": "nike.com"
}
```

**Output (`success`)**
```json
{
  "name": "Nike",
  "domain": "nike.com",
  "claimed": true,
  "description": "Nike delivers innovative products, experiences and services to inspire and enable athletes.",
  "links": [
    {
      "name": "twitter",
      "url": "https://twitter.com/Nike"
    },
    {
      "name": "instagram",
      "url": "https://instagram.com/nike"
    }
  ],
  "logos": [
    {
      "theme": "dark",
      "type": "symbol",
      "formats": [
        {
          "src": "https://asset.brandfetch.io/id_nike/symbol_dark.svg",
          "format": "svg"
        },
        {
          "src": "https://asset.brandfetch.io/id_nike/symbol_dark.png",
          "format": "png",
          "width": 512,
          "height": 512
        }
      ]
    }
  ],
  "colors": [
    {
      "hex": "#111111",
      "type": "primary",
      "brightness": 17
    },
    {
      "hex": "#FFFFFF",
      "type": "secondary",
      "brightness": 255
    }
  ],
  "fonts": [
    {
      "name": "Futura Extra Bold",
      "type": "title"
    }
  ]
}
```

---

#### Example 2: Search Brands (`searchBrands`)

**Configuration**
```json
{
  "operation": "searchBrands",
  "query": "Nike"
}
```

**Output (`success`)**
```json
[
  {
    "name": "Nike",
    "domain": "nike.com",
    "claimed": true,
    "icon": "https://asset.brandfetch.io/id_nike/icon.png",
    "brandId": "id_nike"
  }
]
```

---

#### Example 3: Get Brand Logos (`getLogos`)

**Configuration**
```json
{
  "operation": "getLogos",
  "domain": "nike.com"
}
```

**Output (`success`)**
```json
[
  {
    "theme": "dark",
    "type": "logo",
    "formats": [
      {
        "src": "https://asset.brandfetch.io/id_nike/logo_dark.svg",
        "format": "svg"
      }
    ]
  }
]
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve Brand Assets with Brandfetch
```

### Sample Workflow: Manual Trigger → Brandfetch → Log Result

```json
{
  "nodes": [
    {
      "id": "trigger",
      "type": "manual-trigger"
    },
    {
      "id": "brandfetch",
      "type": "brandfetch",
      "config": {
        "operation": "getBrand",
        "domain": "nike.com"
      }
    },
    {
      "id": "log-result",
      "type": "log"
    }
  ]
}
```

### Common Design Patterns

- **Webhook / Form → Brandfetch → Storage:** Automatically fetch brand logos when a client submits their website domain and save vector assets to cloud storage.
- **Brandfetch (`getColors`) → Function → Dynamic CSS:** Retrieve official brand color hex codes to dynamically style customer portals or automated email designs.
- **Search Brands → User Selection → Get Brand:** Build brand search autocomplete and profile fetching into onboarding workflows.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Domain is required for getBrand / getLogos / getColors / getFonts`

**Cause:** The selected operation requires a `domain` parameter, but none was provided.

**Solution:** Provide a valid domain name (e.g., `"nike.com"`) in the `domain` field or via expression (`{{input.domain}}`).

---

#### `Unauthorized / Invalid API Key`

**Cause:** Brandfetch rejected the request because the API key is missing or invalid.

**Solution:** Pass a valid Brandfetch API key in `apiKey` or check your API key status on Brandfetch.

---

#### `Brand not found (404)`

**Cause:** Brandfetch does not currently have indexed brand data for the specified domain.

**Solution:** Verify the domain spelling (use `"nike.com"` instead of full URLs like `"https://www.nike.com/us"`). If searching by company name, use `searchBrands` first.

---

#### `Rate limit exceeded (429)`

**Cause:** API request volume has exceeded your Brandfetch plan limits.

**Solution:** Implement workflow caching or upgrade your Brandfetch API subscription.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](./http-request.md) – Perform custom API requests to external brand services
- [Function](./function.md) – Transform and format Brandfetch logo URLs and color schemes
- [If](./if.md) – Branch workflow logic based on brand lookup results

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-13 | Complete documentation release for Brandfetch node |

<!-- /SECTION: changelog -->

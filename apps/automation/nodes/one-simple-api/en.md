---
node_id: "one-simple-api"
title: "One Simple API"
description: "Web utility API: get a page's social/meta info, generate a PDF or a screenshot of a web page, and get currency exchange rates."
category: "peer-only"
subcategory: "Web Utilities"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - onesimpleapi
  - screenshot
  - pdf
  - exchange-rate
  - web-scraping
  - utilities
related_nodes:
  - html-to-pdf
  - http-request
  - log
---

<!-- SECTION: overview -->
# One Simple API

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Use **One Simple API** (`https://onesimpleapi.com`) to run common web utilities from a Fusion workflow. The node supports four operations:
- **`socialInfo`:** Get the title, description, image and other meta / Open Graph data of a web page.
- **`pdf`:** Generate a PDF of a web page.
- **`screenshot`:** Take a screenshot of a web page, with an optional custom size.
- **`exchangeRate`:** Get the exchange rate between two currencies.

### Use Cases

- **Link previews:** Build rich previews (title, image, description) for links shared in a channel or a CRM.
- **Archiving:** Save web pages as PDF or images for compliance or reporting.
- **Visual monitoring:** Take regular screenshots of a website to track changes.
- **Finance automations:** Convert amounts between currencies in invoices or reports.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `socialInfo` | `socialInfo`, `pdf`, `screenshot` or `exchangeRate`. |
| `apiToken` | `string` | Yes | — | One Simple API token. Supports expressions. |
| `url` | `string` | For `socialInfo`, `pdf`, `screenshot` | — | Web page to process (e.g. `https://example.com`). |
| `fromCurrency` | `string` | For `exchangeRate` | — | Source currency code (e.g. `USD`). |
| `toCurrency` | `string` | For `exchangeRate` | — | Target currency code (e.g. `EUR`). |
| `width` | `string` | No | `1280` | Screenshot viewport width, in pixels. |
| `height` | `string` | No | `800` | Screenshot viewport height, in pixels. |

### Operations

| Operation | Endpoint | Required Fields |
|-----------|----------|-----------------|
| `socialInfo` | `GET /api/page_info` | `apiToken`, `url` |
| `pdf` | `GET /api/pdf` | `apiToken`, `url` |
| `screenshot` | `GET /api/screenshot` | `apiToken`, `url` (`width` / `height` optional) |
| `exchangeRate` | `GET /api/exchange_rate` | `apiToken`, `fromCurrency`, `toCurrency` |

> For `screenshot`, the size is sent as `screen=custom:WIDTHxHEIGHT`. If only one of `width` / `height` is set, the other uses its default.

### Authentication & API Token

1. Create an account on **One Simple API** (`https://onesimpleapi.com`).
2. Open your **dashboard / API token** page and copy the token.
3. Paste it into the `apiToken` field (or store it in Fusion Secrets).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream values in `url` or the currencies through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | JSON response of One Simple API. If the response is not JSON, it is returned as `{ "result": "<text>" }`. |
| `error` | `Error` | Emitted on missing fields or API errors. |

### Output Examples

#### `socialInfo`

```json
{
  "url": "https://example.com",
  "title": "Example Domain",
  "description": "This domain is for use in illustrative examples in documents.",
  "image": null
}
```

#### `screenshot` / `pdf`

```json
{
  "url": "https://onesimpleapi.com/files/screenshots/abc123.png"
}
```

#### `exchangeRate`

```json
{
  "from_currency": "USD",
  "to_currency": "EUR",
  "exchange_rate": 0.92
}
```

> The exact fields depend on One Simple API; the node returns its response as-is.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Get web page info with One Simple API
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **One Simple API Node:** Calls the selected operation.
3. **Log Node:** Displays the result.

### Example Configurations

#### Screenshot

```json
{
  "apiToken": "{{ secrets.ONESIMPLEAPI_TOKEN }}",
  "operation": "screenshot",
  "url": "https://fusion-ai.com",
  "width": "1440",
  "height": "900"
}
```

#### Exchange rate

```json
{
  "apiToken": "{{ secrets.ONESIMPLEAPI_TOKEN }}",
  "operation": "exchangeRate",
  "fromCurrency": "USD",
  "toCurrency": "MAD"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store your One Simple API `apiToken` in Fusion's **Secrets** vault. Do not commit hardcoded tokens into version control.

- The token is sent as a URL query parameter: avoid logging full request URLs.
- Screenshots and PDFs of private pages may expose sensitive content; only process URLs you are allowed to capture.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `apiToken is required`
- **Cause:** `apiToken` is empty.
- **Solution:** Provide your One Simple API token.

#### `url is required for socialInfo` / `pdf` / `screenshot`
- **Cause:** `url` is empty for an operation that needs it.
- **Solution:** Fill in `url` with a full address, including `https://`.

#### `fromCurrency and toCurrency are required`
- **Cause:** One of the currency codes is missing for `exchangeRate`.
- **Solution:** Use ISO codes such as `USD`, `EUR`, `MAD`.

#### `One Simple API Error: 401` / `403`
- **Cause:** Invalid token, or the operation is not included in your plan.
- **Solution:** Check the token and your plan on the One Simple API dashboard.

#### `One Simple API Error: 429`
- **Cause:** Monthly quota or rate limit reached.
- **Solution:** Wait for the quota reset or upgrade your plan.

#### The screenshot size is not applied
- **Cause:** `width` / `height` must be numbers in pixels.
- **Solution:** Use values like `1280` and `800`, without `px`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Full documentation: operations, parameters, authentication, output examples, troubleshooting |
| 1.0.0 | 2026-08-04 | Initial documentation |

<!-- /SECTION: changelog -->

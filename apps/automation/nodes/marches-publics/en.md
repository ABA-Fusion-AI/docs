---
node_id: "marches-publics"
title: "Moroccan Tender Scraper"
description: "Scrape tender data from marchespublics.gov.ma — list tenders by type and date range, or fetch full details for a specific tender."
category: "utilities"
subcategory: "web-scraping"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - morocco
  - tenders
  - procurement
  - scraping
  - marchespublics
related_nodes:
  - http-request
  - function
  - log
  - filter-array
---

<!-- SECTION: header -->
# Moroccan Tender Scraper

> **Category:** Utilities | **Type:** Action Node

Scrape tender data from the official Moroccan public procurement portal [marchespublics.gov.ma](https://www.marchespublics.gov.ma). Supports listing tenders with pagination and date filters, or fetching complete details for a single tender.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Moroccan Tender Scraper** node retrieves public procurement data from Morocco's official tender portal, **marchespublics.gov.ma**, operated by the Direction des Marchés Publics (DMP).

The node supports two distinct **actions**:

- **`AO` (Appel d'Offres — list):** Retrieve paginated lists of tender notices, optionally filtered by publication date or deposit deadline.
- **`detail`:** Fetch the complete details of a specific tender by its URL.

### Key Features

- **Official Source:** Data pulled directly from marchespublics.gov.ma
- **Two Modes:** List mode for discovery, detail mode for full procurement data
- **Pagination Control:** Configure page number, items per page, and maximum pages
- **Date Filtering:** Filter by publication date range or deposit/submission deadline
- **Resilience:** Configurable retry count and delay between requests
- **No Authentication Required:** Public data, no API key needed

### Use Cases

- Monitor new tenders published daily and send notifications
- Archive tender listings in a database for historical analysis
- Feed procurement data into a business intelligence dashboard
- Alert on tenders matching specific sectors or keywords (via Function node)
- Automate competitive intelligence for suppliers and consultants

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

The node behaviour depends on the selected **Action**.

---

### Action: `AO` — List Tenders

Retrieves a paginated list of tender notices from marchespublics.gov.ma.

#### Parameters

| Parameter        | Type     | Required | Default | Description |
|------------------|----------|----------|---------|-------------|
| `action`         | `enum`   | Yes      | `AO`    | Set to `AO` to list tenders. |
| `page`           | `number` | No       | `1`     | Starting page number. |
| `maxPages`       | `number` | No       | `5`     | Maximum number of pages to scrape. |
| `perPage`        | `number` | No       | `20`    | Number of results per page. |
| `maxRetries`     | `number` | No       | `3`     | Number of retry attempts on network failure. |
| `delayMs`        | `number` | No       | `2000`  | Delay in milliseconds between page requests (rate limiting). |
| `pubDateStart`   | `string` | No       | —       | Filter tenders published on or after this date. Format: `DD/MM/YYYY`. |
| `pubDateEnd`     | `string` | No       | —       | Filter tenders published on or before this date. Format: `DD/MM/YYYY`. |
| `depotDateStart` | `string` | No       | —       | Filter tenders with a submission deadline on or after this date. Format: `DD/MM/YYYY`. |
| `depotDateEnd`   | `string` | No       | —       | Filter tenders with a submission deadline on or before this date. Format: `DD/MM/YYYY`. |

#### Date Format

All date parameters use the `DD/MM/YYYY` format:

```text
07/08/2026  → August 7, 2026
01/01/2026  → January 1, 2026
```

Leave date fields empty to retrieve all tenders regardless of date.

---

### Action: `detail` — Fetch Tender Details

Fetches the full detail page for a specific tender.

#### Parameters

| Parameter | Type     | Required | Default | Description |
|-----------|----------|----------|---------|-------------|
| `action`  | `enum`   | Yes      | —       | Set to `detail` to fetch a single tender. |
| `url`     | `string` | No       | —       | The full URL of the tender detail page on marchespublics.gov.ma. |

Typically the `url` value comes from a tender item returned by the `AO` action.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input   | Type  | Description |
|---------|-------|-------------|
| `input` | `any` | Incoming workflow data passed from the preceding node. |

### Outputs

| Output    | Type    | Description |
|-----------|---------|-------------|
| `success` | `array` or `object` | Scraped tender data. An array of tenders for `AO`, or a single tender object for `detail`. |
| `error`   | `Error` | Emitted when the request fails, the portal is unreachable, or scraping encounters an error. |

---

### AO Output Structure

The `AO` action returns an array of tender listings:

```json
[
  {
    "title": "Acquisition de matériel informatique",
    "reference": "AO-2026-001234",
    "buyer": "Ministère de la Santé",
    "pubDate": "07/08/2026",
    "depotDate": "25/08/2026",
    "type": "Appel d'offres ouvert",
    "url": "https://www.marchespublics.gov.ma/index.php?page=entreprise.EntrepriseAdvancedSearch&..."
  }
]
```

| Field       | Type     | Description |
|-------------|----------|-------------|
| `title`     | `string` | Title or object of the tender |
| `reference` | `string` | Unique tender reference number |
| `buyer`     | `string` | Name of the contracting authority |
| `pubDate`   | `string` | Publication date of the tender notice |
| `depotDate` | `string` | Submission/deposit deadline |
| `type`      | `string` | Type of procurement procedure |
| `url`       | `string` | Direct URL to the tender detail page |

---

### Detail Output Structure

The `detail` action returns a single object with full tender information:

```json
{
  "title": "Acquisition de matériel informatique",
  "reference": "AO-2026-001234",
  "buyer": "Ministère de la Santé",
  "pubDate": "07/08/2026",
  "depotDate": "25/08/2026",
  "type": "Appel d'offres ouvert",
  "estimatedBudget": "500 000,00 MAD",
  "description": "Fourniture et livraison de matériel informatique...",
  "documents": [
    {
      "name": "CPS - Cahier des Prescriptions Spéciales",
      "url": "https://www.marchespublics.gov.ma/..."
    }
  ],
  "contact": {
    "name": "Direction des Approvisionnements",
    "address": "Rabat, Maroc",
    "phone": "+212 5XX XX XX XX"
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Fetch the First 5 Pages of Tenders

List the latest 100 tenders (5 pages × 20 per page).

**Configuration:**

```text
Action:   AO
Page:     1
MaxPages: 5
PerPage:  20
DelayMs:  2000
```

**Output:** An array of up to 100 tender objects.

---

### Example 2: Filter Tenders by Publication Date

Retrieve tenders published in August 2026.

**Configuration:**

```text
Action:       AO
PubDateStart: 01/08/2026
PubDateEnd:   31/08/2026
MaxPages:     10
PerPage:      20
```

---

### Example 3: Filter by Submission Deadline

Find tenders whose submission deadline falls within the next two weeks.

**Configuration:**

```text
Action:          AO
DepotDateStart:  10/08/2026
DepotDateEnd:    24/08/2026
```

---

### Example 4: Fetch Full Details for a Specific Tender

Use the URL from an `AO` listing to retrieve complete tender details.

**Configuration:**

```text
Action: detail
Url:    https://www.marchespublics.gov.ma/index.php?page=entreprise.EntrepriseAdvancedSearch&...
```

---

### Example 5: Daily Tender Monitoring with Notifications

Automatically check for new tenders each morning and send an email summary.

**Workflow pattern:**

```text
Cron Trigger (every day at 08:00)
  → Moroccan Tender Scraper (action: AO, pubDateStart: today, pubDateEnd: today)
  → Function (format tender list as HTML table)
  → Email Send
```

---

### Example 6: Enrich Listings with Full Details

Combine the list and detail actions to build a complete tender dataset.

**Workflow pattern:**

```text
Moroccan Tender Scraper (action: AO)
  → For Each (item of success output)
      → Moroccan Tender Scraper (action: detail, url: {{ $item.url }})
      → Database Insert
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Scrape Moroccan public tenders from marchespublics.gov.ma
```

### Common Patterns

- **Daily Digest:** Cron Trigger → Moroccan Tender Scraper (AO) → Email Send
- **Keyword Alerting:** Moroccan Tender Scraper → Filter Array (by keyword) → Teams / SMS
- **Database Archiving:** Moroccan Tender Scraper → For Each → Database Insert
- **Detail Enrichment:** AO listing → For Each → Detail fetch → Merge → Store
- **BI Dashboard:** Cron → Scraper → HTTP Request (POST to dashboard API)

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Empty results

**Cause:** The date filter range contains no published tenders, or the portal returned an empty page.

**Solution:** Widen the date range or remove the date filters and verify results manually on marchespublics.gov.ma.

#### Network timeout / scraping fails intermittently

**Cause:** The portal may throttle requests or be temporarily slow.

**Solution:**
- Increase `delayMs` (e.g., to `3000` or `5000`) to slow down requests.
- Increase `maxRetries` (e.g., to `5`) to allow more retry attempts.
- Avoid running too many pages in a single execution.

#### Only partial results returned

**Cause:** `maxPages` was reached before all matching tenders were retrieved.

**Solution:** Increase `maxPages` and reduce `perPage` to avoid hitting portal limits per request.

#### Detail action returns incomplete data

**Cause:** The tender detail page structure may vary per tender type, or the URL may be stale.

**Solution:** Verify the URL is a valid and accessible marchespublics.gov.ma page. Use the URL returned directly from an `AO` result.

#### Rate limiting / IP block

**Cause:** Excessive requests in a short period may trigger portal-side rate limiting.

**Solution:**
- Set `delayMs` to at least `2000` ms (default).
- Avoid running the scraper more than once per minute.
- Use a smaller `perPage` and `maxPages` for high-frequency schedules.

### Error Reference

| Error | Cause | Solution |
|-------|-------|----------|
| Network timeout | Portal unreachable or slow | Increase `delayMs` and `maxRetries` |
| Empty response | No tenders match the filter | Widen or remove date filters |
| Invalid URL | `detail` URL is malformed | Use a URL from an `AO` result |
| Parse error | Portal HTML structure changed | Report to the Fusion Team for a node update |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Filter Array](../filter-array/en.md) — Filter tenders by keyword, sector, or date
- [For Each](../for-each/en.md) — Iterate over tender listings to fetch details
- [Function](../function/en.md) — Transform or format tender data
- [HTTP Request](../http-request/en.md) — Make custom requests to marchespublics.gov.ma
- [Log](../log/en.md) — Inspect raw scraper output

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date       | Changes         |
|---------|------------|-----------------|
| 1.0.0   | 2026-08-10 | Initial release |

<!-- /SECTION: changelog -->

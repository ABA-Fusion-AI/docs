---
node_id: "emploima"
title: "Emploi.ma Scraper"
description: "Scrapes and normalizes job listings from Emploi.ma."
category: "web-search"
subcategory: "Job Boards"
version: "1.0.0"
language: "en"
last_updated: "2026-08-07"
author: "Fusion Team"
tags:
  - morocco
  - jobs
  - scraping
  - emploi
  - recruitment
related_nodes:
  - rekrute-scraper
  - linkedin-job-scraper
  - function
  - http-request
---

<!-- SECTION: overview -->
# Emploi.ma Scraper

> **Category:** Web Search &amp; Information &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Scrape and normalize job listings from [Emploi.ma](https://www.emploi.ma), Morocco's leading job board. The node crawls listing pages, visits each job detail page in parallel, and returns structured job records — including title, company, city, education level, contract type, experience, description, and a direct URL to the offer.

### Use Cases

- **Job Market Monitoring:** Automatically collect and track new job postings on Emploi.ma for market intelligence or competitive analysis.
- **Candidate Matching:** Scrape listings and pass them to an AI node to match against candidate profiles.
- **Recruitment Dashboards:** Feed scraped data into a database or spreadsheet for visualization and reporting.
- **Alert Systems:** Run on a schedule and notify a team via Slack or email when new listings matching a keyword appear.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `maxPages` | `number` | No | `5` | Number of Emploi.ma listing pages to scrape. Each page contains multiple job cards. |
| `concurrency` | `number` | No | `5` | Number of job detail pages fetched in parallel. Higher values speed up scraping but increase the chance of being rate-limited. |

### Tuning Recommendations

| Scenario | `maxPages` | `concurrency` |
|----------|-----------|--------------|
| Quick test | `1` | `2` |
| Standard run | `5` | `5` *(default)* |
| Deep collection | `20` | `3` |

> **Note:** Emploi.ma may rate-limit or block automated requests. Start with low values and increase gradually. If the site blocks the request, the node throws: `No Emploi.ma job listings found. The website may have changed or blocked the request.`

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This node does not require an upstream data input. It triggers the scraping process as soon as it is executed. Connect a **Manual Trigger** or **Cron** node to start it.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `array` | Array of normalized job objects scraped from Emploi.ma. |
| `error` | `Error` | Emitted if no listings are found or the site is unreachable. |

### Output Schema (`success`)

The node returns an array of job objects. Each object has the following fields:

```json
[
  {
    "site": "Emploi.ma",
    "intitulé_poste": "Développeur Full Stack",
    "entreprise": "Tech Corp Maroc",
    "ville": "Casablanca",
    "niveau_études": "BAC+5",
    "type_contrat": "CDI",
    "expérience": "3 à 5 ans d'expérience",
    "publication_date": "il y a 2 jours",
    "description": "Nous recherchons un développeur Full Stack maîtrisant React et Node.js...",
    "lien_offre": "https://www.emploi.ma/offre-emploi-maroc/developpeur-full-stack-12345"
  }
]
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `site` | `string` | Always `"Emploi.ma"`. |
| `intitulé_poste` | `string` | Job title extracted from the listing page `<h1>`. |
| `entreprise` | `string` | Company name. |
| `ville` | `string` | City where the job is located. |
| `niveau_études` | `string` | Detected education level: `BAC`, `BAC+2`, `BAC+3`, `BAC+4`, `BAC+5`, `DOCTORAT`, or `Non spécifié`. |
| `type_contrat` | `string` | Detected contract type: `CDI`, `CDD`, `STAGE`, `INTÉRIM`, `FREELANCE`, `ANAPEC`, `ALTERNANCE`, or `Non spécifié`. |
| `expérience` | `string` | Required experience extracted from the page. |
| `publication_date` | `string` | Publication date as displayed on the listing page. |
| `description` | `string` | First 500 characters of the job description (truncated with `...` if longer). |
| `lien_offre` | `string` | Direct URL to the full job listing on Emploi.ma. |

### Normalization Logic

The node automatically detects `niveau_études` and `type_contrat` from the full page text using keyword patterns:

| Detected value | Triggered by |
|----------------|-------------|
| `BAC+5` | Keywords: `bac+5`, `master`, `ingénieur` |
| `BAC+4` | Keywords: `bac+4`, `licence` |
| `BAC+3` | Keyword: `bac+3` |
| `BAC+2` | Keyword: `bac+2` |
| `BAC` | Keyword: `bac` |
| `DOCTORAT` | Keywords: `doctorat`, `phd` |
| `CDI` | Keyword: `cdi` |
| `CDD` | Keyword: `cdd` |
| `STAGE` | Keyword: `stage` |
| `INTÉRIM` | Keywords: `intérim`, `interim` |
| `FREELANCE` | Keyword: `freelance` |
| `ANAPEC` | Keyword: `anapec` |
| `ALTERNANCE` | Keyword: `alternance` |

If no pattern matches, the field is set to `"Non spécifié"`.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Collect Emploi.ma Listings
```

### How it flows

1. **Manual Trigger** (or **Cron**): Starts the workflow.
2. **Emploi.ma Scraper:** Scrapes up to `maxPages` listing pages and fetches job details with the configured `concurrency`. Returns a normalized array of job records.
3. **Log Node** (or downstream processing): Receives the full array for display, storage, or further processing.

### Common Patterns

- **Scheduled Collection:** Use a Cron node (e.g., `0 8 * * *`) to collect listings every morning and push them to a Google Sheet or database.
- **AI Filtering:** Pass the results array to an AI Chat node to filter only listings matching a specific profile or technology stack.
- **Slack Alerts:** Send new listings to a Slack channel by combining the scraper with a For Each node and a Slack node.
- **Deduplication:** Store seen `lien_offre` URLs in a cache or database and filter out already-processed listings before sending notifications.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `No Emploi.ma job listings found`
- **Cause:** The site blocked the request, the page structure changed, or the network request timed out (10-second timeout per page).
- **Solution:** Reduce `concurrency` and `maxPages`, then retry. If the issue persists, the site may have updated its HTML structure.

#### Empty fields (`"Non spécifié"`) in output
- **Cause:** The job listing page did not contain the expected keywords or HTML selectors for that field.
- **Solution:** Use the `lien_offre` URL to inspect the raw page. Some listings intentionally omit certain details.

#### Slow execution
- **Cause:** `maxPages` is high and `concurrency` is low, causing many sequential requests.
- **Solution:** Increase `concurrency` (e.g., `10`) to fetch more detail pages in parallel. Be aware this may trigger rate limiting.

#### Truncated description
- **Cause:** Descriptions longer than 500 characters are automatically truncated by the node.
- **Solution:** Use the `lien_offre` field to fetch the full page content via an HTTP Request node if you need the complete description.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Rekrute Scraper](./rekrute-scraper.md) – Scrape job listings from Rekrute.com
- [LinkedIn Job Scraper](./linkedin-job-scraper.md) – Collect job listings from LinkedIn
- [HTTP Request](./http-request.md) – Fetch the full job page content using the `lien_offre` URL
- [Function](./function.md) – Post-process or filter the scraped job array

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-07 | Initial documentation |

<!-- /SECTION: changelog -->

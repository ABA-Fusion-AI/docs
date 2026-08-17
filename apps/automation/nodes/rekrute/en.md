---
node_id: "rekrute-scraper"
title: "Rekrute Scraper"
description: "Scrapes job listings from Rekrute.com using fast HTML parsing."
category: "web-search"
subcategory: "Job Boards"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - rekrute
  - morocco
  - jobs
  - scraping
  - recruitment
  - web-search
  - emploi
related_nodes:
  - emploima
  - rekrute-job-parser
  - linkedin-job-scraper
  - http-request
  - function
  - log
---

<!-- SECTION: header -->
# Rekrute Scraper

> **Category:** Web Search & Information | **Type:** Action Node

Scrape and extract structured job listings from [ReKrute.com](https://www.rekrute.com), Morocco's premier executive and professional job portal, using fast concurrent HTML parsing.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Rekrute Scraper** node automates the collection and normalization of employment listings from ReKrute.com. By crawling listing directories and parsing individual offer pages in parallel, the node extracts rich job metadata—including job titles, hiring companies, locations, required study levels, contract types, experience brackets, publication dates, and comprehensive job descriptions.

This node is designed for recruitment platforms, HR aggregators, AI candidate-matching pipelines, and labor market intelligence dashboards seeking automated access to Moroccan job market opportunities.

### Key Features

- **Rich Job Detail Extraction:** Parses job title, company name, city/region, contract type (CDI, CDD, Freelance, Stage), experience requirements, education level, and full job descriptions.
- **Configurable Crawl Depth (`maxPages`):** Control the number of listing pages to traverse per execution.
- **Concurrent Detail Fetching (`concurrency`):** Retrieve multiple detailed job offer pages simultaneously to maximize throughput without exceeding server tolerance.
- **Standardized Structured Output:** Delivers normalized JSON arrays ready for direct insertion into SQL/NoSQL databases, Google Sheets, or vector stores.
- **Zero API Key Requirement:** Directly parses public HTML endpoints without mandatory credentials or paid subscriptions.

### Scraping Flow

```text
Manual / Scheduled Trigger
             ↓
Fetch Rekrute.com Listing Pages (1 to maxPages)
             ↓
Extract Job Offer Links from Listing Cards
             ↓
Fetch Detail Pages in Parallel (concurrency limit)
             ↓
Parse & Normalize Job Attributes
(title, company, city, contract, education, experience, description, URL)
             ↓
Emit Structured Array Output (Log / Database / AI Matcher)
```

### Use Cases

- **Moroccan Tech & Executive Job Monitoring:** Automatically collect new IT, banking, finance, and engineering openings across Casablanca, Rabat, Tanger, and remote positions.
- **Candidate-Job AI Matching:** Scrape live job descriptions and feed them into an AI Agent node to match against candidate CVs.
- **Recruitment Market Intelligence:** Track which multinational corporations and Moroccan enterprises are actively hiring, monitor popular tech stacks, and analyze salary/contract trends.
- **Automated Job Alerts:** Run scheduled scrapers (via Cron Trigger) to notify candidate communities or recruitment teams via Slack, Telegram, or Email when matching roles appear.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `maxPages` | `number` | ❌ No | `5` | Number of Rekrute listing pages to crawl (each page typically contains 15–20 job offers). |
| `concurrency` | `number` | ❌ No | `5` | Number of detailed job pages fetched simultaneously in parallel. |

---

### Parameter Details

#### `maxPages`
Controls how many pages of job listings are traversed from ReKrute.com.
- **Type:** `number`
- **Default:** `5`
- **Recommended Values:**
  - `1` — Quick test or real-time polling of latest additions.
  - `5` — Standard scheduled run (~75–100 job offers).
  - `20` — Deep historical crawl or initial database seeding.

#### `concurrency`
Controls the number of simultaneous HTTP requests made when visiting individual job offer pages.
- **Type:** `number`
- **Default:** `5`
- **Recommended Values:**
  - `2`–`5` — Standard, respectful crawling that minimizes risk of temporary IP blocking.
  - `8`–`10` — High-speed crawling for low page counts.

---

### Tuning Recommendations

| Scenario | `maxPages` | `concurrency` | Expected Listings | Recommended Frequency |
|:---------|:----------:|:-------------:|:-----------------:|:---------------------|
| **Quick Test / Debugging** | `1` | `2` | ~15–20 jobs | On demand |
| **Daily Scheduled Update** | `5` | `5` | ~75–100 jobs | Every 6 to 12 hours |
| **Deep Weekly Sync** | `20` | `3` | ~300–400 jobs | Weekly |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream workflow trigger or execution payload. Starts the scraping process. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `array` | Array of normalized job objects extracted from Rekrute.com. |
| `error` | `Error` | Emitted if the website is unreachable, blocked, or HTML parsing fails. |

---

### Output Data Structure

The `success` output returns an array of structured job records:

```json
[
  {
    "site": "Rekrute.com",
    "titre": "Ingénieur Support Java / J2EE",
    "entreprise": "Capgemini Maroc",
    "ville": "Casablanca",
    "region": "Casablanca-Settat",
    "secteur_activite": "Informatique / Technologies de l'information",
    "fonction": "Informatique / Développement",
    "niveau_experience": "De 3 à 5 ans",
    "niveau_etudes": "Bac +5 et plus",
    "type_contrat": "CDI",
    "teletravail": "Hybride",
    "date_publication": "16/08/2026",
    "description": "Nous recherchons un Ingénieur Support Java / J2EE pour accompagner nos projets bancaires internationaux. Vous serez en charge de l'analyse des incidents, de la maintenance corrective et évolutive des applications...",
    "lien_offre": "https://www.rekrute.com/offre-emploi-ingenieur-support-java-j2ee-casablanca-123456.html"
  },
  {
    "site": "Rekrute.com",
    "titre": "Chef de Projet IT & Digital",
    "entreprise": "Multinationale Conseil",
    "ville": "Rabat",
    "region": "Rabat-Salé-Kénitra",
    "secteur_activite": "Conseil / Audit",
    "fonction": "Gestion de projet / Organisation",
    "niveau_experience": "De 5 à 10 ans",
    "niveau_etudes": "Bac +5 et plus",
    "type_contrat": "CDI",
    "teletravail": "Non",
    "date_publication": "15/08/2026",
    "description": "Dans le cadre du renforcement de nos équipes, nous recrutons un Chef de Projet IT expérimenté pour piloter des chantiers de transformation digitale...",
    "lien_offre": "https://www.rekrute.com/offre-emploi-chef-de-projet-it-digital-rabat-789012.html"
  }
]
```

---

### Output Fields Reference

| Field | Type | Description |
|-------|------|-------------|
| `site` | `string` | Source platform identifier (`"Rekrute.com"`). |
| `titre` | `string` | Full title of the advertised position. |
| `entreprise` | `string` | Name of the hiring enterprise or recruitment agency. |
| `ville` | `string` | Primary city where the position is based (e.g. Casablanca, Rabat, Tanger). |
| `region` | `string` | Administrative region in Morocco. |
| `secteur_activite` | `string` | Industry domain of the hiring company. |
| `fonction` | `string` | Functional role category. |
| `niveau_experience` | `string` | Experience range required (e.g., `Débutant`, `De 3 à 5 ans`, `> 5 ans`). |
| `niveau_etudes` | `string` | Required academic qualification (e.g. `Bac +5 et plus`, `Bac +3`). |
| `type_contrat` | `string` | Contract format (`CDI`, `CDD`, `Freelance`, `Stage`). |
| `teletravail` | `string` | Remote work policy (`Hybride`, `Total`, `Non`). |
| `date_publication` | `string` | Publication or update date of the job listing. |
| `description` | `string` | Full textual description of the role, missions, and required skills. |
| `lien_offre` | `string` | Canonical direct URL to the job posting on ReKrute.com. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Standard Daily Job Crawl

Crawl 5 pages with standard concurrency to collect recent listings.

**Parameter Configuration:**

```text
MaxPages: 5
Concurrency: 5
```

**Result:**
Extracts approximately 75–100 normalized job postings published across Morocco.

---

### Example 2: Quick Smoke Test Run

Scrape only the first page with low concurrency for rapid pipeline testing.

**Parameter Configuration:**

```text
MaxPages: 1
Concurrency: 2
```

**Result:**
Quickly returns the top 15–20 most recent job offers without generating high network traffic.

---

### Example 3: Deep Market Intelligence Collection

Extract up to 20 pages of active job listings with moderate concurrency.

**Parameter Configuration:**

```text
MaxPages: 20
Concurrency: 3
```

**Result:**
Performs a deep crawl of over 300 active job offers across multiple industry sectors.

---

### Example 4: AI Resume Matcher & Alert Workflow

Scrape Rekrute job offers daily and alert qualified candidates via Telegram.

**Workflow Pipeline:**

```text
Cron Trigger (Daily at 08:00 AM)
  → Rekrute Scraper (maxPages: 5, concurrency: 5)
  → For Each (iterate through offers)
  → AI Agent (compare job description with candidate database)
  → If (Match Score > 85%)
  → Telegram Bot (send instant alert with lien_offre)
```

---

### Example 5: Multi-Platform Job Aggregator

Combine listings from multiple Moroccan job boards into a centralized database.

**Workflow Pipeline:**

```text
Manual Trigger
  ├── Rekrute Scraper (maxPages: 3)
  └── Emploi.ma Scraper (maxPages: 3)
        ↓
  Merge / Combine
        ↓
  Function (Deduplicate by company and title)
        ↓
  PostgreSQL / MongoDB (Insert into "moroccan_jobs" collection)
        ↓
  Log (Output total saved listings)
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Scrapes job listings from Rekrute.com using fast HTML parsing
```

### Common Workflow Patterns

- **Daily Market Monitor:** `Cron Trigger` → `Rekrute Scraper` → `Filter Array` (e.g. `ville === "Casablanca"`) → `Google Sheets / Database Append`.
- **Keyword Alert Bot:** `Cron Trigger` → `Rekrute Scraper` → `Function` (regex match for "React" or "Python") → `Slack / Discord Notification`.
- **Talent Pool Matching:** `Rekrute Scraper` → `AI Transform / LLM` → `Postgres Update`.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Rate Limiting or Temporary IP Block (HTTP 429 / 403)
- **Symptom:** Node errors with network timeout or `Access Denied`.
- **Cause:** Setting `concurrency` too high or running deep scrapes too frequently.
- **Solution:** Reduce `concurrency` to `2` or `3`, increase intervals between runs, or add a **Delay** node between batch triggers.

#### Empty Results Array
- **Symptom:** Node completes successfully but returns `[]`.
- **Cause:** Temporary network disruption or anti-scraping challenge triggered by high frequency.
- **Solution:** Lower `maxPages` to `1` or `2` and test again. Verify outbound internet connectivity.

#### Incomplete Job Description Fields
- **Symptom:** Some listings have brief or missing description fields.
- **Cause:** Some hiring companies post brief summaries or link to external career sites.
- **Solution:** Use the provided `lien_offre` to view the original source or handle optional fields gracefully in downstream **Function** nodes.

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `No Rekrute job listings found` | Website structure changed or IP temporarily blocked | Lower concurrency and verify target site accessibility |
| `Network error / Timeout` | Target server unreachable or connection timed out | Reduce `concurrency` and retry |
| `Invalid page number` | `maxPages` set to `0` or negative | Set `maxPages` to a positive integer (`>= 1`) |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Emploi.ma Scraper](../emploima/en.md) — Scrape job listings from Emploi.ma
- [LinkedIn Job Scraper](../linkedin-job-scraper/en.md) — Extract job postings from LinkedIn
- [HTTP Request](../http-request/en.md) — Perform custom REST/HTTP calls to job APIs
- [Function](../function/en.md) — Filter, transform, and clean scraped job records
- [Filter Array](../filter-array/en.md) — Filter job records by city, experience, or keywords
- [Log](../log/en.md) — View scraped job payloads in the execution console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial release supporting fast HTML parsing of Rekrute.com job listings with configurable pagination and concurrency control |

<!-- /SECTION: changelog -->

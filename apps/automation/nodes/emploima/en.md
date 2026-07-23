---
node_id: "emploima"
title: "Emploi.ma Scraper"
description: "Scrape and normalize job listings from Emploi.ma"
category: "web-search"
subcategory: "jobs"
language: "en"
tags: [morocco, jobs, scraping, emploi]
related_nodes: [rekrute-scraper, linkedin-job-scraper]
---

<!-- SECTION: header -->
# Emploi.ma Scraper

> **Category:** Web Search & Information | **Type:** Action Node

Collects job listings from Emploi.ma, then returns normalized job records with the title, company, city, education level, contract type, experience, description, and listing URL.
<!-- /SECTION: header -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Required | Default | Description |
|---|---:|---:|---|
| `maxPages` | No | `5` | Number of listing pages to scan. |
| `concurrency` | No | `5` | Maximum job-detail pages fetched in parallel. |

The website may rate-limit or block automated requests. Start with a small page count and concurrency.
<!-- /SECTION: configuration -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Collect Emploi.ma listings
```
<!-- /SECTION: examples -->

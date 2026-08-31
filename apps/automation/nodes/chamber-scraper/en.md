---
node_id: "chamber-scraper"
title: "Chambre des Représentants Scraper"
description: "Scrape parliamentary questions and detailed government responses from the Moroccan House of Representatives (Chambre des Représentants)."
category: "web-search"
subcategory: "Government & Public Data"
version: "1.0.0"
language: "en"
last_updated: "2026-08-31"
author: "Fusion Team"
tags:
  - morocco
  - parliament
  - chambre-des-representants
  - government
  - scraping
  - questions
  - arabic
related_nodes:
  - http-request
  - function
  - filter-array
  - log
---

<!-- SECTION: header -->
# Chambre des Représentants Scraper

> **Category:** Web Search &amp; Information &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Scrape parliamentary questions, ministerial responses, parliamentary groups, and official document attachments directly from the official portal of the Moroccan Parliament (House of Representatives / [Chambre des Représentants](https://www.chambredesrepresentants.ma)).

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Chambre des Représentants Scraper** node enables automated monitoring and data extraction from Morocco's legislative assembly portal (`chambredesrepresentants.ma`). It collects oral and written questions addressed by parliamentarians to government ministries, along with tracking statuses, official answers, and attached PDF documents.

Built with lightweight HTTP fetching and targeted regular expression extraction, the node operates quickly without requiring a headless browser. It includes built-in request rate limiting (polite delays between page requests) to respect server resources.

### Key Features

- **Dual Operation Modes:** Fast listing extraction (`listingOnly`) or in-depth detail page harvesting (`listingWithDetails`).
- **Flexible Filter Controls:** Filter questions by Ministry ID, Parliamentary Group ID, Parliamentarian ID, and response status (Answered vs. Unanswered).
- **Custom Target URL Support:** Direct the scraper to any specific question category or pre-filtered URL on the parliament portal.
- **Deep Detail Extraction:** Retrieves full question body text, official ministerial answer text, parliamentarian lists, parliamentary group affiliation, question numbers, and PDF file attachments.
- **Automatic Pagination:** Crawls through multiple consecutive listing pages with configurable limits.
- **Dynamic Input Override:** Accepts incoming URLs directly from upstream nodes.

### Use Cases

- **Legislative & Policy Monitoring:** Track questions submitted to specific government ministries (e.g., Health, Education, Equipment & Water).
- **Parliamentary Analytics:** Measure ministerial response rates and analyze topics raised by various political groups.
- **Automated Alerts & Digest:** Run scheduled workflows to detect new parliamentary questions and broadcast summaries to Slack, Microsoft Teams, or Email.
- **Data Archiving & Research:** Extract full parliamentary transcripts and linked PDF documents for NLP research and public policy databases.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `targetUrl` | `string` | ❌ No | `""` | Full URL to scrape. When provided, manual filter query parameters below are ignored. |
| `field_ministeres_new_target_id` | `string` | ❌ No | `""` | Ministry filter target ID (e.g., `11833`). Active only when `targetUrl` is empty. |
| `groupes_id` | `string` | ❌ No | `""` | Parliamentary group filter ID. Active only when `targetUrl` is empty. |
| `parlementaires_associes` | `string` | ❌ No | `""` | Parliamentarian filter ID. Active only when `targetUrl` is empty. |
| `field_transfere_ou_non_value` | `string` | ❌ No | `"All"` | Filter by answer status: `"All"`, `"1"` (Answered), or `"2"` (Not answered). Active only when `targetUrl` is empty. |
| `operation` | `enum` | ❌ No | `"listingWithDetails"` | Operation mode: `listingOnly` (summary cards only) or `listingWithDetails` (fetches individual question pages). |
| `maxPages` | `number` | ❌ No | `5` | Number of pagination pages to crawl. |
| `detailsLimit` | `number` | ❌ No | `10` | Maximum number of items to fetch full details for. Active only when `operation` is `"listingWithDetails"`. |

---

### Parameter Details

#### `targetUrl`
When provided, the scraper targets this exact URL directly (e.g., custom search results or written questions sections like `https://www.chambredesrepresentants.ma/ar/مراقبة-العمل-الحكومي/الأسـئلة-الكتابية`). When left empty, the node automatically constructs the URL targeting the oral questions section (`الأسـئلة-الشفوية`) combined with the filter parameters configured below.

#### `field_ministeres_new_target_id`
Specifies the numeric ID of the ministry to filter by on the Moroccan parliament portal (e.g., `11833` for Ministry of Equipment and Water).

#### `groupes_id`
Specifies the numeric ID corresponding to a parliamentary group (الفريق النيابي).

#### `parlementaires_associes`
Specifies the ID of a specific parliamentarian / deputy (النائب البرلماني).

#### `field_transfere_ou_non_value`
Filters questions based on whether they have been answered by the government:
- `All`: Retrieves both answered and pending questions.
- `1`: Retrieves only answered questions (`أجيب عنه`).
- `2`: Retrieves only unanswered questions (`لم يجب عنه`).

#### `operation`
- `listingOnly`: Crawls listing pages and returns basic metadata (title, URL, publication date, ministry name, questioner, and status).
- `listingWithDetails`: Extracts the listing and additionally visits each question's detail page (up to `detailsLimit`) to extract full question text, answers, and attachments.

#### `maxPages`
The number of paginated pages to fetch (0-indexed). The scraper automatically stops early if no next page link exists or no items are found.

#### `detailsLimit`
Sets an upper cap on how many individual question URLs are scraped for full details in `listingWithDetails` mode. This prevents long execution times when listing pages contain numerous questions.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` / `string` | Upstream data. If a string starting with `http://` or `https://` is provided, it dynamically overrides the target scrape URL. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when scraping succeeds. Contains summary counts and the array of scraped question objects. |
| `error` | `Error` | Emitted if network requests fail, timeout occurs, or no questions are found. |

---

### Output Data Structure

#### 1. When `operation` is `listingOnly`

```json
{
  "totalFound": 20,
  "data": [
    {
      "title": "وضعية البنيات التحتية الطرقية بالعالم القروي",
      "url": "https://www.chambredesrepresentants.ma/ar/مراقبة-العمل-الحكومي/الأسـئلة-الشفوية/16343",
      "date": "2024-02-15",
      "ministry": "وزارة التجهيز والماء",
      "parliament": "محمد بن عبد الله",
      "questionNumber": null,
      "status": "أجيب عنه"
    }
  ]
}
```

#### 2. When `operation` is `listingWithDetails`

```json
{
  "totalScraped": 2,
  "totalFoundInListing": 20,
  "data": [
    {
      "title": "وضعية البنيات التحتية الطرقية بالعالم القروي",
      "url": "https://www.chambredesrepresentants.ma/ar/مراقبة-العمل-الحكومي/الأسـئلة-الشفوية/16343",
      "date": "2024-02-15",
      "ministry": "وزارة التجهيز والماء",
      "parliament": "محمد بن عبد الله",
      "questionNumber": "16343",
      "status": "أجيب عنه",
      "details": {
        "title": "وضعية البنيات التحتية الطرقية بالعالم القروي",
        "questionNumber": "16343",
        "date": "2024-02-20",
        "ministry": "وزارة التجهيز والماء",
        "parliament": "محمد بن عبد الله, فاطمة الزهراء",
        "group": "الفريق الاشتراكي - المعارضة الاتحادية",
        "status": "أجيب عنه",
        "questionText": "السيد الوزير المحترم، يعاني سكان العالم القروي من صعوبة التنقل بسبب تدهور بعض المسالك الطرقية...",
        "answerText": "جواب السيد الوزير: تضع الوزارة تأهيل الشبكة الطرقية القروية ضمن أولوياتها من خلال برنامج تقليص الفوارق المجالية...",
        "attachments": [
          {
            "title": "وثيقة الجواب.pdf",
            "url": "https://www.chambredesrepresentants.ma/sites/default/files/questions/16343_reponse.pdf"
          }
        ]
      }
    }
  ]
}
```

### Output Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `totalFound` / `totalFoundInListing` | `number` | Total number of question items discovered across the crawled listing pages. |
| `totalScraped` | `number` | Number of question items for which full details were scraped (in `listingWithDetails` mode). |
| `data[].title` | `string` | Question subject / title in Arabic. |
| `data[].url` | `string` | Full direct URL to the question page on the parliament portal. |
| `data[].date` | `string` | Date associated with the question or answer. |
| `data[].ministry` | `string` | Target government ministry name (الوزارة المختصة). |
| `data[].parliament` | `string` | Name(s) of the parliamentarian(s) who authored the question. |
| `data[].status` | `string` | Answer status (`"أجيب عنه"` or `"لم يجب عنه"`). |
| `data[].details.questionNumber` | `string` | Official parliamentary question registration number (رقم السؤال). |
| `data[].details.group` | `string` | Parliamentary political group name (الفريق النيابي). |
| `data[].details.questionText` | `string` | Full text body of the question submitted to the ministry. |
| `data[].details.answerText` | `string` | Full text body of the government / ministerial response. |
| `data[].details.attachments` | `array` | List of PDF document links attached to the question or answer. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Fast Listing of Recent Oral Questions

Quickly fetch the 20 most recent questions without opening each individual detail page.

**Parameter Configuration:**
```text
Operation: listingOnly
MaxPages: 1
```

---

### Example 2: Scrape Full Details & Government Answers

Fetch the latest 5 questions along with their full text, ministerial answers, and PDF attachments.

**Parameter Configuration:**
```text
Operation: listingWithDetails
MaxPages: 1
DetailsLimit: 5
```

---

### Example 3: Filter Answered Questions Only

Target only questions that have received an official response (`أجيب عنه`).

**Parameter Configuration:**
```text
Operation: listingWithDetails
FieldTransfereOuNonValue: 1
MaxPages: 2
DetailsLimit: 10
```

---

### Example 4: Scrape Written Questions via Custom Target URL

Direct the scraper to extract written parliamentary questions (`الأسـئلة الكتابية`) instead of the default oral questions.

**Parameter Configuration:**
```text
TargetUrl: https://www.chambredesrepresentants.ma/ar/مراقبة-العمل-الحكومي/الأسـئلة-الكتابية
Operation: listingWithDetails
MaxPages: 1
DetailsLimit: 3
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Scrape Moroccan Parliament Questions and Details
```

### How it Flows

1. **Manual Trigger (or Cron Trigger):** Initiates the workflow manually or on a predefined schedule.
2. **Chambre des Représentants Scraper:** Connects to `chambredesrepresentants.ma`, crawls the listing pages, and extracts structured question and answer records.
3. **Downstream Processing:**
   - **Function Node:** Filters questions by specific topics or reformats Arabic text for summary reports.
   - **Database / Google Sheets:** Archives parliamentary questions and response timelines over time.
   - **AI Chat / LLM Node:** Summarizes lengthy ministerial replies or generates press briefs.
   - **Messaging Nodes (Slack / Teams / Email):** Dispatches real-time alerts when high-priority parliamentary questions are published.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `No questions found`
- **Cause:** The requested filter returned zero results, the parliament portal is experiencing downtime, or the HTML markup changed.
- **Solution:** Verify the URL and filter IDs in a web browser. Ensure the `targetUrl` starts with `http://` or `https://`.

#### `Request timeout`
- **Cause:** The target website server took longer than 10 seconds to respond.
- **Solution:** The node automatically catches timeouts and skips failed pages. If persistent, reduce `maxPages` or retry during off-peak hours.

#### Detail fields are `null`
- **Cause:** The question has not been answered yet (unanswered status), or the specific detail (such as team image or attachment) is not present on that question page.
- **Solution:** This is normal for questions still awaiting response (`لم يجب عنه`). Check the `status` field.

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Invalid target URL: ...` | `targetUrl` or input data does not start with `http://` or `https://` | Check the URL format |
| `No questions found...` | Empty result page, network block, or invalid filter ID | Verify filters and website availability |
| `Scraping failed: ...` | Network connection interrupted during crawling | Check internet connection and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](../http-request/en.md) – Send generic HTTP requests to any web endpoint or API
- [Function](../function/en.md) – Process, transform, or aggregate scraped parliamentary records
- [Filter Array](../filter-array/en.md) – Filter question records by ministry, group, or keyword
- [Log](../log/en.md) – Output scraped question summaries to the workflow execution console
- [Hespress](../hespress/en.md) – Monitor Moroccan national and political news feeds

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-31 | Initial documentation for Chambre des Représentants Scraper node |

<!-- /SECTION: changelog -->

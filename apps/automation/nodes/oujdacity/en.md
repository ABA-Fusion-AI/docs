---
node_id: "oujdacity"
title: "OujdaCity"
description: "Fetch news from OujdaCity RSS feed."
category: "utilities"
subcategory: "news"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - morocco
  - news
  - oujda
  - oujdacity
  - oriental
  - rss
  - feed
  - articles
  - action
related_nodes:
  - hespress
  - hespress-english
  - agadir24
  - le-matin
  - http-request
  - function
  - log
  - filter-array
---

<!-- SECTION: header -->
# OujdaCity

> **Category:** Utilities | **Type:** Action Node

Fetch the latest news articles and regional updates directly from the [OujdaCity](https://www.oujdacity.net/) RSS feed.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **OujdaCity** node retrieves current headlines, regional reports, social news, and cultural updates from OujdaCity (`oujdacity.net`), one of the premier regional and national news portals serving Oujda and the Oriental region of Morocco.

It fetches and parses the official RSS feed into clean, structured JSON objects containing article headlines, publication dates, direct web URLs, summary descriptions, and categories.

### Key Features

- **Regional & National Moroccan News:** Real-time coverage focusing on Oujda, the Oriental region, and broader Moroccan news.
- **Configurable Limit:** Adjust `maxItems` to restrict how many news items are retrieved.
- **Structured JSON Output:** Formats raw RSS XML into easy-to-consume JSON arrays.
- **Zero Configuration / Keyless:** No credentials, API tokens, or account setup required.

### Use Cases

- **Regional News Monitoring:** Track local developments, society, and municipal events in Oujda and the Oriental province.
- **Automated News Digest:** Send daily briefings of top regional stories to Telegram, Slack, WhatsApp, or Email.
- **Content Aggregation:** Aggregate Moroccan news from various regions alongside other regional nodes (e.g., Hespress, Agadir 24).
- **AI Sentiment & Analysis:** Pass news headlines to LLMs or AI agents for summarization, translation, or sentiment analysis.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `maxItems` | `number` | ❌ No | `10` | Maximum number of news articles to retrieve from the RSS feed. |

---

### Parameter Details

#### `maxItems`
Specifies the maximum count of article objects returned in the output array.
- Default: `10`
- Example values: `5`, `10`, `20`

---

### Feed URL

The node fetches news directly from the official OujdaCity RSS feed:

```text
https://www.oujdacity.net/feed
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming trigger or workflow data to start node execution. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `array` | Emitted when fetching succeeds. Returns an array of parsed article objects. |
| `error` | `Error` | Emitted when the network request or RSS parsing fails. |

---

### Output Data Structure

The `success` output provides an array of structured news article objects:

```json
[
  {
    "title": "افتتاح فعاليات المعرض الجهوي للاقتصاد الاجتماعي والتضامني بوجدة",
    "link": "https://www.oujdacity.net/regional-news-10234.html",
    "pubDate": "Mon, 17 Aug 2026 10:15:00 +0000",
    "description": "انطلقت بمدينة وجدة فعاليات المعرض الجهوي للاقتصاد الاجتماعي والتضامني بمشاركة العديد من التعاونيات والفاعلين...",
    "category": "الجهة الشرقية",
    "author": "OujdaCity",
    "guid": "https://www.oujdacity.net/?p=10234"
  },
  {
    "title": "مشاريع تنموية جديدة لتعزيز البنية التحتية بجهة الشرق",
    "link": "https://www.oujdacity.net/development-projects-10235.html",
    "pubDate": "Mon, 17 Aug 2026 09:30:00 +0000",
    "description": "صادق مجلس جهة الشرق على مجموعة من المشاريع التنموية الرامية إلى تأهيل البنية التحتية ودعم التشغيل...",
    "category": "اقتصاد",
    "author": "OujdaCity",
    "guid": "https://www.oujdacity.net/?p=10235"
  }
]
```

| Field | Type | Description |
|-------|------|-------------|
| `title` | `string` | Headline of the news article |
| `link` | `string` | Direct web URL to the full article on OujdaCity |
| `pubDate` | `string` | Article publication timestamp (RFC 822 / ISO format) |
| `description` | `string` | Short snippet, excerpt, or summary of the article |
| `category` | `string` | Category or section tag (e.g. الجهة الشرقية, اقتصاد, ثقافة) |
| `author` | `string` | Article author or publisher name |
| `guid` | `string` | Unique identifier or canonical URL for the RSS item |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Fetch Latest 10 News Articles

Fetch the 10 most recent news articles from OujdaCity.

**Parameter Configuration:**

```text
MaxItems: 10
```

---

### Example 2: Morning Regional News Briefing

Fetch 5 articles every morning at 08:30 AM and deliver them via email or messaging app.

**Workflow Pattern:**

```text
Cron Trigger (08:30 AM)
  → OujdaCity (maxItems: 5)
  → Function (format articles into bullet points)
  → Email / WhatsApp / Telegram
```

---

### Example 3: Filter News by Category or Keyword

Fetch articles and filter for specific regional topics or keywords.

**Workflow Pattern:**

```text
Manual Trigger
  → OujdaCity (maxItems: 20)
  → Filter Array (item.title.includes('وجدة') || item.category === 'الجهة الشرقية')
  → Log
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Fetch news from OujdaCity RSS feed
```

### Common Patterns

- **Daily Briefing:** Cron Trigger → OujdaCity → Teams / Slack / Discord.
- **Regional Aggregation:** OujdaCity + Agadir 24 + Hespress → Merge → Unified Feed.
- **AI Executive Summary:** OujdaCity → AI Agent (summarize local news) → Email.
- **Archiving & Database Sync:** OujdaCity → For Each → Database (PostgreSQL / MySQL / Firestore).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Empty results returned

**Cause:** The OujdaCity RSS feed server may be temporarily down or undergoing maintenance.

**Solution:** Retry after a brief delay. Verify internet connectivity to `www.oujdacity.net`.

#### UTF-8 Character Encoding

**Cause:** Downstream logger or external database does not properly handle Arabic or UTF-8 characters.

**Solution:** Ensure all subsequent workflow steps, database tables, and communication channels support UTF-8 character encoding.

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Failed to fetch RSS feed` | Network failure or DNS resolution issue | Check outbound server internet connection |
| `Invalid RSS XML format` | Remote endpoint returned non-XML error page | Retry after a few moments |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Hespress](../hespress/en.md) — Fetch news from Hespress Arabic RSS feed
- [Hespress English](../hespress-english/en.md) — Fetch news from Hespress English RSS feed
- [Agadir 24](../agadir24/en.md) — Fetch news from Agadir 24 RSS feed
- [Le Matin](../le-matin/en.md) — Fetch news from Le Matin RSS feed by category
- [HTTP Request](../http-request/en.md) — Fetch custom RSS feeds or REST endpoints
- [Filter Array](../filter-array/en.md) — Filter news articles by category or keywords
- [Function](../function/en.md) — Transform news data into custom formats
- [Log](../log/en.md) — Display news objects in workflow execution logs

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial release |

<!-- /SECTION: changelog -->

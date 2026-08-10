---
node_id: "hespress"
title: "Hespress"
description: "Fetch news from Hespress RSS feed."
category: "utilities"
subcategory: "news"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - morocco
  - news
  - hespress
  - rss
  - feed
  - arabic
related_nodes:
  - hespress-english
  - http-request
  - function
  - log
  - filter-array
---

<!-- SECTION: header -->
# Hespress

> **Category:** Utilities | **Type:** Action Node

Fetch the latest news articles directly from the main Arabic [Hespress](https://www.hespress.com/) RSS feed.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Hespress** node retrieves current news headlines, political reports, economic updates, and national stories from Hespress (`hespress.com`), the leading online news portal in Morocco.

It fetches and parses the official RSS feed into clean, structured JSON objects containing article titles, publication dates, direct web URLs, summary descriptions, and categories.

### Key Features

- **Real-Time Arabic News:** Access breaking news and top stories from Morocco's premier news source.
- **Configurable Limit:** Adjust `maxItems` to control how many news stories are retrieved.
- **Structured JSON Output:** Formats raw RSS XML entries into easy-to-process JSON objects.
- **Zero Configuration / Keyless:** No credentials or API tokens required.

### Use Cases

- **Moroccan News Alerting:** Monitor headlines in real-time and send notifications to WhatsApp, Telegram, or Email.
- **Media Monitoring:** Track national events, government announcements, and regional breaking news.
- **Content Syndication:** Aggregate Arabic news stories for display on enterprise dashboards or portals.
- **NLP & Sentiment Processing:** Feed Arabic news text to LLMs or translation nodes for downstream analysis.

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
- Example values: `5`, `10`, `25`

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming trigger or workflow data. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `array` | Emitted when fetching succeeds. Contains an array of parsed article objects. |
| `error` | `Error` | Emitted when network request or feed parsing fails. |

---

### Output Data Structure

The `success` output provides an array of structured news article objects:

```json
[
  {
    "title": "مجلس الحكومة يدارس عددا من مشاريع المراسيم والقوانين",
    "link": "https://www.hespress.com/goverment-meeting-102345.html",
    "pubDate": "Mon, 10 Aug 2026 14:30:00 +0000",
    "description": "يعقد مجلس الحكومة يوم الخميس المقبل اجتماعا لمدارسة عدد من النصوص القانونية...",
    "category": "سياسة",
    "author": "هسبريس"
  },
  {
    "title": "ارتفاع أسعار الفائدة والسياسة النقدية للمغرب",
    "link": "https://www.hespress.com/economy-news-102346.html",
    "pubDate": "Mon, 10 Aug 2026 12:15:00 +0000",
    "description": "أفاد بنك المغرب في نشريته الأسبوعية الأخيرة بأن سعر الصرف وتوقعات التضخم...",
    "category": "اقتصاد",
    "author": "هسبريس"
  }
]
```

| Field | Type | Description |
|-------|------|-------------|
| `title` | `string` | Headline of the news article (in Arabic) |
| `link` | `string` | Direct URL to the article on Hespress |
| `pubDate` | `string` | Article publication timestamp (RFC 822 format) |
| `description` | `string` | Short abstract or summary snippet |
| `category` | `string` | Article category or section (e.g. سياسة, اقتصاد, مجتمع) |
| `author` | `string` | Publisher or author identifier |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Fetch Top 10 Moroccan News Stories

Fetch the 10 most recent news articles from Hespress.

**Parameter Configuration:**

```text
MaxItems: 10
```

---

### Example 2: Morning News Briefing Workflow

Fetch 5 articles every morning at 8:00 AM and send them as a summary message.

**Workflow Pattern:**

```text
Cron Trigger (08:00 AM)
  → Hespress (maxItems: 5)
  → Function (format Arabic text into bullet points)
  → Email / Messaging Node
```

---

### Example 3: Filter Political News Only

Fetch recent articles and filter specifically for political news (`سياسة`).

**Workflow Pattern:**

```text
Manual Trigger
  → Hespress (maxItems: 20)
  → Filter Array (item.category === 'سياسة')
  → Log
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Fetch news from Hespress RSS feed
```

### Common Patterns

- **Live Digest:** Cron Trigger → Hespress → Email Send / Slack.
- **Multilingual Aggregation:** Hespress (Arabic) + Hespress English → Merge → Broadcast.
- **Topic Filter:** Hespress → Filter Array (by keyword or category) → Log / Database.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Empty array returned

**Cause:** The Hespress RSS feed server is experiencing high traffic or temporary downtime.

**Solution:** Retry after a brief delay. Verify internet connectivity to `www.hespress.com`.

#### Encoding issues with Arabic text

**Cause:** Downstream system or logger does not support UTF-8 character encoding.

**Solution:** Ensure all subsequent nodes handle UTF-8 encoding properly.

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Failed to fetch RSS feed` | Network failure or DNS resolution error | Check internet connection |
| `Invalid RSS XML format` | Server returned an error page instead of XML | Retry after a few minutes |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Hespress English](../hespress-english/en.md) — Fetch news from Hespress English RSS feed
- [HTTP Request](../http-request/en.md) — Custom RSS or HTTP endpoint requests
- [Filter Array](../filter-array/en.md) — Filter articles by category or keyword
- [Function](../function/en.md) — Custom formatting of news items
- [Log](../log/en.md) — Display news objects in workflow console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-10 | Initial release |

<!-- /SECTION: changelog -->

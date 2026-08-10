---
node_id: "hespress-english"
title: "Hespress English"
description: "Fetch news from Hespress English RSS feed."
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
  - english
related_nodes:
  - http-request
  - function
  - log
  - filter-array
---

<!-- SECTION: header -->
# Hespress English

> **Category:** Utilities | **Type:** Action Node

Fetch the latest English news articles directly from the official [Hespress English](https://en.hespress.com/) RSS feed.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Hespress English** node retrieves published news articles, breaking stories, and political/economic reports from Hespress English (`en.hespress.com`), Morocco's leading digital news platform.

It parses the official RSS feed, converting raw XML news entries into clean, structured JSON objects containing article titles, publication dates, direct URLs, descriptions, and media metadata.

### Key Features

- **Real-Time News Stream:** Pulls the latest published stories from Hespress English.
- **Configurable Limit:** Set `maxItems` to restrict the number of returned articles.
- **Structured JSON Output:** Returns clean JavaScript arrays containing title, link, date, summary, and author details.
- **No API Key Required:** Works out of the box using public RSS feeds.

### Use Cases

- **Automated News Digest:** Send daily summaries of top Moroccan news to Telegram, Slack, or email.
- **Media Monitoring & Analytics:** Track headlines and coverage regarding Moroccan economics, diplomacy, or sports.
- **Content Aggregation:** Feed English news articles into external portals, mobile apps, or RSS readers.
- **AI Sentiment Analysis:** Pass news headlines to AI nodes (e.g. OpenAI, OpenRouter) to generate sentiment scores or executive summaries.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `maxItems` | `number` | ❌ No | `10` | Maximum number of news articles to retrieve from the feed. |

---

### Parameter Details

#### `maxItems`
Limits the number of news items returned in the output array.
- Default: `10`
- Example values: `5`, `10`, `20`

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming data from the preceding node (used to trigger execution). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `array` | Emitted when news fetching succeeds. Returns an array of article objects. |
| `error` | `Error` | Emitted when network request or RSS parsing fails. |

---

### Output Data Structure

The `success` output returns an array of structured article objects:

```json
[
  {
    "title": "Morocco, Spain Highlight Exemplary Bilateral Partnership",
    "link": "https://en.hespress.com/102345-morocco-spain-highlight-exemplary-bilateral-partnership.html",
    "pubDate": "Mon, 10 Aug 2026 14:30:00 +0000",
    "description": "Rabat - Minister of Foreign Affairs Nasser Bourita held talks on Monday with his Spanish counterpart...",
    "category": "Politics",
    "author": "Hespress English"
  },
  {
    "title": "Moroccan Dirham Appreciates Against Euro in Early August",
    "link": "https://en.hespress.com/102346-moroccan-dirham-appreciates-against-euro.html",
    "pubDate": "Mon, 10 Aug 2026 12:15:00 +0000",
    "description": "Bank Al-Maghrib reports a 0.35% appreciation of the dirham against the euro during the first week of August...",
    "category": "Economy",
    "author": "Hespress English"
  }
]
```

| Field | Type | Description |
|-------|------|-------------|
| `title` | `string` | Headline of the news article |
| `link` | `string` | Direct web URL to the full article on Hespress English |
| `pubDate` | `string` | Article publication timestamp (RFC 822 format) |
| `description` | `string` | Text snippet or summary of the news story |
| `category` | `string` | News category or topic classification (if available) |
| `author` | `string` | Article author or publisher name |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Fetch Latest 10 News Articles

Fetch the top 10 current news headlines from Hespress English.

**Parameter Configuration:**

```text
MaxItems: 10
```

---

### Example 2: Daily Email News Digest

Periodically fetch the top 5 stories every morning and email them.

**Workflow Pattern:**

```text
Cron Trigger (08:00 AM)
  → Hespress English (maxItems: 5)
  → Function (format articles into HTML email list)
  → Email Send
```

---

### Example 3: Headline Sentiment Analysis with AI

Fetch news and analyze sentiment using an AI model.

**Workflow Pattern:**

```text
Manual Trigger
  → Hespress English (maxItems: 10)
  → AI Agent (analyze sentiment of each headline)
  → Log
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Fetch news from Hespress English RSS feed
```

### Common Patterns

- **News Notification:** Cron Trigger → Hespress English → Teams / Slack Notification.
- **Archiving & Database Sync:** Hespress English → For Each → Database Insert.
- **Category Filtering:** Hespress English → Filter Array (where category === 'Economy') → Log.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Empty results returned

**Cause:** Temporary network downtime or connection timeout reaching `en.hespress.com`.

**Solution:** Retry execution. Ensure the server running the workflow has outbound internet connectivity to HTTPS port 443.

#### CORS or Network Timeout Error

**Cause:** Hespress RSS feed endpoint may be temporarily throttled or rate-limited.

**Solution:** Keep `maxItems` at a reasonable number (e.g. 10 to 20) and avoid triggering the feed multiple times per second.

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Failed to fetch RSS feed` | Network failure or DNS resolution issue | Check server internet connectivity |
| `Invalid RSS XML format` | Remote feed returned non-XML error page | Retry after a few minutes |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](../http-request/en.md) — Fetch custom RSS feeds or APIs
- [Filter Array](../filter-array/en.md) — Filter news articles by category or keywords
- [Function](../function/en.md) — Reformat news items into HTML/Markdown
- [Log](../log/en.md) — Inspect fetched news objects

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-10 | Initial release |

<!-- /SECTION: changelog -->

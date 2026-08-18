---
node_id: "la-vie-eco"
title: "La Vie Eco"
description: "Fetch news from La Vie Eco RSS feed."
category: "Web Search & Information"
subcategory: "News"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - morocco
  - news
  - la-vie-eco
  - rss
  - feed
  - economics
  - ecology
related_nodes:
  - http-request
  - filter-array
  - hespress
  - le-matin
  - oujdacity
---

<!-- SECTION: header -->
# La Vie Eco

> **Category:** Web Search & Information | **Type:** Action Node

Fetch the latest news articles from the La Vie Eco RSS feed for economic, ecological, and business coverage from Morocco.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **La Vie Eco** node retrieves published news articles, economic reports, and ecological stories from the official La Vie Eco RSS feed. It parses the RSS feed, converting raw XML news entries into clean, structured JSON objects containing article titles, publication dates, direct URLs, descriptions, and metadata.

The node is useful for monitoring economic trends, business news, and environmental coverage from a Moroccan perspective.

### Key Features

- **Real-Time Economic News:** Pulls the latest published business and ecological stories
- **Configurable Limit:** Set `maxItems` to restrict the number of returned articles
- **Structured JSON Output:** Returns clean JavaScript arrays containing title, link, date, summary, and author details
- **RSS Feed Parsing:** Automatically extracts and normalizes RSS metadata
- **No API Key Required:** Works out of the box using public RSS feeds

### Typical Use Cases

- **Automated News Digest:** Send daily economic news summaries to Telegram, Slack, or email
- **Economic Monitoring:** Track headlines and coverage regarding Moroccan economics and business
- **Content Aggregation:** Feed economic articles into dashboards, apps, or news readers
- **AI Analysis:** Pass headlines to AI nodes for sentiment analysis or executive summaries
- **Business Intelligence:** Collect and analyze economic trends from Morocco

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `maxItems` | `number` | ❌ No | `10` | Maximum number of news articles to retrieve from the feed |

---

### Parameter Details

#### `maxItems`

Limits the number of news items returned in the output array.

- Default: `10`
- Recommended values: `5`, `10`, `15`, `20`
- Accepts any positive integer

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming data from the preceding node (used to trigger execution) |

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
    "title": "Morocco Boosts Solar Energy Investment",
    "link": "https://lavieco.ma/articles/solar-investment",
    "pubDate": "Mon, 18 Aug 2026 14:30:00 +0000",
    "description": "Morocco announces new solar energy initiatives to boost renewable power capacity...",
    "category": "Energy",
    "author": "La Vie Eco"
  },
  {
    "title": "Economic Growth Report: Q2 2026 Analysis",
    "link": "https://lavieco.ma/articles/q2-growth",
    "pubDate": "Mon, 18 Aug 2026 12:15:00 +0000",
    "description": "Economic analysis showing Morocco's GDP growth of 3.5% in Q2 2026...",
    "category": "Economy",
    "author": "La Vie Eco"
  }
]
```

| Field | Type | Description |
|-------|------|-------------|
| `title` | `string` | Headline of the news article |
| `link` | `string` | Direct web URL to the full article |
| `pubDate` | `string` | Article publication timestamp (RFC 822 format) |
| `description` | `string` | Text snippet or summary of the news story |
| `category` | `string` | News category or topic classification (if available) |
| `author` | `string` | Article author or publisher name |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Fetch Latest 10 Economic News Articles

Fetch the top 10 current economic and ecological news headlines from La Vie Eco.

**Parameter Configuration:**

```text
maxItems: 10
```

**Result:**

```json
[
  {
    "title": "Morocco Boosts Solar Energy Investment",
    "link": "https://lavieco.ma/articles/solar-investment",
    "pubDate": "Mon, 18 Aug 2026 14:30:00 +0000",
    "description": "Morocco announces new solar energy initiatives..."
  }
]
```

---

### Example 2: Daily Economic News Digest

Periodically fetch the top 5 economic stories every morning and email them.

**Workflow Pattern:**

```text
Cron Trigger (08:00 AM)
  → La Vie Eco (maxItems: 5)
  → Function (format articles into email)
  → Email Send
```

---

### Example 3: Economic News Sentiment Analysis

Fetch news and analyze sentiment using an AI model.

**Workflow Pattern:**

```text
Manual Trigger
  → La Vie Eco (maxItems: 10)
  → AI Agent (analyze sentiment of headlines)
  → Log Results
```

---

### Example 4: Filter Business News by Category

Extract and filter only business-related articles.

**Workflow Pattern:**

```text
Manual Trigger
  → La Vie Eco (maxItems: 20)
  → Filter Array (category === "Economy")
  → Database Insert
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Fetch news from La Vie Eco RSS feed
```

### Common Patterns

- **News Notification:** Cron Trigger → La Vie Eco → Slack / Teams Notification
- **Database Sync:** La Vie Eco → For Each → Database Insert
- **Category Filtering:** La Vie Eco → Filter Array (where category === 'Energy') → Log
- **Content Aggregation:** La Vie Eco → Array Merge → Dashboard Display

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Empty results returned

**Cause:** Temporary network downtime or connection timeout reaching the La Vie Eco RSS feed.

**Solution:** Retry execution. Ensure the server running the workflow has outbound internet connectivity to HTTPS port 443.

#### CORS or Network Timeout Error

**Cause:** La Vie Eco RSS feed endpoint may be temporarily throttled or rate-limited.

**Solution:** Keep `maxItems` at a reasonable number (10 to 20) and avoid triggering the feed multiple times per second.

#### Invalid RSS Format Error

**Cause:** Remote feed returned non-XML content or changed structure.

**Solution:** Retry after a few minutes. Verify the source feed is still accessible.

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Failed to fetch RSS feed` | Network failure or DNS resolution issue | Check server internet connectivity |
| `Invalid RSS XML format` | Remote feed returned non-XML error page | Retry after a few minutes |
| `Connection timeout` | Feed server took too long to respond | Retry or reduce `maxItems` |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](../http-request/en.md) — Fetch custom RSS feeds or APIs
- [Filter Array](../filter-array/en.md) — Filter news articles by category or keywords
- [Function](../function/en.md) — Reformat news items into HTML/Markdown
- [Hespress](../hespress/en.md) — Moroccan general news feed
- [Le Matin](../le-matin/en.md) — Moroccan news by category
- [Log](../log/en.md) — Inspect fetched news objects

<!-- /SECTION: related -->

---

<!-- SECTION: security -->
## Security

The node uses public RSS feeds with no authentication required. No sensitive credentials need to be stored. Ensure your server has outbound internet connectivity to access the feed.

<!-- /SECTION: security -->

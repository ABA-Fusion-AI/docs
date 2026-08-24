---
node_id: "tanja-news"
title: "Tanja News"
description: "Fetch news from Tanja News RSS feed."
category: "web-search-information"
subcategory: "news"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - morocco
  - tangier
  - tanja-news
  - news
  - rss
  - feed
  - arabic
related_nodes:
  - hespress
  - hespress-english
  - oujdacity
  - map-news
  - http-request
  - ai-chat
---

<!-- SECTION: header -->
# Tanja News

> **Category:** Web Search & Information | **Subcategory:** News | **Type:** Action Node

Fetch the latest regional and national news headlines directly from the [Tanja News](https://tanjanews.com/) (طنجة نيوز) RSS feed.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Tanja News** node retrieves current news headlines, regional reports, community events, and economic updates from Tanja News (`tanjanews.com`), one of the leading regional news portals focused on Tangier, Tetouan, Al Hoceima, and Northern Morocco.

It fetches the live RSS XML feed and automatically parses it into a clean, structured JSON object containing channel metadata and an array of news items with titles, publication dates, direct article links, summaries, and category tags.

### Key Features

- **Regional & National Moroccan News:** Monitor real-time local updates and breaking news from Tangier and the northern region of Morocco.
- **Automatic XML & CDATA Parsing:** Automatically decodes XML entities and strips CDATA formatting to output clean, readable Arabic text.
- **Configurable Item Limit:** Use `maxItems` to restrict the number of returned articles per request.
- **Rich Article Metadata:** Extracts article titles, links, summaries (`description`), publication dates (`pubDate`), authors, GUIDs, and category tags.
- **Zero Configuration / Keyless:** Connects directly to the public RSS endpoint with no API keys or credentials needed.

### Processing Flow

```text
Workflow Trigger (Manual or Scheduled)
  ↓
Read maxItems Parameter (default: 10)
  ↓
Fetch live RSS XML from https://tanjanews.com/feed
  ↓
Parse XML channel info and article items
  ↓
Decode XML entities (CDATA, HTML entities)
  ↓
Slice items array to maxItems limit
  ↓
Emit structured RSSFeed object to success output
```

### Use Cases

- **Regional News Alerting:** Monitor Tangier and Northern Morocco news on an hourly schedule and post breaking updates to Telegram, WhatsApp, or Slack.
- **AI-Powered Daily Digest:** Feed incoming Tanja News articles to an [AI Chat](../ai-chat/en.md) node to generate a 3-bullet morning summary of local events.
- **Media Monitoring & Aggregation:** Aggregate articles from multiple Moroccan news portals (Hespress, Tanja News, OujdaCity, MAP News) into a unified dashboard.
- **Sentiment & Content Analysis:** Track local civic topics, infrastructure projects, and public discussions in Northern Morocco.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `maxItems` | `number` | No | `10` | Maximum number of news articles to retrieve from the RSS feed (e.g. `1`, `5`, `10`, `20`). |

### Default Configuration

```json
{
  "maxItems": 10
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow trigger or previous node payload passed to trigger the execution step. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Returned on successful feed retrieval, containing feed channel details and an array of parsed article items. |
| `error` | `object` | Returned if network or XML parsing fails. |

### Output Schema

```typescript
{
  title?: string;          // Channel title (e.g. "طنجة نيوز | TanjaNews")
  link?: string;           // Channel homepage URL ("https://tanjanews.com")
  description?: string;    // Channel description
  language?: string;       // Language code (e.g. "ar")
  lastBuildDate?: string;  // Last build date string
  pubDate?: string;        // Feed publication date
  items: Array<{
    title?: string;        // Article title / headline
    link?: string;         // Direct URL to the full article
    description?: string;  // Article summary / excerpt
    pubDate?: string;      // Publication date string (e.g. "Mon, 24 Aug 2026 18:30:00 +0000")
    author?: string;       // Author name, if specified
    guid?: string;         // Unique article identifier / URL
    category?: string[];   // Array of article category tags (e.g. ["حوادث", "طنجة"])
  }>;
}
```

### Successful Response Example

```json
{
  "title": "طنجة نيوز - موقع إخباري مغربي شامل",
  "link": "https://tanjanews.com",
  "description": "آخر الأخبار والمستجدات في طنجة والشمال والمغرب",
  "language": "ar",
  "items": [
    {
      "title": "افتتاح مشروع تنموي جديد بمدينة طنجة",
      "link": "https://tanjanews.com/article-example-123",
      "description": "شهدت مدينة طنجة اليوم تدشين مشروع جديد يهدف إلى تعزيز البنية التحتية...",
      "pubDate": "Mon, 24 Aug 2026 16:45:00 +0000",
      "author": "طاقم التحرير",
      "guid": "https://tanjanews.com/?p=12345",
      "category": ["طنجة", "اقتصاد", "محليات"]
    }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Latest Breaking News (1 Article)

Fetch only the most recent article headline from Tanja News:

```json
{
  "maxItems": 1
}
```

### Example 2: Top 5 Recent Stories

Fetch the top 5 latest articles for a daily summary:

```json
{
  "maxItems": 5
}
```

### Example 3: Default Feed Extraction (10 Articles)

```json
{
  "maxItems": 10
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Fetch the latest Tanja News articles and inspect the feed
```

### Common Workflow Patterns

- **Breaking News Bot:** Scheduled Trigger (Every 30 min) → Tanja News (`maxItems: 1`) → For-Each → Telegram / Discord Webhook (Send article title & link).
- **Arabic News Digest:** Scheduled Trigger (Every morning) → Tanja News (`maxItems: 5`) → AI Chat (Summarize articles into bullet points) → Email Send.
- **Regional News Aggregator:** Manual Trigger → Tanja News + Hespress + OujdaCity → Function (Combine feeds) → Google Sheets.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Feed fetch failed / network timeout

**Cause:** The target website `tanjanews.com` is temporarily unreachable or the workflow server cannot resolve the host.

**Solution:** Check internet connectivity, test accessing `https://tanjanews.com/feed` in a browser, and retry the workflow.

### Empty items array returned

**Cause:** The RSS feed at `tanjanews.com/feed` returned valid XML with zero `<item>` elements.

**Solution:** Verify that the website's RSS feed is active and has recently published content.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Hespress](../hespress/en.md) - Fetch national news from Hespress RSS feed
- [Hespress English](../hespress-english/en.md) - Fetch English-language news from Hespress
- [OujdaCity](../oujdacity/en.md) - Fetch regional news from Eastern Morocco (Oujda)
- [AI Chat](../ai-chat/en.md) - Summarize and translate Arabic news headlines

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for Tanja News node |

<!-- /SECTION: changelog -->

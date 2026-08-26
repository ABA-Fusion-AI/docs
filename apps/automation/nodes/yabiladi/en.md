---
node_id: "yabiladi"
title: "Yabiladi"
description: "Fetch news from Yabiladi RSS feed."
category: "utilities"
subcategory: "news"
version: "1.0.0"
language: "en"
last_updated: "2026-08-26"
author: "Fusion Team"
tags:
  - yabiladi
  - morocco
  - maroc
  - news
  - rss
  - feed
  - articles
  - diaspora
  - mre
  - french
related_nodes:
  - hespress
  - hespress-english
  - map-news
  - le-matin
  - http-request
  - function
  - filter-array
  - log
---

<!-- SECTION: header -->
# Yabiladi

> **Category:** Utilities | **Subcategory:** News | **Type:** Action Node

Fetch the latest Moroccan news, diaspora reports, and societal updates directly from the official [Yabiladi](https://www.yabiladi.com/) RSS feed.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Yabiladi** node connects workflows directly to the official RSS feed of [Yabiladi.com](https://www.yabiladi.com/), a premier digital news and community portal in Morocco with dedicated coverage of national politics, economy, society, culture, and the Moroccan diaspora (Marocains Résidant à l'Étranger - MRE).

The node automatically retrieves the remote XML feed (`https://www.yabiladi.com/rss/`), parses channel-level metadata and article entries, strips CDATA blocks, decodes XML character entities, and produces clean, structured JSON objects ready for downstream workflow actions.

### Key Features

- **Real-Time News Stream:** Access current news headlines, articles, and community stories from Yabiladi.
- **Configurable Limit:** Set `maxItems` to control the number of articles retrieved.
- **Structured JSON Output:** Formats raw RSS XML entries into typed JSON objects containing titles, direct links, publication dates, summaries, authors, GUIDs, and category tags.
- **Automatic CDATA & XML Entity Decoding:** Cleans up CDATA sections and decodes standard and numeric XML entities.
- **Zero Authentication Required:** Operates directly with public RSS streams without API tokens or credentials.

### Use Cases

- **Moroccan & Diaspora News Monitoring:** Track national stories, government decisions, and diaspora-related developments in real time.
- **Automated Digest & Alerts:** Schedule regular morning or evening news bulletins and send them to Telegram, Slack, WhatsApp, or Email.
- **AI-Powered News Summarization:** Pass Yabiladi article summaries to LLM nodes (e.g., Anthropic, OpenAI, DeepSeek) for automated news digests and sentiment analysis.
- **Content Syndication:** Ingest and aggregate Moroccan news feeds for display on internal portals, dashboards, or mobile applications.

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
Specifies the maximum count of article objects returned in the `items` array.
- Default: `10`
- Example values: `5`, `10`, `25`
- If set to `0` or omitted, all items available in the RSS feed (up to the feed's native capacity) are returned.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming trigger or workflow event to initiate the feed retrieval. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when feed retrieval and parsing succeed. Returns the feed metadata and an array of parsed articles. |
| `error` | `Error` | Emitted when the network request or RSS parsing fails. |

---

### Output Data Structure Example

```json
{
  "title": "Yabiladi.com - Le premier portail du Maroc",
  "link": "https://www.yabiladi.com/",
  "description": "Toute l'actualité du Maroc et des Marocains du monde",
  "language": "fr",
  "lastBuildDate": "Wed, 26 Aug 2026 15:30:00 +0100",
  "pubDate": "Wed, 26 Aug 2026 15:30:00 +0100",
  "items": [
    {
      "title": "Maroc : Hausse des investissements directs étrangers au premier semestre",
      "link": "https://www.yabiladi.com/articles/details/154321/maroc-hausse-investissements-directs-etrangers.html",
      "description": "L'Office des Changes a rapporté une augmentation significative des flux nets d'investissements directs étrangers (IDE) au Maroc au cours des six premiers mois...",
      "pubDate": "Wed, 26 Aug 2026 14:15:00 +0100",
      "author": "Yabiladi",
      "guid": "https://www.yabiladi.com/articles/details/154321.html",
      "category": [
        "Economie",
        "Maroc"
      ]
    },
    {
      "title": "Marocains du monde : Facilitation des démarches administratives pour les vacances",
      "link": "https://www.yabiladi.com/articles/details/154322/marocains-monde-facilitation-demarches.html",
      "description": "De nouvelles mesures de simplification des procédures consulaires et douanières ont été déployées pour accompagner les MRE durant la période estivale...",
      "pubDate": "Wed, 26 Aug 2026 12:45:00 +0100",
      "author": "Yabiladi",
      "guid": "https://www.yabiladi.com/articles/details/154322.html",
      "category": [
        "Société",
        "MRE"
      ]
    }
  ]
}
```

---

### Field Reference

#### Top-Level Feed Object Fields

| Field | Type | Description |
|-------|------|-------------|
| `title` | `string` | Channel title of the Yabiladi RSS feed |
| `link` | `string` | Main website link (`https://www.yabiladi.com/`) |
| `description` | `string` | Description of the RSS feed |
| `language` | `string` | Feed language code (`fr`) |
| `lastBuildDate` | `string` | Timestamp when the feed was last updated |
| `pubDate` | `string` | Publication date of the feed |
| `items` | `array` | List of parsed article objects (limited to `maxItems`) |

#### Item Object Fields (`items[]`)

| Field | Type | Description |
|-------|------|-------------|
| `title` | `string` | Headline of the news article |
| `link` | `string` | Direct web URL to the full article on Yabiladi |
| `description` | `string` | Article summary with CDATA removed and XML entities decoded |
| `pubDate` | `string` | Article publication timestamp |
| `author` | `string` | Article author or publication source |
| `guid` | `string` | Unique identifier or canonical URL for the item |
| `category` | `string[]` | List of category tags assigned to the article |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Fetch Top 10 Latest News Articles

Retrieve the 10 most recent news stories from Yabiladi:

**Configuration:**

- **MaxItems:** `10`

---

### Example 2: Daily Morning News Bulletin

Fetch 5 top stories every morning at 08:30 AM and dispatch them as an email bulletin:

**Configuration:**

- **MaxItems:** `5`

**Workflow Pattern:**

```text
Cron Trigger (08:30 AM)
  → Yabiladi (maxItems: 5)
  → Function (format articles into an HTML newsletter)
  → Email Send
```

---

### Example 3: Filter MRE & Diaspora Stories

Poll Yabiladi and filter specifically for stories categorized under *MRE* or *Société*:

**Workflow Pattern:**

```text
Interval Trigger (every 1 hour)
  → Yabiladi (maxItems: 20)
  → Filter Array (where item.category includes 'MRE')
  → Telegram Bot Send
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Fetch news from Yabiladi RSS feed
```

### Common Patterns

- **Scheduled News Digest:** Cron Trigger → Yabiladi → Function (Markdown/HTML Formatter) → Slack / Email.
- **Multilingual Morocco News Hub:** Parallel branches of Yabiladi (French) + Hespress (Arabic) + Hespress English (English) → Merge → Database Storage.
- **AI-Powered Summary Bot:** Yabiladi → Function (Extract headlines) → AI Chat (Generate bullet-point summary) → Discord Bot Send.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Empty `items` array returned

**Cause:** The Yabiladi RSS feed server is experiencing temporary downtime or network disruption.

**Solution:** Verify that `https://www.yabiladi.com/rss/` is accessible and retry after a brief delay.

#### Special characters or accents appear corrupted

**Cause:** Downstream logging or processing nodes do not support UTF-8 character encoding.

**Solution:** Ensure all subsequent workflow nodes (e.g. database actions, email senders, webhook targets) handle UTF-8 text encoding properly.

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Yabiladi feed parsing failed: Failed to fetch RSS feed: <status> <statusText>` | Network connection error, DNS failure, or remote server returned an HTTP error | Check internet connection to `www.yabiladi.com` |
| `Yabiladi feed parsing failed: <message>` | Malformed XML response or parsing error | Retry after a few minutes |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Hespress](../hespress/en.md) — Fetch news from Hespress Arabic RSS feed
- [Hespress English](../hespress-english/en.md) — Fetch news from Hespress English RSS feed
- [MAP News](../map-news/en.md) — Fetch news from Maghreb Arabe Presse (MAP) RSS feeds
- [Le Matin](../le-matin/en.md) — Fetch news from Le Matin RSS feeds
- [HTTP Request](../http-request/en.md) — Custom HTTP requests and generic RSS polling
- [Filter Array](../filter-array/en.md) — Filter news items by keywords or categories
- [Function](../function/en.md) — Transform and format news articles
- [Log](../log/en.md) — Output and inspect news data in the workflow console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-26 | Initial release of Yabiladi RSS Action Node |

<!-- /SECTION: changelog -->

---
node_id: "map-news"
title: "MAP News"
description: "Fetch news from Maghreb Arabe Presse (MAP) RSS feeds by category and language."
category: "utilities"
subcategory: "news"
version: "1.0.0"
language: "en"
last_updated: "2026-08-26"
author: "Fusion Team"
tags:
  - map
  - map-news
  - maghreb-arabe-presse
  - morocco
  - maroc
  - news
  - rss
  - feed
  - arabic
  - french
  - english
related_nodes:
  - hespress
  - hespress-english
  - http-request
  - function
  - filter-array
  - log
---

<!-- SECTION: header -->
# MAP News (Maghreb Arabe Presse)

> **Category:** Utilities | **Type:** Action Node

Fetch the latest Moroccan and international news articles directly from official [Maghreb Arabe Presse (MAP)](https://www.mapnews.ma/) RSS feeds by category and language.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **MAP News** node connects to the official RSS service of **Maghreb Arabe Presse (MAP)**, Morocco's national news agency (`mapnews.ma`). It enables workflows to fetch real-time news across 8 official topical categories in Arabic, French, or English.

The node automatically retrieves the remote XML feed, parses all channel metadata and individual news items, strips CDATA wrappers, decodes XML entities, and outputs structured JSON ready for downstream workflow consumption.

### Key Features

- **Multilingual Support:** Fetch news feeds published in Arabic (`ar`), French (`fr`), or English (`en`).
- **8 Dedicated News Categories:** Query feeds for Politics, Economy, Social, Culture, Sport, Regional, General, and World.
- **Configurable Limit:** Set `maxItems` to control the number of articles returned.
- **Structured JSON Output:** Formats raw RSS XML feeds into a typed JSON object containing channel metadata and a list of parsed article entries.
- **Automatic XML Entity & CDATA Processing:** Cleans up CDATA blocks and decodes character entities (e.g. `&amp;`, `&quot;`, numeric/hex entities).
- **Keyless / Zero Authentication:** Access public MAP news streams directly without API keys or credentials.

### Use Cases

- **Multilingual Media Monitoring:** Monitor official government announcements, royal activities, and national reports in Arabic, French, or English.
- **Real-Time News Alerting:** Periodically poll breaking headlines and dispatch alerts to Slack, Telegram, WhatsApp, or Email.
- **AI-Powered Summarization & Sentiment:** Feed official news headlines into LLM nodes (e.g., Anthropic, OpenAI, DeepSeek) for daily intelligence briefs and sentiment analysis.
- **Content Syndication:** Aggregate official Moroccan news into internal dashboards, portals, and mobile applications.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `category` | `enum` | ❌ No | `"general"` | Select the MAP news category to fetch. |
| `language` | `enum` | ❌ No | `"en"` | Select the language for MAP news (`en`, `fr`, `ar`). |
| `maxItems` | `number` | ❌ No | `10` | Maximum number of news articles to retrieve from the feed. |

---

### Category Options

The node maps category selections to the official MAP news RSS feed endpoints:

| Category Value | Category Label | MAP RSS Path | Example URL Pattern |
|----------------|----------------|--------------|---------------------|
| `politics` | Politics | `politique` | `https://mapnews.ma/{language}/actualites/politique/rss` |
| `economy` | Economy | `economie` | `https://mapnews.ma/{language}/actualites/economie/rss` |
| `social` | Social | `social` | `https://mapnews.ma/{language}/actualites/social/rss` |
| `culture` | Culture | `culture` | `https://mapnews.ma/{language}/actualites/culture/rss` |
| `sport` | Sport | `sport` | `https://mapnews.ma/{language}/actualites/sport/rss` |
| `regional` | Regional | `regional` | `https://mapnews.ma/{language}/actualites/regional/rss` |
| `general` | General | `general` | `https://mapnews.ma/{language}/actualites/general/rss` |
| `world` | World | `monde` | `https://mapnews.ma/{language}/actualites/monde/rss` |

---

### Language Options

| Language Code | Language | Description |
|---------------|----------|-------------|
| `en` | English | English language edition of MAP News (`/en/`) |
| `fr` | French | French language edition of MAP News (`/fr/`) |
| `ar` | Arabic | Arabic language edition of MAP News (`/ar/`) |

---

### Parameter Details

#### `category`
Specifies the topical section to retrieve. Default is `general`.

#### `language`
Specifies the language edition of the RSS feed. Default is `en`.

#### `maxItems`
Specifies the upper bound of news article objects returned in the `items` array.
- Default: `10`
- Example values: `5`, `10`, `25`
- If set to `0` or omitted, all items returned by the feed are preserved (up to the feed's native capacity).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming trigger or workflow event to initiate feed retrieval. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when feed retrieval and parsing succeed. Contains the feed metadata and an array of parsed articles. |
| `error` | `Error` | Emitted when the network request or RSS parsing fails. |

---

### Output Data Structure

The `success` output emits a structured RSS feed object:

```json
{
  "title": "Actualités Générales - MAP",
  "link": "https://mapnews.ma/fr/actualites/general",
  "description": "Fil d'actualité générale de la MAP",
  "language": "fr",
  "lastBuildDate": "Wed, 26 Aug 2026 10:00:00 +0100",
  "pubDate": "Wed, 26 Aug 2026 10:00:00 +0100",
  "items": [
    {
      "title": "Maroc - Signature de plusieurs accords bilatéraux stratégiques à Rabat",
      "link": "https://mapnews.ma/fr/actualites/politique/signature-accords-bilateraux",
      "description": "Rabat - Plusieurs accords bilatéraux ont été signés aujourd'hui à Rabat visant à renforcer la coopération économique et technologique...",
      "pubDate": "Wed, 26 Aug 2026 09:30:00 +0100",
      "author": "MAP",
      "guid": "https://mapnews.ma/fr/actualites/politique/signature-accords-bilateraux",
      "category": [
        "Politique",
        "Maroc"
      ]
    },
    {
      "title": "Performance positive des exportations nationales au premier semestre",
      "link": "https://mapnews.ma/fr/actualites/economie/performance-exportations-semestre",
      "description": "Casablanca - L'Office des Changes a publié les indicateurs mensuels des échanges extérieurs reflétant une progression notable des exportations...",
      "pubDate": "Wed, 26 Aug 2026 08:45:00 +0100",
      "author": "MAP",
      "guid": "https://mapnews.ma/fr/actualites/economie/performance-exportations-semestre",
      "category": [
        "Economie"
      ]
    }
  ]
}
```

#### Top-Level Feed Object Fields

| Field | Type | Description |
|-------|------|-------------|
| `title` | `string` | Title of the MAP RSS channel (if provided) |
| `link` | `string` | Main website link associated with the feed |
| `description` | `string` | Channel description |
| `language` | `string` | Language code declared in the RSS feed |
| `lastBuildDate` | `string` | Timestamp when the RSS feed was last generated |
| `pubDate` | `string` | Feed publication date |
| `items` | `array` | List of parsed article objects (up to `maxItems`) |

#### Item Object Fields (`items[]`)

| Field | Type | Description |
|-------|------|-------------|
| `title` | `string` | Headline of the news article |
| `link` | `string` | Direct web URL to the full article on MAP News |
| `description` | `string` | Article summary / snippet with CDATA and XML entities decoded |
| `pubDate` | `string` | Article publication timestamp |
| `author` | `string` | Author or news agency identifier (`MAP`) |
| `guid` | `string` | Unique identifier or canonical URL for the article |
| `category` | `string[]` | Array of category tags assigned to the article |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Fetch Latest 10 Arabic Politics Articles

Fetch the 10 most recent political news stories published in Arabic:

**Configuration:**

- **Category:** `politics`
- **Language:** `ar`
- **MaxItems:** `10`

---

### Example 2: Daily Morning Economy Briefing in French

Fetch 5 top economy news stories every morning at 08:30 AM and format them for an executive email:

**Configuration:**

- **Category:** `economy`
- **Language:** `fr`
- **MaxItems:** `5`

**Workflow Pattern:**

```text
Cron Trigger (08:30 AM)
  → MAP News (category: "economy", language: "fr", maxItems: 5)
  → Function (format articles into HTML email bulletin)
  → Email Send
```

---

### Example 3: English General News Digest for Slack / Telegram

Poll breaking Moroccan general news in English and send instant alerts to a communications channel:

**Configuration:**

- **Category:** `general`
- **Language:** `en`
- **MaxItems:** `10`

**Workflow Pattern:**

```text
Interval Trigger (every 30 mins)
  → MAP News (category: "general", language: "en", maxItems: 10)
  → Deduplicate / Filter (new articles only)
  → Slack / Telegram Bot Send
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Fetch news from MAP News RSS feed
```

### Common Patterns

- **Scheduled News Digest:** Cron Trigger → MAP News → Function (HTML/Markdown Format) → Email Send / Slack.
- **Multilingual News Aggregator:** Fetch MAP News in Arabic and French in parallel → Merge → Database Storage.
- **AI Intelligence & Summary Pipeline:** MAP News → Function (extract headlines) → AI Chat / LLM (generate executive bullet points) → Notification.
- **Category Filter & Routing:** MAP News → Filter Array (filter by specific tag or keyword in `title`/`description`) → Regional Desk Webhook.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Empty `items` array returned

**Cause:** The MAP News server returned a feed with no current items for the selected category/language, or the remote endpoint timed out.

**Solution:** Verify that the chosen category and language combination exists on `mapnews.ma`. Test another category (e.g. `general`) or retry after a brief delay.

#### Arabic characters display incorrectly in downstream systems

**Cause:** Downstream consumers (loggers, database connections, or messaging nodes) might not have UTF-8 encoding enabled.

**Solution:** Ensure all subsequent nodes handle UTF-8 text encoding. The MAP News node parses and returns standard UTF-8 text strings.

#### Network or DNS fetch failure

**Cause:** Temporary network disruption or connection timeout connecting to `mapnews.ma`.

**Solution:** Verify internet connectivity to HTTPS port 443 on `mapnews.ma`.

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Invalid category: <category>` | The configured category is not among the supported enum values | Select a valid category from the dropdown (`politics`, `economy`, `social`, `culture`, `sport`, `regional`, `general`, `world`) |
| `MAP news feed parsing failed: Failed to fetch RSS feed: 404 Not Found` | The specified category or language path does not exist on the MAP server | Check category and language combination |
| `MAP news feed parsing failed: <message>` | Network timeout, DNS failure, or malformed XML response | Check connectivity to `https://mapnews.ma` and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Hespress](../hespress/en.md) — Fetch news from Hespress Arabic RSS feed
- [Hespress English](../hespress-english/en.md) — Fetch news from Hespress English RSS feed
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
| 1.0.0 | 2026-08-26 | Initial release of MAP News RSS Action Node |

<!-- /SECTION: changelog -->

---
node_id: "al-sabah"
title: "Al Sabah RSS Feed"
description: "Fetch the latest news articles from the Al Sabah (Assabah) Moroccan news portal RSS feed, optionally filtered by category."
category: "RSS"
subcategory: "Moroccan News"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - al-sabah
  - assabah
  - rss
  - news
  - morocco
  - media
related_nodes:
  - le-matin
  - hespress
  - telquel
  - rss-feed
---

<!-- SECTION: overview -->
# Al Sabah RSS Feed

> **Category:** RSS &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Fetch the latest news articles published by **Al Sabah** (`https://assabah.ma`), a Moroccan Arabic-language daily, through its public RSS feed. The node can read the main feed or the feed of a single category, and returns each article as a clean JSON object (title, link, date, author, description, categories).

No API key is required.

### Use Cases

- **News monitoring:** Track Moroccan news on a topic and forward new articles to Slack, Teams or email.
- **Press review:** Build a daily digest of Al Sabah headlines alongside other Moroccan sources.
- **Content analysis:** Send article titles and descriptions to an LLM for summarization, translation or classification.
- **Archiving:** Store articles in a database or spreadsheet for later analysis.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `limit` | `number` | No | `10` | Maximum number of articles to fetch. |
| `category` | `string` | No | — | Category slug of the feed (e.g. `news`, `sports`, `politics`). Leave empty to read the main feed. |

### Feed URLs

| `category` | Feed URL |
|------------|----------|
| *(empty)* | `https://assabah.ma/feed` |
| `<slug>` | `https://assabah.ma/category/<slug>/feed` |

> The category value must match the slug used in the website URL (`https://assabah.ma/category/<slug>`). Open a category page on assabah.ma and copy the last part of its URL.

### Authentication

No authentication is needed. The feed is public.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Feed information and the list of articles. Contains `success: true`, or `success: false` with an `error` message. |
| `error` | `Error` | Emitted on uncaught runtime errors. |

> Fetch and parsing errors do **not** go to the `error` output: they are returned on `success` with `success: false`. Use an **If** node on `success` to branch on failures.

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | `true` when the feed was read. |
| `source` | `string` | Always `al-sabah`. |
| `feedTitle` | `string` | Title of the feed. |
| `feedDescription` | `string` | Description of the feed. |
| `feedLink` | `string` | Website link of the feed. |
| `articles` | `array` | Articles: `title`, `link`, `pubDate`, `author`, `description`, `categories`, `guid`. |
| `count` | `number` | Number of articles returned. |

### Output Examples

#### Successful Fetch

```json
{
  "success": true,
  "source": "al-sabah",
  "feedTitle": "الصباح",
  "feedDescription": "جريدة الصباح المغربية",
  "feedLink": "https://assabah.ma",
  "articles": [
    {
      "title": "عنوان المقال",
      "link": "https://assabah.ma/123456.html",
      "pubDate": "Tue, 17 Nov 2026 08:30:00 +0000",
      "author": "الصباح",
      "description": "ملخص قصير للمقال...",
      "categories": ["وطنية"],
      "guid": "https://assabah.ma/?p=123456"
    }
  ],
  "count": 1
}
```

#### Fetch Error

```json
{
  "success": false,
  "error": "Failed to fetch RSS feed: 404 Not Found",
  "source": "al-sabah"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Read the latest Al Sabah articles
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Al Sabah RSS Feed Node:** Downloads and parses the RSS feed.
3. **Log Node:** Displays the list of articles.

### Example Configuration

```json
{
  "limit": 5,
  "category": ""
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Failed to fetch RSS feed: 404 Not Found`
- **Cause:** The `category` slug does not exist on assabah.ma.
- **Solution:** Copy the exact slug from the category page URL, or leave `category` empty to use the main feed.

#### `Failed to fetch RSS feed: 403 Forbidden` / `5xx`
- **Cause:** The website is blocking the request or is temporarily unavailable.
- **Solution:** Retry later and avoid very frequent schedules.

#### `success: false` with a parsing error
- **Cause:** The URL returned something other than a valid RSS feed (e.g. an HTML error page).
- **Solution:** Open the feed URL in a browser to check that it returns RSS XML.

#### Fewer articles than `limit`
- **Cause:** The feed only exposes its most recent items.
- **Solution:** This is expected; run the workflow on a schedule to collect articles over time.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation: parameter reference, feed URLs, output schema, and workflow integration |

<!-- /SECTION: changelog -->

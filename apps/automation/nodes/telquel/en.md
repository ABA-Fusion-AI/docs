---
node_id: "telquel"
title: "TelQuel"
description: "Fetch news from TelQuel RSS feed."
category: "RSS"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:

- rss
- telquel
- news
- morocco
- maroc
- articles
- rss-feed
- media

related_nodes:
- le-matin
- function
- if
- http-request

---

# TelQuel

> **Category:** rss-nodes | **Type:** Action Node

Fetch news articles from **TelQuel**'s single fixed RSS feed (`telquel.ma/feed`).

The **TelQuel** node retrieves TelQuel's RSS feed, parses the XML response via regex-based extraction, and returns the results as structured workflow data. Unlike [Le Matin](./le-matin.md), this node has a **single, fixed feed URL** with no category selection.

### Supported Features

- Fetch the TelQuel RSS feed (fixed URL, no category options)
- Parse RSS XML via regex extraction
- Extract feed metadata (title, link, description, language, dates)
- Extract article fields (title, link, description, pubDate, author, guid, categories)
- Limit the number of returned articles using `maxItems`
- Decode XML entities and remove CDATA wrappers
- Support workflow-based news monitoring

### Use Cases

- Monitor TelQuel news
- Build a Moroccan news aggregation workflow alongside other RSS sources
- Send new articles to notifications
- Store articles in a database
- Filter articles using an `If` node
- Transform articles using a `Function` node

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `maxItems` | `number` | ❌ No | `10` | Maximum number of RSS articles to return. |

There is no `category` parameter — TelQuel's feed is a single, fixed URL:

```text
https://telquel.ma/feed
```

---

## Inputs & Outputs

### Inputs

The node does not require workflow input.

All configuration is provided through the node configuration.

### Outputs

The node returns an RSS feed object containing feed metadata and an array of articles.

| Output | Type | Description |
| ------ | ---- | ----------- |
| `title` | `string` | RSS feed title, when available. |
| `link` | `string` | RSS feed link, when available. |
| `description` | `string` | RSS feed description, when available. |
| `language` | `string` | RSS feed language, when available. |
| `lastBuildDate` | `string` | Date when the feed was last built, when available. |
| `pubDate` | `string` | Feed publication date, when available. |
| `items` | `array` | List of RSS articles. |

### RSS Item Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `title` | `string` | Article title. |
| `link` | `string` | Article URL. |
| `description` | `string` | Article description or summary. |
| `pubDate` | `string` | Article publication date. |
| `author` | `string` | Article author, when available. |
| `guid` | `string` | Unique RSS item identifier. |
| `category` | `string[]` | Article categories, only present if at least one `<category>` tag exists. |

---

## Output Example

```json
{
  "title": "TelQuel.ma",
  "link": "https://telquel.ma",
  "description": "TelQuel - Actualités du Maroc",
  "language": "fr-FR",
  "lastBuildDate": "Mon, 24 Aug 2026 10:00:00 GMT",
  "pubDate": "Mon, 24 Aug 2026 09:45:00 GMT",
  "items": [
    {
      "title": "Example TelQuel Article",
      "link": "https://telquel.ma/2026/08/24/example-article",
      "description": "Article description text.",
      "pubDate": "Mon, 24 Aug 2026 09:45:00 GMT",
      "author": "TelQuel",
      "guid": "https://telquel.ma/?p=123456",
      "category": [
        "Politique",
        "Maroc"
      ]
    }
  ]
}
```

---

## Configuration Examples

### Default Configuration

Returns up to 10 articles.

```json
{
  "maxItems": 10
}
```

### More Articles

```json
{
  "maxItems": 30
}
```

### Return All Available Articles

Set `maxItems` to `0` or another non-positive value.

```json
{
  "maxItems": 0
}
```

---

## Workflow Integration

### Sample Workflow: Fetch Latest News

```json
{
  "nodes": [
    {
      "id": "telquel",
      "type": "telquel",
      "config": {
        "maxItems": 10
      }
    }
  ]
}
```

### Sample Workflow: RSS → Function

```json
{
  "nodes": [
    {
      "id": "telquel",
      "type": "telquel",
      "config": {
        "maxItems": 10
      }
    },
    {
      "id": "process-news",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: RSS → If

```json
{
  "nodes": [
    {
      "id": "telquel",
      "type": "telquel",
      "config": {
        "maxItems": 10
      }
    },
    {
      "id": "filter-news",
      "type": "if"
    }
  ]
}
```

### Sample Workflow: Multi-Source Aggregation

```json
{
  "nodes": [
    {
      "id": "telquel",
      "type": "telquel"
    },
    {
      "id": "le-matin",
      "type": "le-matin",
      "config": {
        "category": "latest"
      }
    },
    {
      "id": "merge-feeds",
      "type": "function"
    }
  ]
}
```

### Common Patterns

- Schedule → TelQuel → Process Articles
- TelQuel → Function → Transform Articles
- TelQuel → If → Filter Articles
- TelQuel → Database → Store Articles
- TelQuel → Notification → Send News
- TelQuel + Le Matin → Function → Merge Feeds — multi-source Moroccan news aggregation

---

## RSS Parsing

The node parses the RSS XML response and extracts channel-level elements:

- `title`
- `link`
- `description`
- `language`
- `lastBuildDate`
- `pubDate`

For each `<item>`, the node extracts:

- `title`
- `link`
- `description`
- `pubDate`
- `author`
- `guid`
- `category` (all `<category>` tags collected into an array)

All extraction is done via **regex matching** against the raw XML text, not a full XML parser — see [Notes](#notes) for the implications of this approach.

---

## XML Processing

### CDATA Handling

RSS feeds may contain CDATA sections such as:

```xml
<description><![CDATA[Article description]]></description>
```

The node removes the CDATA wrapper before returning the content.

### XML Entity Decoding

The parser decodes common XML entities:

| Entity | Result |
| ------ | ------ |
| `&lt;` | `<` |
| `&gt;` | `>` |
| `&amp;` | `&` |
| `&quot;` | `"` |
| `&apos;` | `'` |

It also supports numeric decimal entities (`&#233;`) and hexadecimal entities (`&#xE9;`).

---

## Item Limit

The `maxItems` parameter controls how many articles are returned.

When `maxItems` is greater than `0` and the feed contains more articles than the configured limit, only the first `maxItems` articles (in feed order) are returned.

When `maxItems` is `0` or another non-positive value, the node does not apply an item limit — all items in the feed are returned.

---

## Error Handling

If fetching or parsing the RSS feed fails, the node throws an error.

The error format is:

```text
TelQuel feed parsing failed: <error message>
```

### RSS Feed Fetch Failure

If the feed cannot be retrieved:

```text
TelQuel feed parsing failed: Failed to fetch RSS feed: <status> <statusText>
```

---

## Troubleshooting

### RSS Feed Fetch Failed

**Cause**

The TelQuel RSS feed (`telquel.ma/feed`) could not be retrieved.

**Solution**

Verify the feed is reachable directly in a browser. The node sends:

```text
User-Agent: Mozilla/5.0 (compatible; RSS Feed Parser)
```

If TelQuel starts blocking this generic user agent, the request may need to be updated.

---

### HTTP Error

**Cause**

The RSS server returned a non-success HTTP status code.

**Solution**

Check that `telquel.ma/feed` is currently available; the error reports the status via:

```text
Failed to fetch RSS feed: <status> <statusText>
```

---

### Missing or Empty Fields

**Cause**

The regex-based parser only extracts a field if its exact tag pattern is present in the XML — feed elements TelQuel doesn't include (e.g. no `<author>` tag on some items) will simply be omitted from the item rather than appearing as an empty string.

**Solution**

Treat all item fields as optional downstream; check for presence before relying on a field like `author` or `category`.

---

### Feed Structure Changes Break Parsing

**Cause**

Since parsing relies on **regex patterns matching specific tag names**, not a real XML parser, any structural change to TelQuel's feed (e.g. namespaced tags like `<dc:creator>` instead of `<author>`, or nested/attribute-based content) would not be picked up by the current patterns.

**Solution**

If fields start returning consistently empty despite the feed clearly containing data (verify by fetching the raw XML), TelQuel's feed structure has likely changed and the regex patterns would need to be updated.

---

### No Articles Returned

**Cause**

The selected RSS feed contains no `<item>` elements, or no article entries are currently published.

**Solution**

Verify the feed currently contains articles by checking `telquel.ma/feed` directly.

---

## Security

The node performs outbound HTTP requests to a single, fixed TelQuel RSS endpoint (`https://telquel.ma/feed`).

No API key or authentication credential is required.

The feed URL is fixed and not user-configurable, which prevents the node from being used as a generic arbitrary URL fetcher.

---

## Notes

The node returns parsed RSS data rather than the raw XML response.

Unlike a proper XML parser, this node uses **regex-based extraction** — a pragmatic but fragile approach. It works well for well-formed, simple RSS feeds but can behave unexpectedly with:

- Nested tags sharing a name with a top-level field (regex is not scope-aware)
- Self-closing or attribute-heavy tags
- Namespaced elements (e.g. `<dc:creator>`, `<content:encoded>`) — these are not extracted at all
- Malformed or non-standard XML that a strict parser would reject but regex may partially match

The node does not:

- Publish articles
- Modify TelQuel content
- Download article pages or media files
- Generate summaries or translate articles
- Store articles or generate embeddings
- Support a category or feed-URL parameter (single fixed feed only)
- Verify the truth of news content

It is intended to retrieve and structure TelQuel RSS content for downstream workflow processing.

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-24 | Initial release |
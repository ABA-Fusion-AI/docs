---
node_id: "agadir24"
title: "Agadir 24"
description: "Fetch news from Agadir 24 RSS feed."
category: "RSS"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - rss
  - agadir24
  - news
  - agadir
  - morocco
  - rss-feed
  - articles
  - action
related_nodes:
  - function
  - if
  - http-request

---

# Agadir 24

> **Category:** rss-nodes | **Type:** Action Node

Fetch news articles from the **Agadir 24 RSS feed**.

The **Agadir 24** node retrieves the RSS feed from `agadir24.info`, parses the XML response, extracts feed metadata and article information, and returns the results as structured workflow data.

The node supports limiting the number of returned articles using the `maxItems` parameter.

### Supported Features

- Fetch the Agadir 24 RSS feed
- Parse RSS XML
- Extract feed metadata
- Extract article titles
- Extract article links
- Extract article descriptions
- Extract publication dates
- Extract authors
- Extract article GUIDs
- Extract article categories
- Limit the number of returned articles
- Decode XML entities and CDATA sections

### Use Cases

- Monitor Agadir 24 news
- Build a Moroccan news aggregation workflow
- Send new articles to notifications
- Store news articles in a database
- Filter articles using an `If` node
- Transform articles using a `Function` node
- Build automated news monitoring workflows

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `maxItems` | `number` | ❌ No | `10` | Maximum number of RSS articles to return. |

### Feed URL

The node uses the following RSS feed:

~~~text
https://agadir24.info/feed
~~~

The feed URL is defined internally by the node and is not exposed as a configuration parameter.

---

## Inputs & Outputs

### Inputs

This node does not require workflow input.

All configuration is provided through the node configuration.

### Outputs

The node returns an RSS feed object containing feed metadata and an array of articles.

| Output | Type | Description |
| ------ | ---- | ----------- |
| `title` | `string` | RSS feed title, when available. |
| `link` | `string` | RSS feed URL, when available. |
| `description` | `string` | RSS feed description, when available. |
| `language` | `string` | RSS feed language, when available. |
| `lastBuildDate` | `string` | Date when the feed was last built, when available. |
| `pubDate` | `string` | Feed publication date, when available. |
| `items` | `array` | List of RSS articles. |

### RSS Item Fields

Each RSS item may contain the following fields:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `title` | `string` | Article title. |
| `link` | `string` | Article URL. |
| `description` | `string` | Article description or summary. |
| `pubDate` | `string` | Article publication date. |
| `author` | `string` | Article author, when available. |
| `guid` | `string` | Unique RSS item identifier. |
| `category` | `string[]` | List of article categories. |

Additional RSS fields are also preserved when supported by the parser.

### Output Example

~~~json
{
  "title": "Agadir 24",
  "link": "https://agadir24.info",
  "description": "Agadir 24 RSS Feed",
  "language": "ar",
  "lastBuildDate": "Mon, 10 Aug 2026 14:00:00 GMT",
  "pubDate": "Mon, 10 Aug 2026 14:00:00 GMT",
  "items": [
    {
      "title": "Example News Article",
      "link": "https://agadir24.info/example-news",
      "description": "Article description.",
      "pubDate": "Mon, 10 Aug 2026 13:30:00 GMT",
      "author": "Agadir 24",
      "guid": "https://agadir24.info/example-news",
      "category": [
        "أكادير",
        "أخبار المغرب"
      ]
    }
  ]
}
~~~

The exact feed metadata and article fields depend on the content returned by the Agadir 24 RSS feed.

---

## Configuration Examples

### Default Configuration

Uses the default `maxItems` value of `10`.

~~~json
{
  "maxItems": 10
}
~~~

### Return 5 Articles

~~~json
{
  "maxItems": 5
}
~~~

### Return All Articles

Set `maxItems` to `0` or another non-positive value to disable the item limit.

~~~json
{
  "maxItems": 0
}
~~~

---

## Workflow Integration

### Sample Workflow: Fetch Latest News

~~~json
{
  "nodes": [
    {
      "id": "agadir24",
      "type": "agadir24",
      "config": {
        "maxItems": 10
      }
    }
  ]
}
~~~

### Sample Workflow: RSS → Function

~~~json
{
  "nodes": [
    {
      "id": "agadir24",
      "type": "agadir24",
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
~~~

### Sample Workflow: RSS → If

~~~json
{
  "nodes": [
    {
      "id": "agadir24",
      "type": "agadir24",
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
~~~

### Common Patterns

- Schedule → Agadir 24 → Process Articles
- Agadir 24 → Function → Transform Articles
- Agadir 24 → If → Filter Articles
- Agadir 24 → Database → Store Articles
- Agadir 24 → Notification → Send News
- Agadir 24 → HTTP Request → Process Article URLs

---

## RSS Parsing

The node parses the RSS XML response and extracts the following channel-level elements:

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
- `category`

Multiple `<category>` elements are collected into the `category` array.

The parser also removes CDATA wrappers and decodes common XML entities.

---

## Item Limit

The `maxItems` parameter controls how many articles are returned.

When `maxItems` is greater than `0` and the feed contains more articles than the configured limit, the node returns only the first `maxItems` articles.

For example:

~~~json
{
  "maxItems": 5
}
~~~

If the RSS feed contains 20 articles, only the first 5 articles are returned.

When `maxItems` is `0` or another non-positive value, the node does not apply an item limit.

---

## Troubleshooting

### RSS Feed Fetch Failed

**Cause**

The Agadir 24 RSS feed could not be retrieved.

**Solution**

Verify that the RSS feed is available:

~~~text
https://agadir24.info/feed
~~~

The node sends the following `User-Agent` header:

~~~text
Mozilla/5.0 (compatible; RSS Feed Parser)
~~~

---

### HTTP Error

**Cause**

The RSS server returned a non-success HTTP status code.

**Solution**

Check the feed availability and HTTP status returned by the server.

The node reports the error using the following format:

~~~text
Agadir 24 feed parsing failed: Failed to fetch RSS feed: <status> <statusText>
~~~

---

### Invalid RSS/XML

**Cause**

The server returned content that cannot be parsed as the expected RSS structure.

**Solution**

Verify that the configured Agadir 24 feed is returning valid RSS XML.

---

### No Articles Returned

**Cause**

The RSS feed contains no `<item>` elements or the feed did not provide article entries.

**Solution**

Check the feed directly and verify that articles are currently available.

---

### Too Many Articles

**Cause**

The workflow only returns the configured number of articles.

**Solution**

Increase `maxItems` or set it to `0` to disable the limit.

Example:

~~~json
{
  "maxItems": 25
}
~~~

---

## Error Handling

If fetching or parsing the RSS feed fails, the node throws an error.

The error format is:

~~~text
Agadir 24 feed parsing failed: <error message>
~~~

This allows downstream workflow error handling to detect failures from the Agadir 24 node.

---

## Feed URL

The Agadir 24 RSS feed used by this node is:

~~~text
https://agadir24.info/feed
~~~

The URL is hardcoded in the node implementation and is not configurable through the node schema.

---

## Related

- [Function](./function.md) – Transform and process RSS articles
- [If](./if.md) – Filter and route articles based on conditions
- [HTTP Request](./http-request.md) – Make requests to external services

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-10 | Initial release |
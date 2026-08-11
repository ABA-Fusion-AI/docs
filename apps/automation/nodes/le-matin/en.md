---
node_id: "le-matin"
title: "Le Matin"
description: "Fetch news from Le Matin RSS feeds by category."
category: "RSS"
version: "1.0.0"
language: "en"
last_updated: "2026-08-11"
author: "Fusion Team"
tags:

- rss
- le-matin
- lemation
- news
- morocco
- maroc
- articles
- rss-feed
- sports
- economy
- world
- society
- culture
- education
- action

related_nodes:
- function
- if
- http-request

---

# Le Matin

> **Category:** rss-nodes | **Type:** Action Node

Fetch news articles from **Le Matin** RSS feeds by category.

The **Le Matin** node retrieves an RSS feed from `lematin.ma`, parses the XML response, extracts feed metadata and article information, and returns the results as structured workflow data.

The node supports multiple Le Matin news categories and allows limiting the number of returned articles using the `maxItems` parameter.

### Supported Features

- Fetch Le Matin RSS feeds
- Select a news category
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
- Decode XML entities
- Remove CDATA wrappers
- Support workflow-based news monitoring

### Use Cases

- Monitor Moroccan news
- Monitor a specific Le Matin category
- Build a Moroccan news aggregation workflow
- Build sports news monitoring
- Monitor economy and business news
- Send new articles to notifications
- Store articles in a database
- Filter articles using an `If` node
- Transform articles using a `Function` node
- Build automated news monitoring workflows

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `category` | `enum` | ❌ No | `"latest"` | Le Matin RSS category to fetch. |
| `maxItems` | `number` | ❌ No | `10` | Maximum number of RSS articles to return. |

### Category Values

| Value | RSS Feed ID | Description |
| ----- | ----------- | ----------- |
| `latest` | `0` | Latest news. |
| `regions` | `2` | Regional news. |
| `sports` | `3` | Sports news. |
| `economy` | `4` | Economy news. |
| `world` | `5` | World news. |
| `nation` | `7` | National news. |
| `society` | `9` | Society news. |
| `tv` | `14` | TV and television content. |
| `editorials` | `25` | Editorial content. |
| `royal_activities` | `26` | Royal activities. |
| `automobile` | `27` | Automobile news. |
| `employment` | `28` | Employment and jobs. |
| `opinions` | `29` | Opinions. |
| `agenda` | `30` | Agenda and events. |
| `lifestyle` | `31` | Lifestyle. |
| `hi_tech` | `32` | High-tech and technology. |
| `specials` | `33` | Special features. |
| `royal_visits` | `60` | Royal visits. |
| `mtf` | `64` | MTF category. |
| `education` | `65` | Education. |
| `siam` | `72` | SIAM category. |
| `africa_development` | `74` | Africa development. |
| `culture` | `84` | Culture. |
| `chronicles` | `126` | Chronicles. |

---

## Feed URLs

The node internally maps each category to a Le Matin RSS feed.

Examples:

```text
https://lematin.ma/rssFeed/0
https://lematin.ma/rssFeed/3
https://lematin.ma/rssFeed/4
https://lematin.ma/rssFeed/5
```

The feed URL is selected internally from the `category` parameter and is not exposed as a direct configuration field.

### Complete Category Mapping

```typescript
{
  latest: "https://lematin.ma/rssFeed/0",
  regions: "https://lematin.ma/rssFeed/2",
  sports: "https://lematin.ma/rssFeed/3",
  economy: "https://lematin.ma/rssFeed/4",
  world: "https://lematin.ma/rssFeed/5",
  nation: "https://lematin.ma/rssFeed/7",
  society: "https://lematin.ma/rssFeed/9",
  tv: "https://lematin.ma/rssFeed/14",
  editorials: "https://lematin.ma/rssFeed/25",
  royal_activities: "https://lematin.ma/rssFeed/26",
  automobile: "https://lematin.ma/rssFeed/27",
  employment: "https://lematin.ma/rssFeed/28",
  opinions: "https://lematin.ma/rssFeed/29",
  agenda: "https://lematin.ma/rssFeed/30",
  lifestyle: "https://lematin.ma/rssFeed/31",
  hi_tech: "https://lematin.ma/rssFeed/32",
  specials: "https://lematin.ma/rssFeed/33",
  royal_visits: "https://lematin.ma/rssFeed/60",
  mtf: "https://lematin.ma/rssFeed/64",
  education: "https://lematin.ma/rssFeed/65",
  siam: "https://lematin.ma/rssFeed/72",
  africa_development: "https://lematin.ma/rssFeed/74",
  culture: "https://lematin.ma/rssFeed/84",
  chronicles: "https://lematin.ma/rssFeed/126"
}
```

---

## Inputs & Outputs

### Inputs

The node does not require workflow input.

All configuration is provided through the node configuration.

The node does not use incoming workflow data to select the category or item limit.

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

Each RSS item may contain:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `title` | `string` | Article title. |
| `link` | `string` | Article URL. |
| `description` | `string` | Article description or summary. |
| `pubDate` | `string` | Article publication date. |
| `author` | `string` | Article author, when available. |
| `guid` | `string` | Unique RSS item identifier. |
| `category` | `string[]` | Article categories. |

The parser is designed to preserve the standard RSS article fields extracted by the node.

---

## Output Example

```json
{
  "title": "Le Matin",
  "link": "https://lematin.ma",
  "description": "Le Matin RSS Feed",
  "language": "fr",
  "lastBuildDate": "Mon, 10 Aug 2026 14:00:00 GMT",
  "pubDate": "Mon, 10 Aug 2026 14:00:00 GMT",
  "items": [
    {
      "title": "Example Moroccan News Article",
      "link": "https://lematin.ma/example-news",
      "description": "Article description.",
      "pubDate": "Mon, 10 Aug 2026 13:30:00 GMT",
      "author": "Le Matin",
      "guid": "https://lematin.ma/example-news",
      "category": [
        "Morocco",
        "News"
      ]
    }
  ]
}
```

The exact feed metadata, language, article fields, and values depend on the selected Le Matin RSS feed.

---

## Configuration Examples

### Default Configuration

Uses the latest news feed and returns up to 10 articles.

```json
{
  "category": "latest",
  "maxItems": 10
}
```

### Sports News

```json
{
  "category": "sports",
  "maxItems": 10
}
```

### Economy News

```json
{
  "category": "economy",
  "maxItems": 20
}
```

### World News

```json
{
  "category": "world",
  "maxItems": 5
}
```

### Technology News

```json
{
  "category": "hi_tech",
  "maxItems": 15
}
```

### Culture News

```json
{
  "category": "culture",
  "maxItems": 10
}
```

### Return All Available Articles

Set `maxItems` to `0` or another non-positive value.

```json
{
  "category": "latest",
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
      "id": "le-matin",
      "type": "le-matin",
      "config": {
        "category": "latest",
        "maxItems": 10
      }
    }
  ]
}
```

### Sample Workflow: Fetch Sports News

```json
{
  "nodes": [
    {
      "id": "le-matin-sports",
      "type": "le-matin",
      "config": {
        "category": "sports",
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
      "id": "le-matin",
      "type": "le-matin",
      "config": {
        "category": "latest",
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
      "id": "le-matin",
      "type": "le-matin",
      "config": {
        "category": "economy",
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

### Common Patterns

- Schedule → Le Matin → Process Articles
- Le Matin → Function → Transform Articles
- Le Matin → If → Filter Articles
- Le Matin → Database → Store Articles
- Le Matin → Notification → Send News
- Le Matin → HTTP Request → Process Article URLs
- Le Matin → RAG Pipeline → Index News

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
- `category`

Multiple `<category>` elements are collected into the `category` array.

For example:

```xml
<category>Morocco</category>
<category>Politics</category>
```

becomes:

```json
{
  "category": [
    "Morocco",
    "Politics"
  ]
}
```

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

It also supports numeric decimal entities:

```text
&#233;
```

and hexadecimal entities:

```text
&#xE9;
```

---

## Item Limit

The `maxItems` parameter controls how many articles are returned.

When `maxItems` is greater than `0` and the feed contains more articles than the configured limit, only the first `maxItems` articles are returned.

Example:

```json
{
  "category": "latest",
  "maxItems": 5
}
```

If the feed contains 20 articles, the node returns only the first 5.

When `maxItems` is `0` or another non-positive value, the node does not apply an item limit.

---

## Category Selection

The selected `category` determines the RSS endpoint.

For example:

```json
{
  "category": "sports"
}
```

selects:

```text
https://lematin.ma/rssFeed/3
```

while:

```json
{
  "category": "economy"
}
```

selects:

```text
https://lematin.ma/rssFeed/4
```

The node rejects categories that are not present in the configured category mapping.

---

## Error Handling

If fetching or parsing the RSS feed fails, the node throws an error.

The error format is:

```text
Le Matin news feed parsing failed: <error message>
```

### Invalid Category

If the configured category does not exist:

```text
Invalid category: <category>
```

### RSS Feed Fetch Failure

If the feed cannot be retrieved:

```text
Le Matin news feed parsing failed: Failed to fetch RSS feed: <status> <statusText>
```

### HTTP Error

If Le Matin returns a non-success HTTP status code, the status and status text are included in the error.

---

## Troubleshooting

### RSS Feed Fetch Failed

**Cause**

The selected Le Matin RSS feed could not be retrieved.

**Solution**

Verify that the selected RSS endpoint is available.

For example:

```text
https://lematin.ma/rssFeed/0
```

The node sends:

```text
User-Agent: Mozilla/5.0 (compatible; RSS Feed Parser)
```

---

### HTTP Error

**Cause**

The RSS server returned a non-success HTTP status code.

**Solution**

Check the selected feed URL and verify that the Le Matin RSS service is available.

The node reports the error using:

```text
Failed to fetch RSS feed: <status> <statusText>
```

---

### Invalid RSS/XML

**Cause**

The server returned content that does not match the expected RSS structure.

**Solution**

Verify that the selected Le Matin endpoint returns valid RSS XML.

The parser expects a `<channel>` element containing RSS feed information and `<item>` elements for articles.

---

### No Articles Returned

**Cause**

The selected RSS feed contains no `<item>` elements or no article entries are currently available.

**Solution**

Try another category or verify that the selected feed currently contains articles.

---

### Too Many Articles

**Cause**

The workflow is limited by `maxItems`.

**Solution**

Increase the value or set it to `0`.

Example:

```json
{
  "category": "latest",
  "maxItems": 25
}
```

---

### Unexpected Category

**Cause**

The configured category is not one of the supported enum values.

**Solution**

Use one of the supported categories:

```text
latest
regions
sports
economy
world
nation
society
tv
editorials
royal_activities
automobile
employment
opinions
agenda
lifestyle
hi_tech
specials
royal_visits
mtf
education
siam
africa_development
culture
chronicles
```

---

## Security

The node performs outbound HTTP requests to Le Matin RSS endpoints.

No API key or authentication credential is required by the node.

The feed URL is selected internally from a fixed category mapping rather than accepting arbitrary user-provided URLs.

This prevents the node configuration from being used as a generic arbitrary URL fetcher.

---

## Notes

The node returns parsed RSS data rather than the raw XML response.

The node does not:

- Publish articles
- Modify Le Matin content
- Download article pages
- Download media files
- Generate summaries
- Translate articles
- Store articles
- Generate embeddings
- Verify the truth of news content

It is intended to retrieve and structure Le Matin RSS content for downstream workflow processing.

---

## Related

- [Function](./function.md) – Transform and process RSS articles
- [If](./if.md) – Filter and route articles based on conditions
- [HTTP Request](./http-request.md) – Make additional HTTP requests
- [Database](./database.md) – Store retrieved articles

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-11 | Initial release |

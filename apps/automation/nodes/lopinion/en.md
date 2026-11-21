---
node_id: "lopinion"
title: "L'Opinion"
description: "Fetch L'Opinion articles by category, with Google News RSS as a fallback."
category: "Web Search & Information"
subcategory: "News"
version: "1.0.0"
language: "en"
last_updated: "2026-11-21"
author: "Fusion Team"
tags:
  - lopinion
  - news
  - morocco
  - rss
related_nodes:
  - manual-trigger
  - function
  - http-request
---

<!-- SECTION: overview -->
# L'Opinion

> **Type:** Action Node

Fetch articles from the selected category on L'Opinion, a French-language Moroccan news website. The node reads the category page and returns a feed-shaped object containing article titles, links and available descriptions and images.

If the website request fails or no article links can be extracted, the node requests Google News RSS for `site:lopinion.ma`. This fallback searches across the domain and can include articles from other categories. The response contains the metadata supplied by the selected path.

Use this node to prepare a news digest, monitor a category or pass article metadata to a summarization workflow.

<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `category` | enum | Yes, with a default | `actu-maroc` | Category page to request. Supports Expressions. |
| `maxItems` | number | Optional | `10` | Maximum items returned when positive. Supports Expressions. |

| Category value | Section |
|---|---|
| `actu-maroc` | Moroccan news |
| `sport` | Sport |
| `eco` | Economy |
| `international` | International news |
| `culture` | Culture |
| `societe` | Society |
| `agora` | Agora |

Use a positive integer for MaxItems. The current implementation uses 10 when the value is zero or omitted. Negative values disable the positive-limit check. Positive fractional values are accepted by the schema and can produce different item counts between the website and RSS paths. A positive integer provides a consistent requested limit.

For the supplied workflow, select `sport` and set MaxItems to `10`.

<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

Connect an upstream trigger to the standard input to start execution. The node reads category and MaxItems from configuration. To use upstream values, bind the configuration fields through Expressions.

### Output

The success output contains the feed object directly.

| Field | Type | Description |
|---|---|---|
| `title` | optional string | Website mode returns `L'Opinion - <category>`. RSS fallback uses the channel title. |
| `link` | optional string | Category page URL in website mode, or channel link from RSS. |
| `description` | optional string | Feed description. |
| `language` | optional string | `fr` in website mode. The RSS parser currently leaves this field unset. |
| `lastBuildDate` | optional string | Fetch-time ISO timestamp generated in website mode. |
| `pubDate` | optional string | Fetch-time ISO timestamp generated for the website feed. |
| `items` | array | Extracted article metadata. |

Each item can include:

| Field | Type | Description |
|---|---|---|
| `title` | string | Decoded article title. |
| `link` | string | Article link. RSS fallback can use a Google News redirect URL. |
| `description` | optional string | Description extracted from the page or RSS. Availability varies by article. |
| `image` | optional string | Image URL extracted from the website. |
| `pubDate` | optional string | Website-relative text such as `il y a 7h`, an ISO fetch-time fallback, or the RSS item's date string. |
| `guid` | optional string | Website mode uses the article link. RSS mode uses the item's GUID when present. |
| `category` | optional string array | Website mode assigns the requested category. RSS fallback currently omits this field. |

Website items preserve extraction order and are deduplicated by full article link. A shorter response can occur when fewer articles are available. Category labels on website items reflect the requested page; extraction can also pick up other supported category links present on that page.

The following excerpt contains the first item from the supplied sport run, which returned ten items. Article titles and relative dates vary over time.

```json
{
  "title": "L'Opinion - sport",
  "link": "https://lopinion.ma/sport",
  "description": "Dernières actualités L'Opinion (sport)",
  "language": "fr",
  "lastBuildDate": "2026-11-21T14:49:44.018Z",
  "pubDate": "2026-11-21T14:49:44.018Z",
  "items": [
    {
      "title": "Mondial. Foot pour amputés : les Guerriers de l’Atlas FI-NA-LIS-STES !",
      "link": "https://lopinion.ma/sport/newsroom/mondial-foot-pour-amputes--les-guerriers-de-latlas-fi-na-lis-stes--_a80624?articleId=d79e3c5d-ec5f-4af4-aa28-357f25fe8c83",
      "image": "https://lopinion.ma/media/uploads/1795245128917-thumbnail.jpg",
      "pubDate": "il y a 7h",
      "guid": "https://lopinion.ma/sport/newsroom/mondial-foot-pour-amputes--les-guerriers-de-latlas-fi-na-lis-stes--_a80624?articleId=d79e3c5d-ec5f-4af4-aa28-357f25fe8c83",
      "category": ["sport"]
    }
  ]
}
```

The website path generates a current timestamp when an article's relative date cannot be extracted. Treat that value as fetch-time metadata until publication time is confirmed from the source article.

<!-- /SECTION: inputs-outputs -->

<!-- SECTION: workflow-example -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Fetch L'Opinion sport articles and inspect the feed
```

This example adapts the supplied export into Manual Trigger → L'Opinion → Log.

1. Import the example graph.
2. Open L'Opinion and select Category `sport`.
3. Set MaxItems to `10`.
4. Start the workflow from Manual Trigger.
5. Inspect `items` in the Log or node Output.

The graph omits configuration values and credentials according to the contribution rules. Enter the category and item limit in the builder.

For a compact inspection, add a Function after L'Opinion:

```javascript
return {
  title: input.title,
  count: input.items.length,
  items: input.items.map(item => ({
    title: item.title,
    link: item.link,
    pubDate: item.pubDate,
    image: item.image ?? null
  }))
};
```

To prepare a digest with an Agent, pass `items` as article metadata and instruct it to preserve source links and distinguish missing descriptions. Summaries based only on titles and descriptions should reflect those available fields. Article body retrieval requires an additional lookup.

<!-- /SECTION: workflow-example -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Observation | Explanation and action |
|---|---|
| Feed title or links differ from the selected category | The Google News RSS fallback may have been used. It searches the whole L'Opinion domain. Inspect feed metadata and item links. |
| `L'Opinion feed fetch failed: ...` | Website extraction failed and the RSS fallback also failed. The final error reports the fallback failure. Check connectivity and provider availability. |
| Missing description or image | The selected page or RSS response may omit that metadata. Handle absent fields downstream. |
| Dates mix relative text and ISO timestamps | Website extraction preserves relative dates and substitutes fetch time when no date is found. RSS supplies its own strings. |
| Fewer than MaxItems articles | Only the extracted or RSS-provided items are available. Compare with the live source. |
| MaxItems zero returns ten articles | Zero uses the default limit in the current implementation. Use a positive integer. |
| Long-running request | Both requests currently use fetch without an explicit timeout. Inspect workflow execution limits and provider availability. |

The fallback occurs automatically and has no dedicated source indicator in the output. The RSS parser can also return an empty `items` array for a response with no recognized channel items.

<!-- /SECTION: troubleshooting -->

<!-- SECTION: related -->
## Related

- [Manual Trigger](/docs/manual-trigger) — Start an article lookup.
- [Function](/docs/function) — Inspect or transform feed items.
- [HTTP Request](/docs/http-request) — Fetch additional source content.

<!-- /SECTION: related -->

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0.0 | 2026-11-21 | Document categories, feed output, website extraction, RSS fallback and the supplied sport workflow. |

<!-- /SECTION: changelog -->

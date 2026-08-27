---
node_id: "giphy-gif-search"
title: "Giphy GIF Search"
description: "Search for animated GIFs, stickers, and multi-resolution media links from Giphy."
category: "Image Search"
subcategory: "media-assets"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:
  - giphy
  - gif
  - search
  - animation
  - media
  - memes
  - discord
  - slack
related_nodes:
  - unsplash-image-search
  - pexels-image-search
  - pixabay-image-search
  - wikimedia-commons
  - function
  - manual-trigger
---

<!-- SECTION: header -->
# Giphy GIF Search

> **Category:** Image Search | **Type:** Action Node

Search the world's largest library of animated GIFs on **Giphy** and retrieve direct media URLs across multiple resolutions and formats (original, downsized, fixed height, fixed width).

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Giphy GIF Search** node connects workflows to Giphy's search engine API. It enables you to perform keyword-based searches for trending memes, animated reactions, and thematic animations to enhance messaging bots, content automation pipelines, and social media notifications.

For each GIF found, the node extracts complete metadata including direct `.gif` and `.mp4` URLs, iframe embed links, content ratings (`g`, `pg`, `pg-13`, `r`), original author usernames, and 11 distinct optimized image renditions tailored for web and mobile performance.

### Key Features

- **Keyword & Phrase Search:** Query millions of animated GIFs using keywords, phrases, emotion tags, or trending topics.
- **11 Pre-Rendered Media Variations:** Access optimized variations for every GIF (`original`, `downsized`, `downsized_medium`, `fixed_height`, `fixed_width`, mobile downsampled, etc.).
- **Configurable Result Limits:** Retrieve between 1 and 50 GIFs per request.
- **Dynamic Query Resolution:** Automatically accepts search terms passed dynamically from upstream triggers, webhooks, or JavaScript `Function` nodes.
- **Direct CDN URLs:** Returns hotlinkable direct media URLs ready for Discord, Slack, email campaigns, and HTML embedding.

### Common Use Cases

- **Chatbot & Community Reactions:** Automatically attach relevant reaction GIFs to Discord bot replies, Slack messages, or Telegram updates.
- **Content Automation & Newsletters:** Embed engaging GIF illustrations into automated marketing emails and social media drafts.
- **Customer Support & Ticket Celebrations:** Post animated celebration GIFs when support tickets are resolved or milestones are reached.
- **AI Agent Tooling:** Allow AI Chat and autonomous agents to search and present visual animations to users.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## How to Use the Giphy GIF Search Node

Configure the node with your Giphy API Key, enter your search keywords, and specify how many results you want returned.

![Giphy GIF Search Configuration](icon.svg)

### Step-by-Step Setup in the Visual Builder

1. **Get a Giphy API Key:** Create a free developer account on the [Giphy Developer Portal](https://developers.giphy.com/) and create an app to obtain your API Key.
2. **Add the Node:** Drag the **Giphy GIF Search** node from the **Image Search** category onto your canvas.
3. **Configure the API Key:** Paste your Giphy API Key into the **ApiKey** field.
4. **Enter the Search Query:**
   - Type search keywords into the **Query** field (e.g. `superman and batman`, `happy dance`, `thank you`).
   - Alternatively, leave the field empty to receive the query dynamically from the preceding node's output.
5. **Set Result Limit:** Choose how many GIFs to return (default is `5`, maximum is `50`).
6. **Connect Outputs:** Wire the `success` output port to a `Function`, `Log`, `Discord Webhook`, or `Slack` node.

---

### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|:-------:|-------------|
| `apiKey` | `string` | ✅ Yes | — | Your Giphy Developer API Key. |
| `query` | `string` | ❌ No* | — | Search keywords or phrase (e.g. `batman`, `celebrate`). *Required at runtime if not passed via input. |
| `limit` | `number` | ❌ No | `5` | Maximum number of GIFs to return (1 to 50). |

---

### Dynamic Query Resolution

If the `query` parameter is left empty in the node configuration, the node dynamically extracts the search query from incoming workflow data:

```
                  ┌────────────────────────────────────────┐
                  │          Incoming Input Data           │
                  └──────────────────┬─────────────────────┘
                                     │
                   Is `query` set in node configuration?
                                     │
                     ┌───────────────┴───────────────┐
                    YES                              NO
                     │                               │
             Use config `query`          Use incoming `input`
                                         (string payload)
```

- **String Payload:** If the previous node outputs a string (e.g. `"victory dance"`), the node automatically searches for that term.
- **Expression Mapping:** You can map properties from upstream objects using expressions (e.g. `{{Webhook.body.keyword}}`).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` | Optional incoming string containing the search query (e.g. `"superman and batman"`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when Giphy returns search results. Contains the query and an array of GIF items. |
| `error` | `Error` | Emitted when `apiKey` is missing, query is empty, or the Giphy API returns an error. |

---

### Output Data Schema

```json
{
  "success": true,
  "query": "superman and batman",
  "results": [
    {
      "id": "13NBiMh0ZNJyX6",
      "title": "superman batman GIF",
      "url": "https://giphy.com/gifs/13NBiMh0ZNJyX6",
      "embed_url": "https://giphy.com/embed/13NBiMh0ZNJyX6",
      "username": "dc",
      "rating": "g",
      "images": {
        "original": "https://media.giphy.com/media/13NBiMh0ZNJyX6/giphy.gif",
        "downsized": "https://media.giphy.com/media/13NBiMh0ZNJyX6/giphy-downsized.gif",
        "downsized_large": "https://media.giphy.com/media/13NBiMh0ZNJyX6/giphy-downsized-large.gif",
        "downsized_medium": "https://media.giphy.com/media/13NBiMh0ZNJyX6/giphy-downsized-medium.gif",
        "downsized_small": "https://media.giphy.com/media/13NBiMh0ZNJyX6/giphy-downsized-small.mp4",
        "fixed_height": "https://media.giphy.com/media/13NBiMh0ZNJyX6/200.gif",
        "fixed_height_downsampled": "https://media.giphy.com/media/13NBiMh0ZNJyX6/200_d.gif",
        "fixed_height_small": "https://media.giphy.com/media/13NBiMh0ZNJyX6/100.gif",
        "fixed_width": "https://media.giphy.com/media/13NBiMh0ZNJyX6/200w.gif",
        "fixed_width_downsampled": "https://media.giphy.com/media/13NBiMh0ZNJyX6/200w_d.gif",
        "fixed_width_small": "https://media.giphy.com/media/13NBiMh0ZNJyX6/100w.gif"
      },
      "source": "https://dccomics.com",
      "source_tld": "dccomics.com",
      "source_post_url": "https://dccomics.com/..."
    }
  ],
  "total_results": 1,
  "note": "Use the 'url' or 'images' field of each result for GIF embedding"
}
```

---

### Output Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates whether the query completed successfully (`true`). |
| `query` | `string` | The search term queried. |
| `results` | `array` | List of GIF result objects. |
| `results[].id` | `string` | Giphy unique GIF ID. |
| `results[].title` | `string` | Descriptive title of the GIF. |
| `results[].url` | `string` | Web link to the GIF page on Giphy.com. |
| `results[].embed_url` | `string` | Iframe embed URL for web embedding. |
| `results[].username` | `string` | Username of the creator or verified channel who uploaded the GIF. |
| `results[].rating` | `string` | Content rating (`g`, `pg`, `pg-13`, `r`). |
| `results[].images.original` | `string` | Full-resolution original animated `.gif` URL. |
| `results[].images.downsized` | `string` | Downsized animated `.gif` under 2MB (ideal for messaging bots). |
| `results[].images.fixed_height` | `string` | Animated `.gif` scaled to 200px height. |
| `results[].images.fixed_width` | `string` | Animated `.gif` scaled to 200px width. |
| `results[].images.fixed_height_small` | `string` | Thumbnail animated `.gif` scaled to 100px height. |
| `total_results` | `number` | Total number of GIF items returned in this response. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Practical Examples

### Example 1: Basic Search for Top 5 Reaction GIFs

Search for `"superman and batman"` reaction GIFs.

**Configuration:**
```json
{
  "apiKey": "YOUR_GIPHY_API_KEY",
  "query": "superman and batman",
  "limit": 5
}
```

---

### Example 2: Extracting First GIF for Discord / Slack Bots

Use a downstream `Function` node to extract the direct GIF URL from the search result.

**Downstream Function Code:**
```javascript
const firstGif = input.results[0];

return {
  title: firstGif.title,
  gifUrl: firstGif.images.original,
  previewUrl: firstGif.images.fixed_height_small,
  giphyPage: firstGif.url
};
```

**Function Output:**
```json
{
  "title": "superman batman GIF",
  "gifUrl": "https://media.giphy.com/media/13NBiMh0ZNJyX6/giphy.gif",
  "previewUrl": "https://media.giphy.com/media/13NBiMh0ZNJyX6/100.gif",
  "giphyPage": "https://giphy.com/gifs/13NBiMh0ZNJyX6"
}
```

---

### Example 3: Dynamic Search Driven by Incoming Webhook / Chatbot

A chat trigger emits a search term (e.g. `"applause"`).

**Upstream Node Output:**
```json
"applause"
```

**Giphy Node Configuration:**
- **ApiKey:** `YOUR_GIPHY_API_KEY`
- **Query:** *(leave empty)*
- **Limit:** `3`

The node automatically searches for `"applause"` and returns the top 3 results.

---

### Example 4: Choosing the Optimal Image Rendition for Performance

Depending on your target platform, select the best image size from the `images` object:

| Target Platform / Use Case | Recommended Field | Format |
|-----------------------------|-------------------|--------|
| Discord / Slack Embeds | `images.downsized` or `images.original` | `.gif` |
| Mobile Notifications / SMS | `images.downsized_small` | `.mp4` (ultra-compact) |
| Web UI Thumbnails | `images.fixed_height_small` (100px) | `.gif` |
| Grid Galleries | `images.fixed_width` (200px) | `.gif` |

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Interactive Workflow Preview

```fusion-workflow
src: example.workflow.json
title: Giphy GIF Search Workflow
```

---

### Sample Workflows

#### 1. Manual Search & Log Pipeline

A simple workflow to manually trigger a Giphy search and log the resulting media URLs:

```json
{
  "nodes": [
    {
      "id": "trigger",
      "type": "manual-trigger",
      "label": "Run Search"
    },
    {
      "id": "giphy-search",
      "type": "giphy-gif-search",
      "config": {
        "apiKey": "YOUR_GIPHY_API_KEY",
        "query": "superman and batman",
        "limit": 5
      }
    },
    {
      "id": "log-results",
      "type": "log",
      "label": "Display Results"
    }
  ],
  "connections": [
    {
      "source": "trigger",
      "target": "giphy-search"
    },
    {
      "source": "giphy-search",
      "target": "log-results"
    }
  ]
}
```

---

#### 2. Chatbot Auto-Reaction Workflow

A pipeline where incoming messages are analyzed, a reaction keyword is generated, and a Giphy reaction is posted back to Slack:

```
  ┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
  │  Slack Trigger   │ ────▶ │  AI Sentiment    │ ────▶ │ Giphy GIF Search │ ────▶ │  Slack Response  │
  │ (New user post)  │       │(Tag: 'celebrate')│       │ (Limit: 1)       │       │ (Post GIF embed) │
  └──────────────────┘       └──────────────────┘       └──────────────────┘       └──────────────────┘
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Errors & Solutions

#### `Giphy GIF search failed: GIPHY_API_KEY is required`
- **Cause:** The `apiKey` parameter was left empty.
- **Solution:** Create a free API key on [developers.giphy.com](https://developers.giphy.com/) and paste it into the `apiKey` field.

---

#### `Giphy GIF search failed: Search query is required`
- **Cause:** The `query` parameter was not configured, and no string input was provided by the upstream node.
- **Solution:** Enter a search query in the node parameters or ensure the preceding node emits a non-empty string.

---

#### `Giphy API error: 403`
- **Cause:** The provided Giphy API Key is invalid, suspended, or has expired.
- **Solution:** Verify your API Key on the Giphy Developer Dashboard.

---

#### `Giphy API error: 429 Too Many Requests`
- **Cause:** Free tier rate limits reached on Giphy (Giphy prototype keys allow up to 42 searches/hour and 1000 searches/day).
- **Solution:** Upgrade to a production Giphy key on the Giphy dashboard or use a `Delay` node to space out requests.

---

### Error Reference Table

| Error Message | Cause | Resolution |
|---------------|-------|------------|
| `GIPHY_API_KEY is required` | Missing API key | Supply an active Giphy API key in configuration. |
| `Search query is required` | Empty search query | Enter a query or pass data from upstream. |
| `Giphy API error: 403` | Invalid credentials | Check API Key validity on Giphy Developer portal. |
| `Giphy API error: 429` | Rate limit exceeded | Upgrade key or add Delay node in loops. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: security -->
## Security & Best Practices

- **Protect Your API Key:** Never commit plain Giphy API keys into shared workflow exports. Store them in Fusion credentials or environment variables.
- **Content Moderation:** Giphy content ratings range from `g` to `r`. If building family-friendly tools, verify the `rating` property in downstream filters.

<!-- /SECTION: security -->

---

<!-- SECTION: related -->
## Related Nodes

- [Unsplash Image Search](../unsplash-image-search/en.md) — Query high-resolution stock photography
- [Pixabay Image Search](../pixabay-image-search/en.md) — Search royalty-free photos and illustrations
- [Pexels Image Search](../pexels-image-search/en.md) — Search curated stock photos and videos
- [Wikimedia Commons](../wikimedia-commons/en.md) — Query open media, vectors, and audio recordings
- [Function](../function/en.md) — Extract and transform Giphy image URLs for messaging platforms

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-27 | Initial release of Giphy GIF Search Action Node with multi-resolution renditions and dynamic input resolution |

<!-- /SECTION: changelog -->

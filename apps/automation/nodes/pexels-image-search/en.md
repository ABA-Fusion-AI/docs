---
node_id: "pexels-image-search"
title: "Pexels Image Search"
description: "Search for free, high-quality stock photos from Pexels with multi-resolution URLs, photographer credits, and landscape aspect ratios."
category: "Image Search"
subcategory: "media-assets"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:
  - pexels
  - image-search
  - stock-photos
  - photography
  - media
  - photos
  - creative
  - social-media
  - banners
  - wallpapers
  - media-assets
related_nodes:
  - unsplash-image-search
  - pixabay-image-search
  - giphy-gif-search
  - google-images-search
  - function
  - webhook
  - slack
  - discord-bot-send
---

<!-- SECTION: header -->
# Pexels Image Search

> **Category:** Image Search | **Subcategory:** Media Assets | **Type:** Action Node

Search millions of free, curated, high-resolution stock photos on [Pexels](https://www.pexels.com/) and retrieve direct image URLs across multiple resolutions (`original`, `large`, `medium`, `small`, `thumbnail`) along with photographer attribution.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Pexels Image Search** node connects automated workflows to the [Pexels API](https://www.pexels.com/api/), providing instant access to a vast library of professional stock photography. It enables workflow builders to dynamically discover images based on descriptive search queries, filter results by quantity, and retrieve optimized image URLs tailored for headers, thumbnails, banners, and social posts.

Every query to Pexels returns rich photo objects containing distinct pre-scaled image dimensions and complete attribution metadata, including photographer names and profile URLs.

```
                  ┌────────────────────────┐
                  │    Trigger / Webhook   │
                  │ (e.g., Blog Published) │
                  └───────────┬────────────┘
                              │
                              ▼
                  ┌────────────────────────┐
                  │  Pexels Image Search   │
                  │  query: "coffee paris" │
                  │  perPage: 3            │
                  └───────────┬────────────┘
                              │
            ┌─────────────────┴─────────────────┐
            │                                   │
      [success]                           [error]
            │                                   │
            ▼                                   ▼
┌───────────────────────────┐       ┌───────────────────────────┐
│     Downstream Nodes      │       │     Error Handling        │
│ • Embed Header in Article │       │ • Fallback Stock Provider │
│ • Post to Slack / Discord │       │ • Alert / Retry Flow      │
│ • Feed into AI Image Node │       │                           │
└───────────────────────────┘       └───────────────────────────┘
```

### Key Capabilities

- **Curated High-Resolution Library:** Access millions of professional stock photos uploaded by creators globally.
- **Multiple Resolution Assets:** Each result delivers ready-to-use image assets:
  - `original`: Full native camera resolution.
  - `large`: Scaled to maximum width of 940px / height of 650px (ideal for blog headers).
  - `medium`: Scaled to height of 350px (ideal for article insets and cards).
  - `small`: Scaled to height of 130px (ideal for mobile previews).
  - `thumbnail`: Cropped to 280x200px (ideal for grids and avatars).
- **Default Landscape Orientation:** Automatically requests landscape-oriented images, perfect for website banners, email headers, and OpenGraph social cards.
- **Photographer Attribution:** Provides full creator attribution metadata (`photographer`, `photographer_url`) to comply with licensing and attribution best practices.
- **Flexible Query Ingestion:** Accepts search queries configured directly in the node parameters or dynamically from upstream nodes via `outputs.<node_label>.<output_port>.<field>` (or incoming string payload).
- **Safe Batch Sizing:** Allows requesting up to 80 photos per call (automatically capped at 80 to respect Pexels API constraints).

---

### Obtaining a Pexels API Key

To use the Pexels Image Search node, you need a free Pexels API key:

1. Create a free account at [pexels.com](https://www.pexels.com/).
2. Navigate to the [Pexels API Portal](https://www.pexels.com/api/).
3. Click **Get Started** and fill out the brief application form.
4. Copy your unique API Key and paste it into the `apiKey` field in the node configuration or pass it via expressions.

> [!NOTE]
> **Free Tier Limits:** The standard free Pexels API key offers **200 requests per hour** and **20,000 requests per month**, which is more than sufficient for most production workflows and bots.

---

### Common Use Cases

- **Automated Blog & Article Publishing:** Receive a published blog post topic via Webhook, search Pexels for a relevant landscape photo, and set it as the featured header image in WordPress or Ghost.
- **Social Media Automation:** Generate automated daily tips or quotes and pair them with matching background imagery before publishing to Twitter, LinkedIn, or Pinterest.
- **Chatbot & Messaging Bots:** Power Slack or Discord commands (e.g. `/photo coffee`) to fetch and return high-resolution photo previews.
- **AI Image Pipeline Ingestion:** Fetch real-world reference photos from Pexels and pass their URLs into AI image-to-image or vision models.
- **E-Commerce Mockups & Prototyping:** Automatically populate test product listings and storefronts with realistic lifestyle and product photography.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Configure the Pexels Image Search node in the workflow canvas by setting your search query, desired number of photos, and Pexels API key.

![Pexels Node Configuration](icon.svg)

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|:----:|:--------:|:-------:|-------------|
| `query` | `string` | ✅ Yes | — | Search keywords or phrase (e.g. `"mountains landscape"`, `"coffee & croissant in paris"`, `"cyberpunk neon city"`). If omitted in config, falls back to the incoming `data` payload string. |
| `perPage` | `number` | ❌ No | `5` | Number of image results to return (1–80). Automatically capped at 80. |
| `apiKey` | `string` | ✅ Yes | `""` | Your Pexels API Authorization token. Required for authenticating with the Pexels API. |

---

### Detailed Parameter Descriptions

#### `query` (Required)
The keyword or descriptive phrase describing the images you want to find.
- **Examples:** `"mountains landscape"`, `"modern minimalist office"`, `"sunset beach"`, `"vintage cars"`.
- **Query Resolution:** If the `query` field is left blank in the configuration, the node will check if the incoming `data` payload is a string and use it as the search query.
- **Dynamic Expression:** Click **Expression** to bind dynamic values from upstream nodes, such as `outputs.Function.success.Query` or `outputs.Webhook.success.body.searchQuery`.

#### `perPage` (Optional)
The maximum number of images to return in the `results` array.
- **Default:** `5`
- **Range:** `1` to `80` (values greater than 80 are automatically capped to 80).
- **Recommendation:** Use `1` to `3` when you only need a single featured image or thumbnail to minimize payload sizes.
- **Dynamic Expression:** Can be bound dynamically via `outputs.Function.success.perPage`.

#### `apiKey` (Required)
The authorization key provided by Pexels.
- **Format:** A 56-character alphanumeric string (e.g. `icpblm5gNbEf...`).
- **Dynamic Expression:** Can be passed statically or dynamically from an upstream node or configuration.

---

### Static vs Expression Mode

You can toggle between direct text entry and dynamic expression evaluation for each parameter:

```
┌─────────────────────────────────────────────────────────────┐
│ Parameters                                                  │
│                                                             │
│ Query                                           [Expression]│
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ outputs.Function.success.Query                          │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ PerPage (Optional)                              [Expression]│
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ 2                                                       │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ ApiKey (Optional)                               [Expression]│
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ outputs.Config.success.apiKey                           │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

- **Static Mode:** Enter fixed values directly into the input fields.
- **Expression Mode:** Click **Expression** next to any field to inject upstream node outputs using the syntax `outputs.<NodeLabel>.<outputPort>.<field>`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|:----:|-------------|
| `input` | `any` | Incoming trigger or upstream action payload. If `query` is not specified in the node configuration, the node accepts a raw string payload from `input` as the fallback search query. |

### Outputs

| Output | Type | Description |
|--------|:----:|-------------|
| `success` | `object` | Emitted when the Pexels API returns matching photo results. Contains the query, results array with multi-resolution URLs, creator attribution, and count. |
| `error` | `Error` | Emitted if the API key is missing, if the search query is empty, or if an API error (e.g. HTTP 401, 429, 500) occurs. |

---

### Output Data Structure Example

When searching with `query: "coffee & croissant in paris"` and `perPage: 2`:

```json
{
  "success": true,
  "query": "coffee & croissant in paris",
  "results": [
    {
      "id": 2133989,
      "url": "https://images.pexels.com/photos/2133989/pexels-photo-2133989.jpeg?auto=compress&cs=tinysrgb&h=650&w=940",
      "photographer": "Daria Shevtsova",
      "photographer_url": "https://www.pexels.com/@daria",
      "src": {
        "original": "https://images.pexels.com/photos/2133989/pexels-photo-2133989.jpeg",
        "large": "https://images.pexels.com/photos/2133989/pexels-photo-2133989.jpeg?auto=compress&cs=tinysrgb&h=650&w=940",
        "medium": "https://images.pexels.com/photos/2133989/pexels-photo-2133989.jpeg?auto=compress&cs=tinysrgb&h=350",
        "small": "https://images.pexels.com/photos/2133989/pexels-photo-2133989.jpeg?auto=compress&cs=tinysrgb&h=130",
        "thumbnail": "https://images.pexels.com/photos/2133989/pexels-photo-2133989.jpeg?auto=compress&cs=tinysrgb&fit=crop&h=200&w=280"
      }
    },
    {
      "id": 1307698,
      "url": "https://images.pexels.com/photos/1307698/pexels-photo-1307698.jpeg?auto=compress&cs=tinysrgb&h=650&w=940",
      "photographer": "Igor Starkov",
      "photographer_url": "https://www.pexels.com/@igor-starkov-233054",
      "src": {
        "original": "https://images.pexels.com/photos/1307698/pexels-photo-1307698.jpeg",
        "large": "https://images.pexels.com/photos/1307698/pexels-photo-1307698.jpeg?auto=compress&cs=tinysrgb&h=650&w=940",
        "medium": "https://images.pexels.com/photos/1307698/pexels-photo-1307698.jpeg?auto=compress&cs=tinysrgb&h=350",
        "small": "https://images.pexels.com/photos/1307698/pexels-photo-1307698.jpeg?auto=compress&cs=tinysrgb&h=130",
        "thumbnail": "https://images.pexels.com/photos/1307698/pexels-photo-1307698.jpeg?auto=compress&cs=tinysrgb&fit=crop&h=200&w=280"
      }
    }
  ],
  "total_results": 2,
  "note": "Use the 'url' field of each result for image generation"
}
```

---

### Output Field Reference

| Field | Type | Description |
|-------|:----:|-------------|
| `success` | `boolean` | Indicates that the search request completed successfully (`true`). |
| `query` | `string` | The search query string sent to the Pexels API. |
| `results` | `array` | Array of image objects matching the query. |
| `results[i].id` | `number` | Unique numeric identifier for the Pexels photo. |
| `results[i].url` | `string` | Primary direct web-ready image URL (defaults to `large`, fallback to `original` or `medium`). |
| `results[i].photographer` | `string` | Name of the photographer who created the image. |
| `results[i].photographer_url` | `string` | URL to the photographer's official Pexels profile. |
| `results[i].src.original` | `string` | Full uncompressed original resolution photo URL. |
| `results[i].src.large` | `string` | Large resolution photo URL (max width: 940px, height: 650px). |
| `results[i].src.medium` | `string` | Medium resolution photo URL (height: 350px). |
| `results[i].src.small` | `string` | Small resolution photo URL (height: 130px). |
| `results[i].src.thumbnail` | `string` | Cropped thumbnail URL (280x200px). |
| `total_results` | `number` | Number of image items returned in the `results` array. |
| `note` | `string` | Guidance note on using the `url` field for downstream image generation pipelines. |

---

### Consuming Output Data in Downstream Nodes

Downstream nodes can reference the output of **Pexels Image Search** (assuming the node is labeled `PexelsImageSearch` or `Pexels_Image_Search`) using expression paths:

| Target Image Asset | Expression Syntax | Typical Usage |
|--------------------|-------------------|---------------|
| **Primary Image URL** | `outputs.PexelsImageSearch.success.results[0].url` | General display, web embedding, AI image generation |
| **High-Res Header** | `outputs.PexelsImageSearch.success.results[0].src.large` | Article headers, full-width web banners |
| **Original Native Image** | `outputs.PexelsImageSearch.success.results[0].src.original` | Print graphics, high-DPI desktop wallpapers |
| **Medium Inset** | `outputs.PexelsImageSearch.success.results[0].src.medium` | Social card embeds, blog post body illustrations |
| **Small Preview** | `outputs.PexelsImageSearch.success.results[0].src.small` | Mobile notification previews, chat thumbnails |
| **Square/Grid Thumbnail** | `outputs.PexelsImageSearch.success.results[0].src.thumbnail` | Card grids, profile headers, catalog index icons |
| **Photographer Attribution** | `outputs.PexelsImageSearch.success.results[0].photographer` | Captions, credit footnotes, metadata tags |
| **Photographer Profile** | `outputs.PexelsImageSearch.success.results[0].photographer_url` | Clickable photographer link |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Stock Photo Search (Coffee & Croissant in Paris)

Search for atmospheric culinary and travel stock photos:

**Configuration:**
- **Query:** `coffee & croissant in paris`
- **PerPage:** `2`
- **ApiKey:** `icpblm5gNbEf...`

**Output (`success`):**
```json
{
  "success": true,
  "query": "coffee & croissant in paris",
  "results": [
    {
      "id": 2133989,
      "url": "https://images.pexels.com/photos/2133989/pexels-photo-2133989.jpeg?auto=compress&cs=tinysrgb&h=650&w=940",
      "photographer": "Daria Shevtsova",
      "photographer_url": "https://www.pexels.com/@daria",
      "src": {
        "original": "https://images.pexels.com/photos/2133989/pexels-photo-2133989.jpeg",
        "large": "https://images.pexels.com/photos/2133989/pexels-photo-2133989.jpeg?auto=compress&cs=tinysrgb&h=650&w=940",
        "medium": "https://images.pexels.com/photos/2133989/pexels-photo-2133989.jpeg?auto=compress&cs=tinysrgb&h=350",
        "small": "https://images.pexels.com/photos/2133989/pexels-photo-2133989.jpeg?auto=compress&cs=tinysrgb&h=130",
        "thumbnail": "https://images.pexels.com/photos/2133989/pexels-photo-2133989.jpeg?auto=compress&cs=tinysrgb&fit=crop&h=200&w=280"
      }
    }
  ],
  "total_results": 1,
  "note": "Use the 'url' field of each result for image generation"
}
```

---

### Example 2: Single Header Image for Blog (Cyberpunk Neon City)

Fetch a single landscape header image for a technology article:

**Configuration:**
- **Query:** `cyberpunk neon city`
- **PerPage:** `1`
- **ApiKey:** `icpblm5gNbEf...`

**Downstream Slack Message Block:**
```json
{
  "channel": "#general",
  "text": "New article published!",
  "blocks": [
    {
      "type": "image",
      "title": { "type": "plain_text", "text": "Cyberpunk Neon City" },
      "image_url": "outputs.PexelsImageSearch.success.results[0].src.large",
      "alt_text": "Photo by outputs.PexelsImageSearch.success.results[0].photographer on Pexels"
    }
  ]
}
```

---

### Example 3: Dynamic Search Query from Upstream Function Node

Dynamically pass keywords generated by an upstream **Function** node labeled `Function`:

**Upstream Function Node (`Function`):**
```javascript
return {
  Query: "minimalist workspace design",
  PerPage: 3
};
```

**Pexels Node Configuration:**
- **Query:** `outputs.Function.success.Query`
- **PerPage:** `outputs.Function.success.PerPage`
- **ApiKey:** `icpblm5gNbEf...`

---

### Example 4: Formatting Image Results for Webhook Response

Use a **Function** node after Pexels to generate an HTML image gallery or markdown list:

```javascript
// Transform results array into clean markdown links with attribution
const images = input.results.map((photo, index) => {
  return `![Photo ${index + 1}](${photo.src.medium})\n*Photo by [${photo.photographer}](${photo.photographer_url}) on Pexels*`;
}).join("\n\n");

return {
  query: input.query,
  markdownGallery: images,
  primaryImageUrl: input.results[0]?.url || ""
};
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search and retrieve curated stock images from Pexels
```

### Common Architecture Patterns

#### Pattern 1: Automatic Blog Header Image Enrichment
```text
[Webhook: New Article Draft] (Extracts title / keywords)
          ↓
[Pexels Image Search] (query: outputs.Webhook.success.body.topic, perPage: 1)
          ↓
[WordPress / Ghost Action] (Upload Image URL as Featured Header)
```

#### Pattern 2: Social Media Marketing Card Generator
```text
[Cron Trigger / Daily Scheduler]
          ↓
[Function: Select Daily Theme] (returns { Query: "inspiring nature sunrise" })
          ↓
[Pexels Image Search] (query: outputs.Function.success.Query, perPage: 1)
          ↓
[Twitter / LinkedIn Action] (Attach outputs.PexelsImageSearch.success.results[0].src.large)
```

#### Pattern 3: On-Demand Chatbot Photo Search
```text
[Chat Trigger / Discord Slash Command] (/image <query>)
          ↓
[Pexels Image Search] (query: outputs.ChatTrigger.success.query, perPage: 3)
          ↓
[Discord Bot Send] (Embed multi-image carousel with photographer links)
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues & Solutions

#### 1. `Pexels image search failed: PEXELS_API_KEY is required`
- **Cause:** The `apiKey` parameter was left blank, empty string, or undefined.
- **Solution:** Provide a valid Pexels API key in the node parameters or configure an expression referencing your upstream config node (e.g. `outputs.Config.success.apiKey`).

#### 2. `Pexels image search failed: Search query is required`
- **Cause:** Both the `query` parameter and incoming payload string were empty.
- **Solution:** Ensure the `query` field contains valid search terms, or verify that the upstream node outputs a non-empty string in `outputs.<node>.success.<field>`.

#### 3. `Pexels image search failed: Pexels API error: 401`
- **Cause:** The provided Pexels API key is invalid, revoked, or formatted incorrectly.
- **Solution:** Log into your [Pexels API Dashboard](https://www.pexels.com/api/), verify the key string, and make sure there are no accidental leading or trailing spaces.

#### 4. `Pexels image search failed: Pexels API error: 429`
- **Cause:** You have exceeded Pexels' rate limit (200 requests per hour on free tier).
- **Solution:** Add a **Delay** or **Debounce** node before high-frequency loops, or request a rate limit increase on the Pexels developer portal.

#### 5. No Photos Returned (`total_results: 0`)
- **Cause:** The search query is overly specific or contains unusual character combinations (e.g. `"zxqjvkww9988zzxx"`).
- **Solution:** Broaden the search query terms or provide a fallback query (e.g. `outputs.Function.success.Query || 'nature landscape'`).

---

### Error Reference Table

| Error Message | Cause | Resolution |
|---------------|-------|------------|
| `PEXELS_API_KEY is required` | API key was not provided in configuration | Enter your Pexels API key or bind via expression. |
| `Search query is required` | Query parameter and input payload are both missing | Supply a search keyword string in `query` or pass from upstream node. |
| `Pexels API error: 401` | Unauthorized / Invalid API Key | Check API key validity in your Pexels dashboard. |
| `Pexels API error: 429` | Rate limit exceeded | Implement delays or reduce search frequency. |
| `Pexels API error: 500` | Temporary upstream Pexels server issue | Add retry logic or configure a fallback search node. |

---

### Best Practices

- **Security:** Do not expose raw tokens in public repositories. Pass API keys through expressions or environment parameters.
- **Resolution Selection:** Use `src.large` for website banners and desktop views, `src.medium` for insets, and `src.thumbnail` for preview cards to optimize page loading times.
- **Licensing & Attribution:** While Pexels photos are free to use commercially without mandatory attribution, giving credit using `photographer` and `photographer_url` supports creators and follows community guidelines.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Unsplash Image Search](../unsplash-image-search/en.md) — Search curated high-resolution photography on Unsplash
- [Pixabay Image Search](../pixabay-image-search/en.md) — Find royalty-free stock photos, illustrations, and vector graphics
- [Giphy GIF Search](../giphy-gif-search/en.md) — Search animated GIFs and stickers
- [Google Images Search](../google-images-search/en.md) — Query web-wide image search index
- [Function](../function/en.md) — Transform and format photo metadata into markdown or HTML embeds
- [Discord Bot Send](../discord-bot-send/en.md) — Dispatch rich photo cards and embeds into Discord channels

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|:-------:|:----:|---------|
| `1.0.0` | 2026-09-01 | Initial complete release of Pexels Image Search action node documentation with parameter guides, multi-resolution asset schemas, attribution reference, expression syntax (`outputs.<node>.<output>.<field>`), and real-world workflow patterns. |

<!-- /SECTION: changelog -->

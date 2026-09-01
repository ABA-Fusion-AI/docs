---
node_id: "unsplash-image-search"
title: "Unsplash Image Search"
description: "Search for free, high-resolution stock photos from Unsplash with multi-resolution URLs, creator attribution, and landscape aspect ratios."
category: "Image Search"
subcategory: "media-assets"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:
  - unsplash
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
  - pexels-image-search
  - pixabay-image-search
  - giphy-gif-search
  - google-images-search
  - function
  - webhook
  - slack
  - discord-bot-send
---

<!-- SECTION: header -->
# Unsplash Image Search

> **Category:** Image Search | **Subcategory:** Media Assets | **Type:** Action Node

Search over 5 million free, high-resolution, community-submitted stock photos on [Unsplash](https://unsplash.com/) and retrieve direct image URLs across multiple resolutions (`raw`, `full`, `regular`, `small`, `thumb`) along with creator attribution and download links.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Unsplash Image Search** node allows you to find and download royalty-free photos directly within your automated workflows. Whether you need a hero banner for a blog post, daily photography for social media posts, visual cards for a chat bot, or image assets for creative pipelines, this node fetches curated photos matching any topic or keyword.

Every search query returns a structured list of image results with multiple pre-sized resolutions, photographer credits, and direct links.

```
┌────────────────────────┐
│    Trigger / Webhook   │  (e.g., New blog topic submitted)
└───────────┬────────────┘
            │
            ▼
┌────────────────────────┐
│ Unsplash Image Search  │  (Searches "coffee & croissant vintage")
└───────────┬────────────┘
            │
      ┌─────┴─────┐
      │           │
  [success]    [error]
      │           │
      ▼           ▼
┌───────────┐ ┌───────────┐
│ Downstream│ │   Error   │  (Send 1080px header to CMS / Slack)
│   Nodes   │ │  Handler  │
└───────────┘ └───────────┘
```

### What You Can Do With This Node

- **Search Stock Photography:** Find images using simple keywords or detailed descriptive prompts (e.g. `"minimalist modern architecture"`, `"cyberpunk city"`).
- **Access Multiple Image Sizes:** Get immediate access to web-ready image URLs without needing manual resizing:
  - `regular` (1080px width): Best for website headers, email banners, and blog hero sections.
  - `small` (400px width): Ideal for mobile previews, cards, and article insets.
  - `thumb` (200px width): Perfect for avatars, grid layouts, and thumbnails.
  - `full` / `raw`: High-resolution masters for print or full-screen downloads.
- **Landscape Aspect Ratio:** Results are pre-filtered to landscape orientation, ensuring consistent layout across web banners and social cards.
- **Creator Attribution:** Automatically access photographer names and portfolio URLs to easily credit creators in your content.
- **Dynamic Workflows:** Connect upstream nodes (Forms, Webhooks, AI Generators, Functions) to dynamically pass search topics.

---

### How to Get Your Unsplash Access Key

To use this node, you need a free Unsplash API Access Key:

1. Create a free account at [unsplash.com](https://unsplash.com/).
2. Go to the [Unsplash Developer Portal](https://unsplash.com/developers) and click **Your Apps** → **New Application**.
3. Accept the terms and give your app a name.
4. Copy the **Access Key** displayed in your app dashboard.
5. Paste the key into the `ApiKey` field in the node settings (or pass it from a configuration node).

> [!TIP]
> Free Unsplash accounts start with **50 requests per hour** (Demo Mode), which is great for building and testing. When ready for high volume, you can request Production status (5,000 requests/hour) directly in your Unsplash dashboard.

---

### Common Use Cases

- **Blog & CMS Publishing:** Automatically find and set a featured header image whenever a new blog article is created.
- **Social Media Automation:** Schedule daily posts with matching background photos and photographer credits.
- **Slack & Discord Bots:** Build an interactive image search command (e.g. `/photo nature`) that sends rich image cards to channels.
- **Email Newsletters:** Dynamically insert relevant visual banners based on the email's topic.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Drag the **Unsplash Image Search** node onto your workflow canvas and click it to open the configuration modal.

![Unsplash Node Configuration](icon.svg)

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|:----:|:--------:|:-------:|-------------|
| `Query` | `string` | ✅ Yes | — | The keywords or phrase to search for (e.g. `"coffee & croissant vintage"`, `"cyberpunk neon city"`). Can be static or dynamic. |
| `PerPage` | `number` | ❌ No | `5` | How many photos to return (1 to 30). |
| `ApiKey` | `string` | ✅ Yes | `""` | Your Unsplash Access Key (Client ID). |

---

### Parameter Details & How to Fill Them

#### 1. `Query` (Required)
Enter the topic or keywords you want to search for.
- **Static Text Example:** Type `"sunset over ocean waves"` directly into the field.
- **Dynamic Expression Example:** Click **Expression** and type `outputs.Function.success.query` or `outputs.Webhook.success.body.topic` to search using text from previous nodes.
- **Fallback:** If left empty in the configuration, the node will use any text string passed directly from the input connection.

#### 2. `PerPage` (Optional)
The number of images to return in the results list.
- **Default:** `5`
- **Range:** `1` to `30` images.
- **Tip:** If you only need one header image, set `PerPage` to `1` to keep the response fast and lightweight.

#### 3. `ApiKey` (Required)
Your Unsplash application's Access Key.
- **Static Entry:** Paste your 43-character Access Key directly.
- **Dynamic Entry:** Pass it from an upstream configuration node using `outputs.Config.success.apiKey`.

---

### Switching Between Text and Expression Mode

In the node modal, each parameter has an **Expression** button on the right:

```
┌─────────────────────────────────────────────────────────────┐
│ Parameters                                                  │
│                                                             │
│ Query                                           [Expression]│
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ outputs.Function.success.query                          │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ PerPage (Optional)                              [Expression]│
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ 3                                                       │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ ApiKey (Optional)                               [Expression]│
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ outputs.Config.success.apiKey                           │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

- **Standard Mode:** Type fixed values directly.
- **Expression Mode:** Click **Expression** to reference values from previous nodes using `outputs.<NodeLabel>.<outputPort>.<field>`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input Port

| Port | Description |
|------|-------------|
| `input` | Connect any trigger or upstream action here. If you don't configure a static `Query`, the node will read the incoming text payload from this port as the search topic. |

### Output Ports

| Port | Color | Description |
|------|:-----:|-------------|
| `success` | 🟢 Green | Emitted when photos are found. Delivers the array of image results, resolutions, and photographer details. |
| `error` | 🔴 Red | Emitted if the API key is invalid, the query is empty, or the hourly rate limit is reached. |

---

### What the Output Looks Like

When searching for `query: "coffee & croissant vintage"` with `perPage: 1`, the node outputs:

```json
{
  "success": true,
  "query": "coffee & croissant vintage",
  "results": [
    {
      "id": "eOpewngf68w",
      "url": "https://images.unsplash.com/photo-1509440159596-0249088772ff?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&q=80&w=1080",
      "user": "Nathan Dumlao",
      "user_url": "https://unsplash.com/@nate_dumlao",
      "download_url": "https://unsplash.com/photos/eOpewngf68w/download",
      "urls": {
        "raw": "https://images.unsplash.com/photo-1509440159596-0249088772ff",
        "full": "https://images.unsplash.com/photo-1509440159596-0249088772ff?crop=entropy&cs=srgb&fm=jpg&q=85",
        "regular": "https://images.unsplash.com/photo-1509440159596-0249088772ff?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&q=80&w=1080",
        "small": "https://images.unsplash.com/photo-1509440159596-0249088772ff?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&q=80&w=400",
        "thumb": "https://images.unsplash.com/photo-1509440159596-0249088772ff?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&q=80&w=200"
      }
    }
  ],
  "total_results": 1,
  "note": "Use the 'url' field of each result for image generation"
}
```

---

### How to Use Output Data in Next Nodes

To pass the photos to subsequent nodes in your workflow (such as a CMS, Discord bot, or Email node), use these expression paths:

| Data You Want | Expression to Use | Best For |
|---------------|-------------------|----------|
| **Main Photo URL** | `outputs.UnsplashImageSearch.success.results[0].url` | General web display, header images |
| **1080px Banner Image** | `outputs.UnsplashImageSearch.success.results[0].urls.regular` | Website hero banners, desktop headers |
| **Mobile / Small Image** | `outputs.UnsplashImageSearch.success.results[0].urls.small` | Chat previews, card thumbnails |
| **Grid Thumbnail** | `outputs.UnsplashImageSearch.success.results[0].urls.thumb` | List icons, compact widgets |
| **High-Res Master** | `outputs.UnsplashImageSearch.success.results[0].urls.full` | Full-screen downloads, print media |
| **Photographer Name** | `outputs.UnsplashImageSearch.success.results[0].user` | Image credits & captions |
| **Photographer Profile** | `outputs.UnsplashImageSearch.success.results[0].user_url` | Clickable credit link |
| **Total Images Found** | `outputs.UnsplashImageSearch.success.total_results` | Checking if results exist |

> [!TIP]
> **Getting the First Result:** `results[0]` gives you the first (most relevant) image. If you requested multiple images (`perPage > 1`), you can access `results[1]`, `results[2]`, or loop through all of them with a **For Each** node.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Step-by-Step Usage Examples

### Example 1: Finding a Header Image for a Blog Post

Search for a single landscape header image:

1. Set **Query** to `"coffee & croissant vintage"`.
2. Set **PerPage** to `1`.
3. Set **ApiKey** to your Unsplash Access Key.
4. In your downstream CMS or Webhook node, set the featured image URL to:
   `outputs.UnsplashImageSearch.success.results[0].urls.regular`

---

### Example 2: Sending a Daily Nature Photo to Discord / Slack

Create a workflow that posts an inspiring photo every morning:

1. Add a **Schedule / Cron Trigger** (e.g. daily at 9:00 AM).
2. Connect an **Unsplash Image Search** node with **Query** set to `"scenic mountain sunrise"` and **PerPage** set to `1`.
3. Connect a **Discord / Slack** node and configure the message:
   - **Image URL:** `outputs.UnsplashImageSearch.success.results[0].urls.regular`
   - **Message Text:** `Photo by outputs.UnsplashImageSearch.success.results[0].user on Unsplash`

---

### Example 3: Dynamic Search from an Upstream Form or Webhook

Search for whatever topic a user submits through a form:

1. In the **Form Trigger** or **Function** node, output a field called `topic`.
2. In the **Unsplash Image Search** node, switch **Query** to **Expression** mode and enter:
   `outputs.Function.success.query`
3. The node will search Unsplash dynamically for each new form submission.

---

### Example 4: Creating a Formatted Image Gallery

Use a **Function** node right after Unsplash to format multiple photo results into markdown cards:

```javascript
// Build a markdown list with images and photographer attribution
const gallery = input.results.map((photo, i) => {
  return `### Photo ${i + 1}\n` +
         `![${input.query}](${photo.urls.regular})\n\n` +
         `*Photo by [${photo.user}](${photo.user_url}) on Unsplash*`;
}).join("\n\n---\n\n");

return {
  topic: input.query,
  formattedGallery: gallery,
  firstImageUrl: input.results[0]?.urls?.regular || ""
};
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search and retrieve curated stock images from Unsplash
```

### Common Workflow Setups

#### Setup 1: Automated Article Header Pipeline
```text
[Article Created Trigger]
          ↓
[Unsplash Image Search] (Query: outputs.ArticleTrigger.success.title, PerPage: 1)
          ↓
[WordPress / Ghost Node] (Upload results[0].urls.regular as Featured Image)
```

#### Setup 2: Chatbot On-Demand Photo Command
```text
[Slack Command Trigger: /photo <topic>]
          ↓
[Unsplash Image Search] (Query: outputs.SlackTrigger.success.text, PerPage: 3)
          ↓
[Slack Message Node] (Post photo card with photographer link)
```

#### Setup 3: Marketing Newsletter Visual Builder
```text
[Newsletter Scheduler]
          ↓
[Unsplash Image Search] (Query: "minimalist business workspace", PerPage: 1)
          ↓
[Email Template Node] (Embed results[0].urls.regular in Header)
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Problems & How to Fix Them

#### 1. `UNSPLASH_ACCESS_KEY is required`
- **Why it happens:** The `ApiKey` field was left empty.
- **How to fix:** Paste your Unsplash Access Key into the `ApiKey` parameter field.

#### 2. `Search query is required`
- **Why it happens:** No search query was entered, and no text was received from the input connection.
- **How to fix:** Enter keywords in the `Query` field, or make sure the upstream node is passing a valid text field in `outputs.<node>.success.<field>`.

#### 3. `Unsplash API error: 401`
- **Why it happens:** The Access Key is invalid, deleted, or has extra spaces.
- **How to fix:** Check your Unsplash Developer dashboard and re-copy the Access Key.

#### 4. `Unsplash API error: 403` or `429`
- **Why it happens:** You exceeded the 50 requests/hour limit for Demo Mode applications.
- **How to fix:** Add a delay between requests, or apply for Production Mode in the Unsplash Developer dashboard to get 5,000 requests/hour.

#### 5. No Photos Found (`total_results: 0`)
- **Why it happens:** The search phrase is too long or contains obscure words.
- **How to fix:** Use simpler keywords (e.g. `"vintage coffee"` instead of `"vintage dark roast coffee cup 1970s bar"`).

---

### Error Quick Reference

| Error | What it Means | What to Do |
|-------|---------------|------------|
| `UNSPLASH_ACCESS_KEY is required` | Missing API Key | Enter your Access Key in the node settings. |
| `Search query is required` | Empty search text | Provide search keywords in `Query`. |
| `Unsplash API error: 401` | Unauthorized | Verify your Access Key in Unsplash. |
| `Unsplash API error: 403 / 429` | Hourly limit reached | Wait for the hour to reset or upgrade to Production tier. |
| `Unsplash API error: 500` | Unsplash server issue | Add a retry or use a fallback image node. |

---

### Best Practices

- **Choose the Right Resolution:** Use `urls.regular` (1080px) for websites and headers. It gives crystal-clear quality without slowing down page load times.
- **Credit Creators:** When displaying images, include the photographer's name (`results[0].user`) and link (`results[0].user_url`) to support community artists.
- **Keep Queries Concise:** 2 to 4 descriptive words (e.g. `"modern wooden desk"`) give the best results.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Pexels Image Search](../pexels-image-search/en.md) — Search alternative curated stock photography from Pexels
- [Pixabay Image Search](../pixabay-image-search/en.md) — Search royalty-free photos, illustrations, and vector graphics
- [Giphy GIF Search](../giphy-gif-search/en.md) — Search animated GIFs and reaction stickers
- [Google Images Search](../google-images-search/en.md) — Search the web for images
- [Function](../function/en.md) — Format and customize image URLs before sending to other apps
- [Discord Bot Send](../discord-bot-send/en.md) — Post image cards into Discord channels

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Description |
|:-------:|:----:|-------------|
| `1.0.0` | 2026-09-01 | Initial complete release of Unsplash Image Search documentation with user guides, multi-resolution asset schemas, expression syntax, and workflow patterns. |

<!-- /SECTION: changelog -->

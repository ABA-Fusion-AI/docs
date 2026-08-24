---
node_id: "tv-maze"
title: "TVMaze"
description: "Search TV shows and get metadata & episodes from TVMaze API."
category: "utilities-misc"
subcategory: "tv-shows"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - tv-maze
  - tv-shows
  - series
  - entertainment
  - movies
  - metadata
  - api
related_nodes:
  - rick-and-morty-api
  - jikan
  - http-request
  - ai-chat
  - discord-bot-send
---

<!-- SECTION: header -->
# TVMaze

> **Category:** Utilities & Misc | **Subcategory:** TV Shows | **Type:** Action Node

Search international TV shows, series schedules, cast details, poster artwork, and cross-platform external IDs using the TVMaze API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **TVMaze** node connects to the public [TVMaze](https://www.tvmaze.com/) API to search for television shows, series, and streaming titles.

It retrieves complete show metadata including user ratings, genres, air days/times, broadcast networks (e.g., AMC, HBO, BBC) or web channels (e.g., Netflix, Hulu), runtime, high-resolution poster artwork, and external IDs (IMDb, TheTVDB, TVRage).

### Key Features

- **Global TV Show Search:** Search across thousands of global scripted, animated, documentary, and reality TV series.
- **Rich Show Metadata:** Access premiere dates, ratings, broadcast schedules, summary synopses, and genre categorizations.
- **Poster & Artwork URLs:** Returns direct image links for both medium and high-resolution original posters (`image.medium`, `image.original`).
- **External Database Identifiers:** Includes IMDb (`imdb`), TheTVDB (`thetvdb`), and TVRage (`tvrage`) IDs to link show data across entertainment platforms.
- **Keyless Public API:** Free to use without needing an API key or account registration.
- **Dynamic Search Support:** Accepts queries configured directly in parameters or passed from previous workflow triggers (chatbots, webhooks, or scheduled jobs).

### Processing Flow

```text
Workflow Trigger / Chatbot Message
  ↓
Read Query (parameter `q` or incoming payload)
  ↓
Validate Search Query is non-empty
  ↓
Call TVMaze Search API (https://api.tvmaze.com/search/shows?q=...)
  ↓
Parse Show Metadata, Ratings, Schedules, Posters & External IDs
  ↓
Emit Structured Shows Array { success, query, total_results, shows }
```

### Use Cases

- **Entertainment Chatbots:** Build a Telegram, Discord, or WhatsApp bot that lets users search TV shows and receive synopsis summaries and posters.
- **TV Watchlist & Airing Alerts:** Query upcoming broadcast schedules and trigger notifications before a new episode airs.
- **Media Server & Library Enrichment:** Enrich local Plex, Jellyfin, or Kodi media collections with IMDb IDs and official poster images.
- **AI Recommendation Engine:** Feed show summaries and genres to an [AI Chat](../ai-chat/en.md) node to suggest similar series based on user preferences.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `q` | `string` | Yes | — | The TV show title or keyword to search for (e.g. `Breaking Bad`, `Friends`, `Batman`, `Prison Break`). |

### Search Guidance

- Searches are case-insensitive and match partial names.
- For popular franchises (e.g., `Batman` or `Star Wars`), the API returns multiple matching series sorted by relevance score.

### Default Configuration

```json
{
  "q": "Breaking Bad"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `unknown` | Incoming workflow trigger or text payload. Used as the search query if the `q` parameter is not provided. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Returned on successful search, containing an array of matched TV shows with rich metadata and poster links. |
| `error` | `object` | Returned if the query is missing or network/API errors occur. |

### Key Show Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | `number` | Unique TVMaze Show ID (e.g. `169`). Can be used to query specific episode endpoints. |
| `name` | `string` | Official show title. |
| `type` | `string` | Show category (e.g. `"Scripted"`, `"Animation"`, `"Reality"`). |
| `language` | `string` | Primary broadcast language (e.g. `"English"`, `"Japanese"`). |
| `genres` | `string[]` | Array of genres (e.g. `["Drama", "Crime", "Thriller"]`). |
| `status` | `string` | Current production status (e.g. `"Ended"`, `"Running"`). |
| `runtime` | `number` | Average episode runtime in minutes. |
| `premiered` | `string` | Initial premiere date (`YYYY-MM-DD`). |
| `rating` | `number` | Average viewer rating on a 10-point scale. |
| `schedule` | `object` | Air time and days of the week (`time`, `days`). |
| `network` | `object` | Broadcast network information (`name`, `country`). |
| `web_channel` | `object` | Streaming platform information (e.g. Netflix, Prime Video). |
| `externals` | `object` | Cross-platform IDs: `imdb` (e.g. `"tt0903747"`), `thetvdb`, and `tvrage`. |
| `image` | `object` | URLs to show posters: `medium` (210x295) and `original` high-res. |
| `summary` | `string` | HTML-formatted synopsis and summary description. |
| `score` | `number` | Search relevance score. |

### Successful Response Example

```json
{
  "success": true,
  "query": "Breaking Bad",
  "total_results": 1,
  "shows": [
    {
      "id": 169,
      "name": "Breaking Bad",
      "url": "https://www.tvmaze.com/shows/169/breaking-bad",
      "type": "Scripted",
      "language": "English",
      "genres": ["Drama", "Crime", "Thriller"],
      "status": "Ended",
      "runtime": 60,
      "premiered": "2008-01-20",
      "official_site": "http://www.amc.com/shows/breaking-bad",
      "schedule": {
        "time": "22:00",
        "days": ["Sunday"]
      },
      "rating": 9.2,
      "network": {
        "id": 20,
        "name": "AMC",
        "country": {
          "name": "United States",
          "code": "US",
          "timezone": "America/New_York"
        }
      },
      "web_channel": null,
      "externals": {
        "tvrage": 18164,
        "thetvdb": 81189,
        "imdb": "tt0903747"
      },
      "image": {
        "medium": "https://static.tvmaze.com/uploads/images/medium_portrait/0/2400.jpg",
        "original": "https://static.tvmaze.com/uploads/images/original_untouched/0/2400.jpg"
      },
      "summary": "<p><b>Breaking Bad</b> follows protagonist Walter White, a chemistry teacher who is diagnosed with cancer...</p>",
      "updated": 1704795240,
      "score": 30.12,
      "tvmaze_url": "https://www.tvmaze.com/shows/169/breaking-bad"
    }
  ],
  "note": "Returns TV show metadata. Use show ID to get episodes via /shows/{id}/episodes"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Search Breaking Bad

```json
{
  "q": "Breaking Bad"
}
```

### Example 2: Search Friends Comedy Series

```json
{
  "q": "Friends"
}
```

### Example 3: Search Batman Franchise Shows

```json
{
  "q": "Batman"
}
```

### Example 4: Search Prison Break

```json
{
  "q": "Prison Break"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search TV shows and retrieve series metadata using TVMaze
```

### Common Workflow Patterns

- **Discord TV Info Bot:** Discord Trigger (`!tv Breaking Bad`) → TVMaze (`q: {{ $trigger.message }}`) → Discord Bot Send (Post poster, IMDb link, rating, and synopsis).
- **Episode Guide Fetcher:** Manual Trigger → TVMaze (Search Show) → HTTP Request (`https://api.tvmaze.com/shows/{{ $node["TVMaze"].shows[0].id }}/episodes`) → Google Sheets (Log Episodes).
- **AI Recommendation Pipeline:** Webhook (User favorite shows) → TVMaze → AI Chat (Analyze genres & suggest next watchlist) → Email Send.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Error: "Search query is required"

**Cause:** The `q` parameter was left empty and no string data was received from the upstream workflow node.

**Solution:** Provide a show name in the `Q` parameter (e.g. `Friends` or `Prison Break`).

### Empty shows array (`total_results: 0`)

**Cause:** The query keyword did not match any television titles in the TVMaze catalog.

**Solution:** Check the spelling of the show title or try using a broader search keyword.

### API rate limit / server error

**Cause:** TVMaze allows up to 20 calls per 10 seconds per IP address. Exceeding this rate may result in HTTP 429 errors.

**Solution:** Add delays between batch executions or throttle request bursts.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Rick and Morty API](../rick-and-morty-api/en.md) - Explore characters and episodes from Rick and Morty
- [Jikan](../jikan/en.md) - Query anime and manga series from MyAnimeList
- [HTTP Request](../http-request/en.md) - Make custom API calls to TVMaze episode or cast endpoints
- [Discord Bot Send](../discord-bot-send/en.md) - Post TV show cards and posters to Discord channels

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for TVMaze node |

<!-- /SECTION: changelog -->

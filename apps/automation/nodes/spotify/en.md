---
node_id: "spotify"
title: "Spotify"
description: "Spotify Web API: search tracks, get track/artist/album info, and user playlists."
category: "peer-only"
subcategory: "Integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-08-07"
author: "Fusion Team"
tags:
  - spotify
  - music
  - integration
  - peer-only
  - api
related_nodes:
  - http-request
  - function
  - ai-chat
---

<!-- SECTION: overview -->
# Spotify

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Interact with the [Spotify Web API](https://developer.spotify.com/documentation/web-api) to search for tracks, retrieve track, artist, and album metadata, and list a user's public playlists — all from within a Fusion workflow.

### Use Cases

- **Music Discovery Pipelines:** Search for tracks by keyword and pass results to an AI node for recommendations or playlist curation.
- **Artist & Album Research:** Retrieve detailed metadata (genres, popularity, followers) for artists or albums programmatically.
- **Playlist Management:** Fetch a user's public playlists to display, sync, or analyze their listening habits.
- **Content Enrichment:** Enrich external datasets (e.g., a song title from a form) with full Spotify track data including preview URLs and album art.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `accessToken` | `string` | Yes | — | Spotify OAuth Bearer token. Obtained via the [Spotify Authorization flow](https://developer.spotify.com/documentation/web-api/concepts/authorization). Expires after ~1 hour. |
| `operation` | `enum` | Yes | `searchTracks` | The action to perform. See available operations below. |
| `query` | `string` | Conditional | — | Search terms. Only used with `searchTracks`. Supports Spotify search operators (e.g., `artist:Daft Punk track:"Get Lucky"`). |
| `trackId` | `string` | Conditional | — | Spotify Track ID. Only used with `getTrack`. |
| `userId` | `string` | Conditional | — | Spotify User ID. Only used with `getUserPlaylists`. |
| `artistId` | `string` | Conditional | — | Spotify Artist ID. Only used with `getArtist`. |
| `albumId` | `string` | Conditional | — | Spotify Album ID. Only used with `getAlbum`. |
| `market` | `string` | No | — | ISO 3166-1 country code (e.g., `US`, `FR`, `MA`). Filters results by market availability. Used with `searchTracks`, `getTrack`, and `getAlbum`. |
| `limit` | `number` | No | `20` | Number of results to return (1–50). Only used with `searchTracks` and `getUserPlaylists`. |

### Available Operations

| Operation | Description | Required Parameter |
|-----------|-------------|-------------------|
| `searchTracks` | Search the Spotify catalog for tracks matching a query. | `query` |
| `getTrack` | Retrieve full metadata for a specific track by its Spotify ID. | `trackId` |
| `getUserPlaylists` | List the public playlists of a specific Spotify user. | `userId` |
| `getArtist` | Retrieve detailed metadata for an artist by their Spotify ID. | `artistId` |
| `getAlbum` | Retrieve full metadata for an album by its Spotify ID. | `albumId` |

### Finding Spotify IDs

Spotify IDs appear in the share URL of any track, artist, or album:

```
https://open.spotify.com/track/4uLU6hMCjMI75M1A2tKUQC
                                   ^^^^^^^^^^^^^^^^^^^^^^^^
                                   This is the Track ID
```

The same pattern applies to artist and album URLs.

### Search Query Syntax (`searchTracks`)

The `query` field supports Spotify's search operators:

| Example | Description |
|---------|-------------|
| `Daft Punk` | Plain keyword search |
| `artist:Daft Punk` | Filter by artist name |
| `track:"Get Lucky"` | Filter by exact track title |
| `artist:Daft Punk track:"Get Lucky"` | Combined filter |
| `genre:pop year:2020` | Filter by genre and release year |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Data and configuration supplied by the preceding workflow node. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The Spotify API response for the selected operation. |
| `error` | `Error` | Emitted on validation errors, authentication failures, network issues, or Spotify API errors. |

### Output Examples

#### `searchTracks`

```json
{
  "tracks": {
    "href": "https://api.spotify.com/v1/search?q=Daft+Punk&type=track&limit=20",
    "total": 1523,
    "items": [
      {
        "id": "4uLU6hMCjMI75M1A2tKUQC",
        "name": "Get Lucky",
        "duration_ms": 369560,
        "popularity": 81,
        "preview_url": "https://p.scdn.co/mp3-preview/...",
        "artists": [{ "id": "4tZwfgrHOc3mvqYlEYSvVi", "name": "Daft Punk" }],
        "album": { "id": "2noRn2Aes5aoNVsU6iWThc", "name": "Random Access Memories" }
      }
    ]
  }
}
```

#### `getArtist`

```json
{
  "id": "4tZwfgrHOc3mvqYlEYSvVi",
  "name": "Daft Punk",
  "genres": ["electro", "french house"],
  "followers": { "total": 12450000 },
  "popularity": 85,
  "images": [{ "url": "https://i.scdn.co/image/..." }],
  "external_urls": { "spotify": "https://open.spotify.com/artist/4tZwfgrHOc3mvqYlEYSvVi" }
}
```

#### `getUserPlaylists`

```json
{
  "total": 5,
  "items": [
    {
      "id": "37i9dQZF1DXcBWIGoYBM5M",
      "name": "Today's Top Hits",
      "public": true,
      "tracks": { "total": 50 },
      "external_urls": { "spotify": "https://open.spotify.com/playlist/37i9dQZF1DXcBWIGoYBM5M" }
    }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Spotify in a Workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **Spotify Node:** Executes the configured operation against the Spotify Web API using the provided `accessToken`.
3. **Log Node:** Displays the API response.

### Common Patterns

- **Track Search → AI Curation:** Search for tracks with `searchTracks`, pass the results to an AI Chat node, and generate a themed playlist description or recommendation.
- **Artist Research Pipeline:** Use `getArtist` to retrieve metadata, then format and push it to a Google Sheet or Notion database.
- **User Playlist Sync:** Fetch a user's playlists with `getUserPlaylists` and sync them to an internal content library.
- **Token Refresh Automation:** Combine with an HTTP Request node to refresh the OAuth access token before it expires (Spotify tokens last ~1 hour), and store the new token in a workflow variable.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store your `accessToken` in Fusion's **Secrets** system. Do not paste tokens directly into workflow parameters or commit them to version control.

Spotify OAuth access tokens:
- Expire after **~1 hour**.
- Must be refreshed using the [Spotify Token Refresh flow](https://developer.spotify.com/documentation/web-api/tutorials/refreshing-tokens) and a valid `refresh_token`.
- Require appropriate **OAuth scopes** for certain operations (e.g., `playlist-read-private` for private playlists).

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Unauthorized` (HTTP 401)
- **Cause:** The `accessToken` is missing, malformed, or expired.
- **Solution:** Generate a fresh access token from the Spotify Developer Console or via your OAuth refresh flow. Tokens expire after ~1 hour.

#### `Forbidden` (HTTP 403)
- **Cause:** The token does not have the required OAuth scope for the requested operation.
- **Solution:** Re-authorize with the correct scopes. For example, `playlist-read-private` is needed to access private playlists.

#### `query is required for searchTracks`
- **Cause:** The `query` field is empty or missing when `operation` is `searchTracks`.
- **Solution:** Provide a non-empty search string in the `query` parameter.

#### `limit must be an integer between 1 and 50`
- **Cause:** The `limit` value is out of range or is not a whole number.
- **Solution:** Set `limit` to an integer between `1` and `50`.

#### Empty results on `searchTracks`
- **Cause:** The query returned no matches, or the `market` filter excludes all available tracks.
- **Solution:** Broaden the search query or remove the `market` filter to search the global catalog.

### Error Codes Summary

| HTTP Code | Meaning | Solution |
|-----------|---------|----------|
| 401 | Unauthorized | Refresh or regenerate the access token |
| 403 | Forbidden | Check OAuth scopes |
| 404 | Not Found | Verify the Track/Artist/Album/User ID |
| 429 | Rate Limited | Add a Delay node between repeated Spotify calls |
| 500 / 503 | Server Error | Retry after a short delay |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](./http-request.md) – Call Spotify's OAuth token endpoint to refresh access tokens
- [Function](./function.md) – Build dynamic search queries or extract specific fields from the response
- [AI Chat](./ai-chat.md) – Generate playlist recommendations from Spotify search results

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-07 | Initial documentation |

<!-- /SECTION: changelog -->

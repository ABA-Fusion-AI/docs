---
node_id: "om-db"
title: "OMDb API"
description: "Search for movie and TV series information using the OMDb API."
category: "Databases & Memory"
subcategory: "Data Platforms"
version: "1.0.0"
language: "en"
last_updated: "2026-08-26"
author: "Fusion Team"
tags:
  - omdb
  - om-db
  - movies
  - cinema
  - films
  - tv-shows
  - series
  - imdb
  - ratings
  - api
  - entertainment
related_nodes:
  - http-request
  - ai-chat
  - function
  - discord-bot-send
  - slack
  - filter-array
  - log
---

<!-- SECTION: header -->
# OMDb API

> **Category:** Databases & Memory | **Subcategory:** Data Platforms | **Type:** Action Node

Search for detailed movie, television series, and episode metadata using the [OMDb (Open Movie Database) API](https://www.omdbapi.com/).

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **OMDb API** node connects workflows directly to the Open Movie Database (OMDb) REST service. It retrieves comprehensive entertainment metadata including movie titles, release years, age ratings, runtime, genre classifications, directors, writers, cast members, plot summaries, posters, IMDb ratings, Rotten Tomatoes scores, box office earnings, and unique IMDb IDs (`tt...`).

Whether you are building automated entertainment chatbots, media monitoring pipelines, movie recommendation systems, or catalog enrichment workflows, this node provides structured movie data in clean JSON format.

### Key Features

- **Movie & Series Search:** Look up metadata for feature films, TV shows, and series episodes by title.
- **Year-Specific Filtering:** Refine searches by specifying a release year (e.g. `1997`, `2022`) to disambiguate remakes and same-name titles.
- **Comprehensive Metadata:** Returns full details including IMDb rating, Metascore, Rotten Tomatoes ratings, awards, box office numbers, and poster image URLs.
- **Dynamic & Expression Support:** Supply static parameters or feed dynamic movie titles from upstream nodes (such as chat triggers, webhooks, or AI agents).
- **Flexible Authentication:** Supports custom user API keys (`apiKey`) from `omdbapi.com` or platform defaults.

### Use Cases

- **Entertainment Chatbots:** Build a Telegram, WhatsApp, or Discord bot where users ask about any movie and receive plot summaries, ratings, and poster cards.
- **AI-Powered Recommendation Systems:** Pass OMDb movie plots and genres into [AI Chat](../ai-chat/en.md) nodes to generate personalized "movies like this" recommendations.
- **Media Catalog Enrichment:** Ingest a list of movie titles from a database or CSV and enrich each record with official ratings, director names, and release dates.
- **Weekly Movie Night Automations:** Poll popular movie titles on a schedule and post reviews and posters to team communication channels (Slack, Microsoft Teams).

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `title` | `string` | ✅ Yes | — | Title of the movie, TV series, or episode to search for (e.g. `titanic`, `batman`, `interstellar`, `breaking bad`). |
| `year` | `string` | ❌ No | — | Release year to narrow down the search (e.g. `1997`, `2022`, `2014`). |
| `apiKey` | `string` | ❌ No | — | Your OMDb API key obtained from [omdbapi.com](https://www.omdbapi.com/apikey.aspx). |

---

### Detailed Parameter Descriptions

#### `title` (Required)
The name of the movie, TV series, or episode.
- Matches partial and full titles.
- Case-insensitive (e.g. `Titanic`, `titanic`, `TITANIC`).
- Supports expressions (e.g. `{{input.movieTitle}}` or `{{outputs.chat.message}}`).

#### `year` (Optional)
The 4-digit release year of the film or series. Useful when multiple movies share the same title (for example, *Batman* 1989 vs *The Batman* 2022).
- Example values: `1997`, `2014`, `2022`.

#### `apiKey` (Optional)
Your personal OMDb API key. You can generate a free API key at [omdbapi.com/apikey.aspx](https://www.omdbapi.com/apikey.aspx).
- If your environment provides a global API key, this field can be left blank.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow trigger or data payload that initiates the search. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the movie search succeeds. Contains the full movie metadata object. |
| `error` | `Error` | Emitted when network request fails, movie is not found, or API key is invalid. |

---

### Output Data Structure Example

```json
{
  "Title": "Titanic",
  "Year": "1997",
  "Rated": "PG-13",
  "Released": "19 Dec 1997",
  "Runtime": "194 min",
  "Genre": "Drama, Romance",
  "Director": "James Cameron",
  "Writer": "James Cameron",
  "Actors": "Leonardo DiCaprio, Kate Winslet, Billy Zane",
  "Plot": "A seventeen-year-old aristocrat falls in love with a kind but poor artist aboard the luxurious, ill-fated R.M.S. Titanic.",
  "Language": "English, Swedish, Italian, French",
  "Country": "United States, Mexico",
  "Awards": "Won 11 Oscars. 126 wins & 83 nominations total",
  "Poster": "https://m.media-amazon.com/images/M/MV5BYzYyN2FiZmUtYWYzMy00MzViLWJkZTMtOGY1ZjgzNWMwN2YxXkEyXkFqcGc@._V1_SX300.jpg",
  "Ratings": [
    {
      "Source": "Internet Movie Database",
      "Value": "7.9/10"
    },
    {
      "Source": "Rotten Tomatoes",
      "Value": "88%"
    },
    {
      "Source": "Metacritic",
      "Value": "75/100"
    }
  ],
  "Metascore": "75",
  "imdbRating": "7.9",
  "imdbVotes": "1,274,832",
  "imdbID": "tt0120338",
  "Type": "movie",
  "DVD": "N/A",
  "BoxOffice": "$674,292,608",
  "Production": "N/A",
  "Website": "N/A",
  "Response": "True"
}
```

---

### Output Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `Title` | `string` | Official title of the film or TV show |
| `Year` | `string` | Release year or span of years for series (e.g. `2008–2013`) |
| `Rated` | `string` | Age certification rating (e.g. `PG-13`, `R`, `TV-MA`) |
| `Released` | `string` | Full release date (e.g. `19 Dec 1997`) |
| `Runtime` | `string` | Total duration (e.g. `194 min` or `45 min`) |
| `Genre` | `string` | Comma-separated genre classifications |
| `Director` | `string` | Director name(s) |
| `Writer` | `string` | Screenwriter(s) |
| `Actors` | `string` | Main cast members |
| `Plot` | `string` | Summary narrative / synopsis of the story |
| `Language` | `string` | Original audio language(s) |
| `Country` | `string` | Country or countries of production |
| `Awards` | `string` | Summary of Academy Awards, nominations, and festival wins |
| `Poster` | `string` | Web URL to the official movie poster image |
| `Ratings` | `array` | Array of review scores from IMDb, Rotten Tomatoes, and Metacritic |
| `Metascore` | `string` | Metacritic aggregate score out of 100 |
| `imdbRating` | `string` | IMDb user score out of 10 (e.g. `7.9`) |
| `imdbVotes` | `string` | Number of IMDb user ratings |
| `imdbID` | `string` | Canonical IMDb identifier (e.g. `tt0120338`) |
| `Type` | `string` | Entity type: `movie`, `series`, or `episode` |
| `BoxOffice` | `string` | Worldwide or domestic box office gross revenue |
| `Response` | `string` | Status flag (`"True"` on success) |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Search by Movie Title and Release Year

Look up the 1997 classic *Titanic*:

**Configuration:**
- **Title:** `titanic`
- **Year:** `1997`

---

### Example 2: Search Specific Film Edition (e.g. The Batman 2022)

Disambiguate the 2022 release from previous Batman movies:

**Configuration:**
- **Title:** `batman`
- **Year:** `2022`

---

### Example 3: Search TV Series

Look up an acclaimed television series without specifying a year:

**Configuration:**
- **Title:** `breaking bad`

---

### Example 4: Dynamic Title Lookup from Chat Message

Extract the movie title dynamically from an incoming chat trigger:

**Configuration:**
- **Title:** `{{input.message}}`
- **Year:** `{{input.year}}`

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search movie and series information using OMDb API
```

### Common Patterns

- **Movie Info Chatbot:** Chat Trigger (e.g. Telegram / Discord) → OMDb API (`title: input.text`) → Function (Format markdown card with poster) → Reply Node.
- **AI Recommendation Engine:** Manual Trigger → OMDb API (`title: "interstellar"`) → AI Chat (Prompt: *"Recommend 3 films with similar cosmic and emotional themes"*) → Slack Send.
- **Database Film Catalog Enrichment:** Database Trigger (New movie record) → OMDb API → Postgres Action (Update record with `imdbRating`, `BoxOffice`, and `Poster`).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Movie not found!`
- **Cause:** The requested title does not match any entry in the OMDb database, or the release year is incorrect.
- **Solution:** Verify the spelling of the movie title. Try searching without the `year` parameter first to see if the title matches.

#### `Invalid API key!` or `No API key provided!`
- **Cause:** The `apiKey` parameter was left blank and no default key was found, or the provided key is invalid / expired.
- **Solution:** Register for a free API key at [omdbapi.com/apikey.aspx](https://www.omdbapi.com/apikey.aspx) and enter it into the `ApiKey` parameter.

#### `Request limit reached!`
- **Cause:** The free tier of OMDb allows up to 1,000 daily requests per API key.
- **Solution:** Upgrade your OMDb tier or implement rate-limiting / caching in your workflow using a [Cache](../cache/en.md) node.

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Movie not found!` | Title not found in database | Check title spelling or remove Year filter |
| `Invalid API key!` | The provided API key is invalid | Check API key entered in node configuration |
| `No API key provided!` | API key parameter missing | Provide a valid OMDb API key |
| `Too many results.` | The search query was too ambiguous | Provide a more specific movie title |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [TheMealDB](../the-meal-db/en.md) — Search recipes and culinary database
- [HTTP Request](../http-request/en.md) — Custom HTTP requests to movie or entertainment APIs
- [AI Chat](../ai-chat/en.md) — Analyze film plots and generate movie recommendations
- [Discord Bot Send](../discord-bot-send/en.md) — Send movie review cards and posters to Discord
- [Slack](../slack/en.md) — Broadcast movie night announcements to Slack channels

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-26 | Initial release of OMDb API Action Node |

<!-- /SECTION: changelog -->

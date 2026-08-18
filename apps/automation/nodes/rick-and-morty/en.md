---
node_id: "rick-and-morty-api"
title: "Rick and Morty API"
description: "Get character information from the Rick and Morty API."
category: "Entertainment"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:

- rick-and-morty
- characters
- entertainment
- tv-show
- api
- pop-culture

related_nodes:
- function
- if
- http-request

---

# Rick and Morty API

> **Category:** entertainment-nodes | **Type:** Action Node

Fetch **character information** from the public **Rick and Morty API**.

The **Rick and Morty API** node queries the character endpoint with optional name and status filters, paginated, and returns a normalized list of matching characters alongside pagination info.

### Supported Features

- Paginated character listing
- Filter by character name (partial match, as supported by the underlying API)
- Filter by character status (`alive`, `dead`, `unknown`)
- Case-insensitive, validated status filtering
- Normalized character result objects
- Pagination metadata pass-through (`info`)

### Use Cases

- Build a character lookup or trivia bot
- Search for characters by name in a fan-content workflow
- Filter characters by status for a themed dataset (e.g. only "alive" characters)
- Paginate through the full character list for a data export or archive
- Feed character data into a `Function` node for further formatting or display

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `page` | `number` | ❌ No | `1` | Page number of results to fetch. |
| `name` | `string` | ❌ No | `""` | Filter characters by name. Empty string means no filter. |
| `status` | `string` | ❌ No | `""` | Filter characters by status: `alive`, `dead`, or `unknown` (case-insensitive). Invalid or empty values are ignored. |

---

## Filter Behavior

- `page`: only appended to the request if greater than `0`.
- `name`: only appended to the request if non-empty.
- `status`: only appended if, after lowercasing, it matches one of `alive`, `dead`, or `unknown`. Any other value (including typos or unsupported statuses) is **silently ignored** — no error is raised, the filter is simply dropped.

If no filters are set, the node fetches the default first page of all characters.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` on a successful call. |
| `info` | `object` | Pagination metadata from the Rick and Morty API (`count`, `pages`, `next`, `prev`). |
| `results` | `array` | List of normalized character objects (see below). |
| `total_results` | `number` | Number of characters returned in `results` for this page. |

### Character Object Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | `number` | Character ID. |
| `name` | `string` | Character name. |
| `status` | `string` | `Alive`, `Dead`, or `unknown`. |
| `species` | `string` | Character species. |
| `type` | `string` | Character subtype/variant, if any. |
| `gender` | `string` | Character gender. |
| `origin` | `object` | Origin location (`name`, `url`). |
| `location` | `object` | Last known location (`name`, `url`). |
| `image` | `string` | URL to the character's image. |
| `episode` | `string[]` | List of episode URLs the character appears in. |
| `url` | `string` | API URL for this character. |
| `created` | `string` | ISO timestamp of when the character was added to the API. |

---

## Output Example

```json
{
  "success": true,
  "info": {
    "count": 826,
    "pages": 42,
    "next": "https://rickandmortyapi.com/api/character?page=2",
    "prev": null
  },
  "results": [
    {
      "id": 1,
      "name": "Rick Sanchez",
      "status": "Alive",
      "species": "Human",
      "type": "",
      "gender": "Male",
      "origin": { "name": "Earth (C-137)", "url": "https://rickandmortyapi.com/api/location/1" },
      "location": { "name": "Citadel of Ricks", "url": "https://rickandmortyapi.com/api/location/3" },
      "image": "https://rickandmortyapi.com/api/character/avatar/1.jpeg",
      "episode": ["https://rickandmortyapi.com/api/episode/1"],
      "url": "https://rickandmortyapi.com/api/character/1",
      "created": "2017-11-04T18:48:46.250Z"
    }
  ],
  "total_results": 1
}
```

---

## Configuration Examples

### Default (First Page, No Filters)

```json
{
  "page": 1
}
```

### Search by Name

```json
{
  "page": 1,
  "name": "Rick"
}
```

### Filter by Status

```json
{
  "page": 1,
  "status": "alive"
}
```

### Combined Filters, Paginated

```json
{
  "page": 2,
  "name": "Morty",
  "status": "alive"
}
```

---

## Workflow Integration

### Sample Workflow: Fetch Characters → Function

```json
{
  "nodes": [
    {
      "id": "rick-morty-search",
      "type": "rick-and-morty-api",
      "config": {
        "name": "Rick",
        "status": "alive"
      }
    },
    {
      "id": "format-characters",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Paginate → If → Loop

```json
{
  "nodes": [
    {
      "id": "rick-morty-page",
      "type": "rick-and-morty-api",
      "config": {
        "page": 1
      }
    },
    {
      "id": "check-next-page",
      "type": "if"
    }
  ]
}
```

### Common Patterns

- Rick and Morty API → Function → Notification — trivia or fun-fact bot
- Rick and Morty API (paginated) → Function → Database — full character archive
- Rick and Morty API → If (status filter) → Chart/visualization pipeline

---

## Error Handling

### API Error

```text
Rick and Morty API error: <status>
```

Raised when the API returns a non-OK HTTP status (e.g. `404` when no characters match the given filters).

### Wrapped Failure

```text
Rick and Morty API request failed: <underlying error message>
```

All errors, including the API error above, are re-thrown wrapped in this message from `handleTick`.

---

## Troubleshooting

### "Rick and Morty API request failed: Rick and Morty API error: 404"

**Cause**

No characters match the given `name`/`status` combination — the Rick and Morty API returns `404` for zero-result searches rather than an empty list.

**Solution**

Broaden or correct the `name` filter, or remove the `status` filter to confirm the character exists at all.

---

### `status` Filter Seems to Have No Effect

**Cause**

The provided `status` value did not match `alive`, `dead`, or `unknown` (case-insensitive) and was silently dropped rather than raising an error.

**Solution**

Use one of the three exact supported values: `alive`, `dead`, or `unknown`.

---

### Fewer Results Than Expected on a Page

**Cause**

The Rick and Morty API paginates at a fixed page size (20 per page); `total_results` reflects only the current page's `results.length`, not the full match count — use `info.count` for the true total.

**Solution**

Read `info.count` and `info.pages` to determine the full result set size, and iterate `page` accordingly.

---

## Security

The node performs outbound HTTP requests to the public Rick and Morty API (`rickandmortyapi.com`).

No API key or authentication credential is required.

---

## Notes

The node returns a normalized character list plus the API's native pagination metadata under `info`.

The node does not:

- Support filtering by species, gender, or type (only `name` and `status`)
- Support the API's `/location` or `/episode` endpoints
- Cache results between calls
- Automatically paginate through all results in a single call
- Retry on failure

It is intended to provide simple, filtered access to Rick and Morty character data for downstream entertainment and pop-culture workflows.

---

## Related

- [Function](./function.md) – Transform, filter, or format character results
- [If](./if.md) – Route workflows based on character status or pagination
- [HTTP Request](./http-request.md) – Query the API's `/location` or `/episode` endpoints directly

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-18 | Initial release |
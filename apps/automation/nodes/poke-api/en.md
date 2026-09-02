---
node_id: "pokeapi"
title: "PokéAPI"
description: "Get Pokémon information using the PokéAPI with @sherwinski/pokeapi-ts library"
category: "Security & Networking"
subcategory: "APIs & Protocols"
version: "1.0.0"
language: "en"
last_updated: "2026-09-02"
author: "Fusion Team"
tags:
  - pokeapi
  - pokemon
  - api
  - search
  - generation
related_nodes:
  - graphql
  - http-request
  - log
---

<!-- SECTION: header -->

# PokéAPI

> **Category:** Security & Networking | **Subcategory:** APIs & Protocols | **Type:** Action Node

Get Pokémon and generation information using the PokéAPI through the `@sherwinski/pokeapi-ts` library.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **PokéAPI** node provides access to Pokémon and generation data using the PokéAPI.

It supports retrieving a Pokémon by name or ID, searching the Pokémon list with pagination, and retrieving a Pokémon generation by name or ID.

The node does not require an API key.

### Key Features

- **Get Pokémon:** Retrieve detailed Pokémon information by name or ID
- **Search Pokémon:** List Pokémon with configurable limit and offset
- **Get Generation:** Retrieve generation information by name or ID
- **Pagination:** Control search result size and offset
- **Error Handling:** Wrap PokéAPI errors in a consistent node error message
- **No Authentication Required:** No API key or credentials are required

### Supported Operations

The node supports three operations:

1. `getPokemon`
2. `searchPokemon`
3. `getGeneration`

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Operation

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `string` | ✅ Yes | `getPokemon` | Selects the PokéAPI operation to execute |

Available values:

```text
getPokemon
searchPokemon
getGeneration
```

---

### Get Pokémon

Use `getPokemon` to retrieve detailed information about one Pokémon.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `nameOrId` | `string` | Conditional | Pokémon name or numeric ID |

Examples:

```text
pikachu
```

or:

```text
25
```

Example configuration:

```text
Operation: getPokemon
NameOrId: pikachu
```

The operation calls the Pokémon lookup method using the configured name or ID.

A request for:

```text
pikachu
```

returns Pokémon data including:

```text
id: 25
name: pikachu
```

A request for:

```text
1
```

returns:

```text
id: 1
name: bulbasaur
```

---

### Search Pokémon

Use `searchPokemon` to retrieve a paginated Pokémon list.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `searchLimit` | `number` | ❌ No | `20` | Number of results requested |
| `searchOffset` | `number` | ❌ No | `0` | Pagination offset |

Example:

```text
Operation: searchPokemon
Search Limit: 5
Search Offset: 0
```

This returns five results starting from the beginning of the Pokémon list.

Example pagination response:

```text
results: 5 items
previous: null
next: ...offset=5&limit=5
```

Using:

```text
Search Limit: 3
Search Offset: 5
```

returns three items starting after the first five Pokémon.

The validated response started with:

```text
charizard
squirtle
wartortle
```

and returned pagination links using:

```text
offset=8&limit=3
```

for the next page.

---

### Search Limit Fallback

The node applies fallback logic when `searchLimit` is `0`.

The runtime uses:

```text
searchLimit || 20
```

Therefore:

```text
Search Limit: 0
```

is treated as:

```text
Search Limit: 20
```

This behavior was confirmed during functional testing.

---

### Get Generation

Use `getGeneration` to retrieve a Pokémon generation by name or ID.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `nameOrId` | `string` | Conditional | Generation name or numeric ID |

Example with ID:

```text
Operation: getGeneration
NameOrId: 1
```

returns:

```text
id: 1
name: generation-i
main_region:
  name: kanto
```

Example with name:

```text
Operation: getGeneration
NameOrId: generation-ii
```

returns:

```text
id: 2
name: generation-ii
main_region:
  name: johto
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

The node primarily uses values from its configuration.

For `getPokemon` and `getGeneration`, the node also contains fallback logic for incoming workflow data.

The lookup value is resolved as:

```text
configured nameOrId
```

or, if not configured:

```text
incoming workflow data
```

If incoming data is a string, it is used directly.

If incoming data is not a string, the node converts it using:

```text
String(data)
```

### Outputs

The exact output depends on the selected operation.

#### `getPokemon`

Returns the Pokémon object provided by the PokéAPI library.

Example fields include:

```json
{
  "id": 25,
  "name": "pikachu"
}
```

The actual response contains additional Pokémon information.

#### `searchPokemon`

Returns a paginated result structure containing values such as:

```json
{
  "count": 1351,
  "next": "...",
  "previous": null,
  "results": []
}
```

#### `getGeneration`

Returns generation information.

Example:

```json
{
  "id": 1,
  "name": "generation-i"
}
```

The actual response contains additional generation information such as the main region and related Pokémon data.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Get Pikachu

Configuration:

```text
Operation: getPokemon
NameOrId: pikachu
```

Validated result includes:

```text
id: 25
name: pikachu
```

---

### Get Pokémon by ID

Configuration:

```text
Operation: getPokemon
NameOrId: 1
```

Validated result includes:

```text
id: 1
name: bulbasaur
```

---

### Search First Five Pokémon

Configuration:

```text
Operation: searchPokemon
Search Limit: 5
Search Offset: 0
```

Validated response:

```text
results: 5 items
previous: null
next: offset=5&limit=5
```

The result begins with Pokémon such as:

```text
bulbasaur
ivysaur
venusaur
```

---

### Search with Offset

Configuration:

```text
Operation: searchPokemon
Search Limit: 3
Search Offset: 5
```

Validated results:

```text
charizard
squirtle
wartortle
```

Pagination:

```text
previous: offset=2&limit=3
next: offset=8&limit=3
```

---

### Get Generation by ID

Configuration:

```text
Operation: getGeneration
NameOrId: 1
```

Validated result includes:

```text
id: 1
name: generation-i
main_region:
  name: kanto
```

---

### Get Generation by Name

Configuration:

```text
Operation: getGeneration
NameOrId: generation-ii
```

Validated result includes:

```text
id: 2
name: generation-ii
main_region:
  name: johto
```

---

### Search Limit Set to Zero

Configuration:

```text
Operation: searchPokemon
Search Limit: 0
Search Offset: 0
```

The node falls back to:

```text
limit: 20
```

Validated response:

```text
results: 20 items
next: offset=20&limit=20
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve Pokémon information for Pikachu
```

### Workflow Structure

```text
Manual Trigger → PokéAPI → Log
```

The example workflow uses:

```text
Operation: getPokemon
NameOrId: pikachu
```

The Log node can be used to inspect the Pokémon object returned by the PokéAPI node.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### Pokémon not found

If a Pokémon does not exist, the PokéAPI library returns an error.

Example invalid value:

```text
pokemon-does-not-exist-999
```

The node wraps the library error using:

```text
PokéAPI request failed:
```

A validated example returned:

```text
PokéAPI request failed: ResourceNotFound: pokemon not found.
```

**Solution:** Verify that the Pokémon name or ID is valid.

---

#### Generation not found

**Cause:** The configured generation name or ID does not correspond to a valid PokéAPI generation.

**Solution:** Use a valid generation identifier such as:

```text
1
generation-i
generation-ii
```

---

#### Search Limit 0 returns 20 results

**Cause:** The node uses fallback logic:

```text
searchLimit || 20
```

Therefore, `0` is replaced by the default value `20`.

**Solution:** Use a positive non-zero value when specifying `searchLimit`.

---

#### Unexpected lookup value from workflow input

For `getPokemon` and `getGeneration`, if `nameOrId` is empty, the node attempts to use incoming workflow data.

Non-string input is converted using:

```text
String(data)
```

For example, an object can become:

```text
[object Object]
```

This can lead to an invalid API lookup.

**Solution:** Configure `nameOrId` explicitly when the incoming data is not a simple string.

---

#### PokéAPI request fails

All operation errors are wrapped with the prefix:

```text
PokéAPI request failed:
```

**Solution:** Inspect the remaining error message to determine whether the resource is invalid or the request failed.

### Behavior Reference

| Behavior | Cause | Solution |
|----------|-------|----------|
| Pokémon not found | Invalid Pokémon name or ID | Verify `nameOrId` |
| Generation not found | Invalid generation name or ID | Use a valid generation identifier |
| Limit `0` becomes `20` | `searchLimit || 20` fallback | Use a positive non-zero limit |
| Unexpected lookup string | Non-string workflow input converted with `String()` | Configure `nameOrId` explicitly |
| Request returns an error | PokéAPI/library failure | Inspect the wrapped error message |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- **GraphQL** - Execute GraphQL API requests
- **HTTP Request** - Send general HTTP requests to external APIs
- **Log** - Inspect PokéAPI responses inside a workflow

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-02 | Initial documentation and validated workflow example |

<!-- /SECTION: changelog -->
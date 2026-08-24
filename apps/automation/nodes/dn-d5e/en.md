---
node_id: "dn-d5e"
title: "D&D 5e API"
description: "Get D&D 5th Edition information including spells, classes, features, and monsters."
category: "utilities-misc"
subcategory: "games"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - dungeons-and-dragons
  - dnd
  - dnd-5e
  - games
  - spells
  - monsters
  - classes
related_nodes:
  - deck-of-cards
  - open-trivia
  - jikan
---

<!-- SECTION: header -->
# D&D 5e API

> **Category:** Utilities & Misc | **Subcategory:** Games | **Type:** Action Node

Retrieve Dungeons & Dragons 5th Edition data, including spells, classes, features, and monsters.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **D&D 5e API** node queries the public D&D 5e API for game rules and reference data.

Use it to retrieve a specific spell or monster, search spells by name, or list available classes and features for game, character-building, and tabletop automation workflows.

### Key Features

- **Spell Lookup:** Retrieve a spell by its API index.
- **Spell Search:** Search for spells by name.
- **Class Data:** Retrieve available D&D 5e classes.
- **Feature Data:** Retrieve available class features.
- **Monster Lookup:** Retrieve a monster by its API index.
- **No Credentials Required:** Uses the public D&D 5e API without an API token or secret.
- **Workflow Ready:** Connect results to log, function, AI, or game-related nodes.

### Use Cases

- Build character creation workflows
- Look up spell descriptions and mechanics
- Retrieve monster statistics for encounter preparation
- Create tabletop campaign assistants
- Search spells by name during gameplay
- Enrich game and role-playing workflows with rules data

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | — | Operation to perform: `getSpell`, `searchSpellByName`, `getClasses`, `getFeatures`, or `getMonster`. |
| `spellIndex` | `string` | Yes for `getSpell` | — | API index of the spell to retrieve, such as `magic-missile`. |
| `spellName` | `string` | Yes for `searchSpellByName` | — | Spell name to search for, such as `Fireball`. |
| `monsterIndex` | `string` | Yes for `getMonster` | — | API index of the monster to retrieve, such as `adult-black-dragon`. |

### Supported Operations

| Operation | Required Parameter | Description |
|-----------|--------------------|-------------|
| `getSpell` | `spellIndex` | Retrieve one spell by its API index. |
| `searchSpellByName` | `spellName` | Search for a spell by name. |
| `getClasses` | None | Retrieve the available D&D 5e classes. |
| `getFeatures` | None | Retrieve the available class features. |
| `getMonster` | `monsterIndex` | Retrieve one monster by its API index. |

### Example Configurations

Retrieve the Magic Missile spell:

```json
{
  "operation": "getSpell",
  "spellIndex": "magic-missile"
}
```

Search for Fireball:

```json
{
  "operation": "searchSpellByName",
  "spellName": "Fireball"
}
```

Retrieve a monster:

```json
{
  "operation": "getMonster",
  "monsterIndex": "adult-black-dragon"
}
```

The public D&D 5e API does not require an API key, bearer token, password, or other secret.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | unknown | Incoming workflow data. The configured operation and its corresponding identifier or search name determine the request. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | object or array | D&D 5e data returned by the selected operation. |
| `error` | object | Error details when validation, network, or API processing fails. |

### Successful Response

The response shape depends on the selected operation. Spell and monster lookups return the requested resource, while class and feature operations return collections or index data. Records may contain names, descriptions, indexes, URLs, and operation-specific game statistics.

Example spell response shape:

```json
{
  "index": "magic-missile",
  "name": "Magic Missile",
  "level": 1,
  "school": {
    "name": "Evocation"
  },
  "desc": [
    "You create three glowing darts of magical force."
  ],
  "url": "/api/2014/spells/magic-missile"
}
```

### Error Output

Errors are routed through the `error` output. They can result from a missing operation parameter, an invalid API index, a network failure, or an unavailable public API.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Get a Spell

```json
{
  "operation": "getSpell",
  "spellIndex": "magic-missile"
}
```

### Example: Search a Spell by Name

```json
{
  "operation": "searchSpellByName",
  "spellName": "Fireball"
}
```

### Example: List Classes

```json
{
  "operation": "getClasses"
}
```

### Example: List Features

```json
{
  "operation": "getFeatures"
}
```

### Example: Get a Monster

```json
{
  "operation": "getMonster",
  "monsterIndex": "adult-black-dragon"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve D&D 5e data and inspect the result
```

### Common Patterns

- **Spell Lookup:** Manual Trigger → D&D 5e API → Log
- **Monster Reference:** Manual Trigger → D&D 5e API → Log
- **Character Builder:** D&D 5e API → Function → Database
- **Game Assistant:** D&D 5e API → AI Chat → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Missing operation parameter

**Cause:** No supported operation was selected.

**Solution:** Set `operation` to one of `getSpell`, `searchSpellByName`, `getClasses`, `getFeatures`, or `getMonster`.

#### Missing spell index

**Cause:** `getSpell` was selected without a `spellIndex` value.

**Solution:** Provide a valid API spell index, such as `magic-missile`.

#### Missing spell name

**Cause:** `searchSpellByName` was selected without a `spellName` value.

**Solution:** Provide a spell name such as `Fireball`.

#### Missing monster index

**Cause:** `getMonster` was selected without a `monsterIndex` value.

**Solution:** Provide a valid API monster index, such as `adult-black-dragon`.

#### Resource not found

**Cause:** The supplied spell or monster index does not exist in the D&D 5e API.

**Solution:** Check the index spelling and use the API's available resource list.

#### Request failed

**Cause:** The public D&D 5e API is temporarily unavailable or the network request failed.

**Solution:** Check connectivity and retry the workflow later. No API token or secret is required.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Deck of Cards](../deck-of-cards/en.md) - Work with card and deck data
- [Open Trivia](../open-trivia/en.md) - Retrieve trivia questions
- [Jikan](../jikan/en.md) - Query anime and manga data

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation |

<!-- /SECTION: changelog -->

---
node_id: "deck-of-cards"
title: "Deck of Cards"
description: "Create, shuffle, draw, and manage decks of cards using the Deck of Cards API."
category: "utilities-misc"
subcategory: "games"
version: "1.0.0"
language: "en"
last_updated: "2026-08-31"
author: "Fusion Team"
tags:
  - cards
  - games
  - deck
  - shuffle
  - draw
  - piles
  - api
related_nodes:
  - open-trivia
  - dn-d5e
  - ffxiv
---

<!-- SECTION: header -->
# Deck of Cards

> **Category:** Utilities & Misc | **Type:** Action Node

Create, shuffle, draw, reshuffle, and manage card piles using the Deck of Cards API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Deck of Cards** node integrates with the Deck of Cards API to create and manage standard playing card decks.

It supports creating new decks, shuffling decks, drawing cards, reshuffling existing decks, creating and managing piles, drawing cards from piles, and returning cards to a deck.

The operation is selected through the `operation` parameter. Each operation exposes its own required configuration fields.

Supported operations:

- New Deck
- Shuffle New Deck
- Draw Cards
- Reshuffle
- Add Cards to Pile
- Shuffle Pile
- List Pile
- Draw From Pile
- Return Cards

### Key Features

- Create a new standard 52-card deck.
- Optionally include Jokers when creating a deck.
- Create and shuffle one or multiple decks.
- Create a partial deck from specific card codes.
- Draw one or more cards from an existing deck.
- Reshuffle an existing deck.
- Create and manage named card piles.
- Shuffle and inspect pile contents.
- Draw cards from the top, bottom, or random position of a pile.
- Return drawn cards or pile cards to the deck.
- Uses the public Deck of Cards API without requiring credentials.

### Processing Flow

```text
Select operation
        ↓
Load operation-specific parameters
        ↓
Build Deck of Cards API request
        ↓
Send request
        ↓
Receive API response
        ↓
Return deck, cards, or pile data
```

### Use Cases

- Card-game workflows.
- Random card selection.
- Game simulations.
- Deck and hand management.
- Drawing random cards.
- Managing temporary card piles.
- Testing game logic in automation workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Operation

The `operation` parameter determines which Deck of Cards API operation is performed.

Supported values:

```text
newDeck
shuffleNew
drawCards
reshuffle
addToPile
shufflePile
listPile
drawFromPile
returnCards
```

The default operation is:

```text
shuffleNew
```

---

### New Deck

Select:

```text
operation: newDeck
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `jokersEnabled` | `boolean` | No | Include two Jokers in the deck. Default is `false`. |

Example:

```text
operation: newDeck
jokersEnabled: false
```

Example output:

```json
{
  "success": true,
  "deck_id": "8jm8kivtzuly",
  "remaining": 52,
  "shuffled": false
}
```

When Jokers are enabled, the created deck includes the additional Joker cards.

---

### Shuffle New Deck

Select:

```text
operation: shuffleNew
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `deckCount` | `number` | No | Number of decks to use. Valid range is `1` to `20`. Default is `1`. |
| `cards` | `string` | No | Comma-separated card codes used to create a partial deck. |

Card codes use:

```text
Value + Suit
```

Supported value examples:

```text
A
2
3
4
5
6
7
8
9
0
J
Q
K
```

Supported suits:

```text
S = Spades
D = Diamonds
C = Clubs
H = Hearts
```

Example partial deck:

```text
AS,2S,KS,AD
```

Example:

```text
operation: shuffleNew
deckCount: 1
cards: leave empty
```

Example output:

```json
{
  "success": true,
  "deck_id": "1bcu424l3206",
  "remaining": 52,
  "shuffled": true
}
```

---

### Draw Cards

Select:

```text
operation: drawCards
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `deckId` | `string` | Yes | Deck ID returned by a previous deck operation. |
| `drawCount` | `number` | No | Number of cards to draw. Minimum value is `1`. Default is `1`. |

Example:

```text
operation: drawCards
deckId: 8jm8kivtzuly
drawCount: 3
```

Expected behavior:

```text
3 cards are returned.
remaining decreases from 52 to 49.
```

The `cards` array contains the cards returned by the API.

Each card can contain fields such as:

```text
code
image
images
value
suit
```

---

### Reshuffle

Select:

```text
operation: reshuffle
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `deckId` | `string` | Yes | ID of the deck to reshuffle. |
| `remainingOnly` | `boolean` | No | Shuffle only the remaining cards. Default is `false`. |

Example:

```text
operation: reshuffle
deckId: 8jm8kivtzuly
remainingOnly: false
```

Example output:

```json
{
  "success": true,
  "deck_id": "8jm8kivtzuly",
  "remaining": 52,
  "shuffled": true
}
```

When `remainingOnly` is enabled, the API reshuffles only the cards currently remaining in the deck.

---

### Add Cards to Pile

Select:

```text
operation: addToPile
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `deckId` | `string` | Yes | ID of the deck. |
| `pileName` | `string` | Yes | Name of the target pile. |
| `pileCards` | `string` | Yes | Comma-separated card codes to add to the pile. |

Example:

```text
operation: addToPile
deckId: 8jm8kivtzuly
pileName: testpile
pileCards: QS,4H
```

Example output:

```json
{
  "success": true,
  "deck_id": "8jm8kivtzuly",
  "remaining": 50,
  "piles": {
    "testpile": {
      "remaining": 2
    }
  }
}
```

In the tested workflow, cards were first drawn from the deck and then added to the pile using the exact card codes returned by the draw operation.

---

### Shuffle Pile

Select:

```text
operation: shufflePile
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `deckId` | `string` | Yes | ID of the deck containing the pile. |
| `pileName` | `string` | Yes | Name of the pile to shuffle. |

Example:

```text
operation: shufflePile
deckId: 8jm8kivtzuly
pileName: testpile
```

Example output:

```json
{
  "success": true,
  "deck_id": "8jm8kivtzuly",
  "remaining": 50,
  "piles": {
    "testpile": {
      "remaining": 2
    }
  }
}
```

The cards remain in the pile, but their order is shuffled.

---

### List Pile

Select:

```text
operation: listPile
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `deckId` | `string` | Yes | ID of the deck containing the pile. |
| `pileName` | `string` | Yes | Name of the pile to inspect. |

Example:

```text
operation: listPile
deckId: 8jm8kivtzuly
pileName: testpile
```

Example output structure:

```json
{
  "success": true,
  "deck_id": "8jm8kivtzuly",
  "remaining": 50,
  "piles": {
    "testpile": {
      "remaining": 2,
      "cards": [
        {
          "code": "QS",
          "value": "QUEEN",
          "suit": "SPADES"
        }
      ]
    }
  }
}
```

The pile response contains the cards currently stored in the named pile.

---

### Draw From Pile

Select:

```text
operation: drawFromPile
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `deckId` | `string` | Yes | ID of the deck containing the pile. |
| `pileName` | `string` | Yes | Name of the pile. |
| `pileDrawCount` | `number` | Conditional | Number of cards to draw from the pile. |
| `pileDrawCards` | `string` | Conditional | Specific comma-separated card codes to draw. |
| `drawFrom` | `string` | No | Position to draw from: `top`, `bottom`, or `random`. Default is `top`. |

At least one of the following must be provided:

```text
pileDrawCount
pileDrawCards
```

Supported `drawFrom` values:

```text
top
bottom
random
```

Example:

```text
operation: drawFromPile
deckId: 8jm8kivtzuly
pileName: testpile
pileDrawCount: 1
drawFrom: top
```

Example output:

```json
{
  "success": true,
  "deck_id": "8jm8kivtzuly",
  "cards": [
    {
      "code": "QS",
      "value": "QUEEN",
      "suit": "SPADES"
    }
  ]
}
```

---

### Return Cards

Select:

```text
operation: returnCards
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `deckId` | `string` | Yes | ID of the deck. |
| `returnCards` | `string` | No | Comma-separated card codes to return. If omitted, the API can return all drawn cards. |
| `returnFromPile` | `boolean` | No | Return cards from a pile instead of the drawn-card collection. Default is `false`. |
| `pileName` | `string` | Conditional | Name of the pile when returning cards from a pile. |

Example:

```text
operation: returnCards
deckId: 8jm8kivtzuly
returnCards: QS
returnFromPile: false
```

Example output:

```json
{
  "success": true,
  "deck_id": "8jm8kivtzuly",
  "remaining": 50
}
```

When returning cards from a pile, configure:

```text
returnFromPile: true
pileName: <pile name>
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses operation-specific parameters.

| Operation | Required Parameters |
|-----------|---------------------|
| `newDeck` | None |
| `shuffleNew` | None |
| `drawCards` | `deckId` |
| `reshuffle` | `deckId` |
| `addToPile` | `deckId`, `pileName`, `pileCards` |
| `shufflePile` | `deckId`, `pileName` |
| `listPile` | `deckId`, `pileName` |
| `drawFromPile` | `deckId`, `pileName`, plus `pileDrawCount` or `pileDrawCards` |
| `returnCards` | `deckId` |

Optional parameters depend on the selected operation.

### Outputs

The output is returned directly from the Deck of Cards API and varies by operation.

Common fields include:

| Field | Description |
|-------|-------------|
| `success` | Indicates whether the API operation succeeded. |
| `deck_id` | Identifier of the current deck. |
| `remaining` | Number of cards remaining in the deck. |
| `shuffled` | Indicates whether the deck was shuffled. |
| `cards` | Array of returned cards. |
| `piles` | Object containing pile information. |

Card objects can contain:

```text
code
image
images
value
suit
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Create a New Deck

**Configuration**

```text
operation: newDeck
jokersEnabled: false
```

**Output**

```json
{
  "success": true,
  "deck_id": "8jm8kivtzuly",
  "remaining": 52,
  "shuffled": false
}
```

### Example 2: Shuffle a New Deck

**Configuration**

```text
operation: shuffleNew
deckCount: 1
```

**Output**

```json
{
  "success": true,
  "deck_id": "1bcu424l3206",
  "remaining": 52,
  "shuffled": true
}
```

### Example 3: Draw Cards

**Configuration**

```text
operation: drawCards
deckId: 8jm8kivtzuly
drawCount: 3
```

**Expected behavior**

```text
3 cards are returned.
remaining decreases from 52 to 49.
```

### Example 4: Reshuffle a Deck

**Configuration**

```text
operation: reshuffle
deckId: 8jm8kivtzuly
remainingOnly: false
```

**Output**

```json
{
  "success": true,
  "deck_id": "8jm8kivtzuly",
  "remaining": 52,
  "shuffled": true
}
```

### Example 5: Add Cards to a Pile

**Configuration**

```text
operation: addToPile
deckId: 8jm8kivtzuly
pileName: testpile
pileCards: QS,4H
```

**Output**

```json
{
  "success": true,
  "deck_id": "8jm8kivtzuly",
  "remaining": 50,
  "piles": {
    "testpile": {
      "remaining": 2
    }
  }
}
```

### Example 6: List a Pile

**Configuration**

```text
operation: listPile
deckId: 8jm8kivtzuly
pileName: testpile
```

The API returns the cards contained in `testpile`.

### Example 7: Shuffle a Pile

**Configuration**

```text
operation: shufflePile
deckId: 8jm8kivtzuly
pileName: testpile
```

The pile remains available with the same number of cards after shuffling.

### Example 8: Draw From a Pile

**Configuration**

```text
operation: drawFromPile
deckId: 8jm8kivtzuly
pileName: testpile
pileDrawCount: 1
drawFrom: top
```

**Output example**

```json
{
  "success": true,
  "deck_id": "8jm8kivtzuly",
  "cards": [
    {
      "code": "QS",
      "value": "QUEEN",
      "suit": "SPADES"
    }
  ]
}
```

### Example 9: Return a Card

**Configuration**

```text
operation: returnCards
deckId: 8jm8kivtzuly
returnCards: QS
returnFromPile: false
```

**Output**

```json
{
  "success": true,
  "deck_id": "8jm8kivtzuly",
  "remaining": 50
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Deck of Cards Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Deck ID Is Required

Several operations require a valid `deckId`.

These include:

```text
drawCards
reshuffle
addToPile
shufflePile
listPile
drawFromPile
returnCards
```

Use a `deck_id` returned by a previous `newDeck` or `shuffleNew` operation.

### Pile Name Is Required

Pile operations require a valid `pileName`.

These include:

```text
addToPile
shufflePile
listPile
drawFromPile
```

When returning cards specifically from a pile, `pileName` is also required.

### Cards Are Not Added to a Pile

The API may return a successful response while the pile remains empty if the specified cards are not currently available for pile assignment.

In the tested workflow, the reliable sequence was:

```text
Create or shuffle deck
        ↓
Draw cards
        ↓
Add those exact card codes to the pile
```

Example:

```text
drawCards → returns QS,4H
addToPile → pileCards: QS,4H
```

### Draw From Pile Requires a Selection

For `drawFromPile`, configure at least one of:

```text
pileDrawCount
pileDrawCards
```

If neither is provided, the node throws:

```text
Either draw count or specific cards must be specified
```

### Deck of Cards API Request Failed

The node depends on the external Deck of Cards API.

If the service is unavailable or returns a non-successful HTTP response, the node throws an error prefixed with:

```text
Deck of Cards API request failed:
```

### Invalid Deck Count

For `shuffleNew`, `deckCount` must be between:

```text
1 and 20
```

Values outside this range are rejected by schema validation.

### Invalid Draw Count

For `drawCards`, `drawCount` must be at least:

```text
1
```

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Open Trivia** — Retrieve trivia content for game-oriented workflows.
- **DnD 5e** — Work with Dungeons & Dragons 5th Edition game data.
- **FFXIV** — Integrate Final Fantasy XIV-related data into workflows.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-31 | Initial documentation for the Deck of Cards node. |

<!-- /SECTION: changelog -->
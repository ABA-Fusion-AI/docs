---
node_id: "the-meal-db"
title: "TheMealDB"
description: "Search for meal recipes using TheMealDB API."
category: "databases-memory"
subcategory: "data-platforms"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - the-meal-db
  - recipes
  - meals
  - food
  - cooking
  - ingredients
  - api
related_nodes:
  - http-request
  - ai-chat
  - function
  - discord-bot-send
---

<!-- SECTION: header -->
# TheMealDB

> **Category:** Databases & Memory | **Subcategory:** Data Platforms | **Type:** Action Node

Search for meal recipes, ingredients, cooking instructions, and culinary metadata using TheMealDB API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **TheMealDB** node integrates with the free public [TheMealDB](https://www.themealdb.com) database to search for international meal recipes by dish name or main ingredient.

It retrieves full recipe details—including step-by-step cooking instructions, ingredient lists with exact measurements, cuisine regions (`strArea`), meal categories (`strCategory`), thumbnail images, and YouTube video tutorials.

### Key Features

- **Recipe Search by Name:** Find recipes by dish name or main ingredient (e.g. `chicken`, `couscous`, `pizza`, `Arrabiata`, `pasta`).
- **Comprehensive Cooking Data:** Returns step-by-step instructions, ingredients (`strIngredient1` to `strIngredient20`), and corresponding measurements (`strMeasure1` to `strMeasure20`).
- **Media & Tutorial Links:** Includes meal thumbnail image URLs (`strMealThumb`) and YouTube tutorial links (`strYoutube`).
- **Free & No Auth Required:** Connects to TheMealDB's open public tier without needing an API key.
- **Dual Input Modes:** Configure a query directly in the `Query` parameter or feed dynamic search terms from upstream triggers (e.g. chatbot messages, form submissions, or AI assistants).

### Processing Flow

```text
Workflow Trigger / Chatbot Message
  ↓
Read Query (from parameter or incoming input)
  ↓
Validate Search Query is non-empty
  ↓
Call TheMealDB search API (search.php?s=...)
  ↓
Parse & Map Recipe Fields (Instructions, Ingredients, Measures, Media)
  ↓
Emit Output Envelope { success, query, results, total_results }
```

### Use Cases

- **AI Culinary Assistant:** Receive user food queries via Slack, Telegram, or WhatsApp, fetch authentic recipes via TheMealDB, and have an [AI Chat](../ai-chat/en.md) node generate personalized meal prep tips.
- **Recipe of the Day Bot:** Schedule a daily workflow that picks popular dishes and posts ingredients and video links to a Discord or Slack community channel.
- **Meal Planning Automation:** Search recipes based on available pantry items and compile grocery shopping lists automatically.
- **Diet & Cooking Apps:** Integrate recipe lookups into customer support portals or food blog workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | `string` | No | — | The meal name or keyword to search for (e.g., `chicken`, `couscous`, `pizza`, `Arrabiata`). If omitted, uses the text received from the upstream workflow node. |

### Query Guidance

- Use broad meal keywords (e.g., `chicken`, `beef`, `salmon`, `pasta`) to retrieve multiple recipe variations.
- Use specific dish names (e.g., `Arrabiata`, `couscous`, `Moussaka`) for targeted lookups.
- Searches are case-insensitive and match partial names.

### Default Configuration

```json
{
  "query": "chicken"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `unknown` | Incoming workflow trigger or text payload. Used as the search query if the `query` parameter is not explicitly defined. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Returned on successful search, containing an array of matched meal recipes and total result count. |
| `error` | `object` | Returned if the query is missing or network/API errors occur. |

### Key Recipe Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `idMeal` | `string` | Unique TheMealDB meal identifier (e.g. `"52772"`). |
| `strMeal` | `string` | Full name of the dish. |
| `strCategory` | `string` | Meal category (e.g. `"Chicken"`, `"Pasta"`, `"Dessert"`, `"Vegetarian"`). |
| `strArea` | `string` | Geographic cuisine origin (e.g. `"Moroccan"`, `"Italian"`, `"Japanese"`, `"Mexican"`). |
| `strInstructions` | `string` | Complete preparation and cooking instructions text. |
| `strMealThumb` | `string` | URL to high-resolution image of the prepared meal. |
| `strTags` | `string` or `null` | Comma-separated category tags (e.g. `"Meat,Casserole"`). |
| `strYoutube` | `string` or `null` | YouTube video tutorial URL. |
| `strIngredient1`–`20` | `string` | Names of recipe ingredients. |
| `strMeasure1`–`20` | `string` | Measurement quantities corresponding to each ingredient. |
| `strSource` | `string` or `null` | Original recipe webpage URL. |

### Successful Response Example

```json
{
  "success": true,
  "query": "Arrabiata",
  "results": [
    {
      "idMeal": "52771",
      "strMeal": "Spicy Arrabiata Penne",
      "strDrinkAlternate": null,
      "strCategory": "Vegetarian",
      "strArea": "Italian",
      "strInstructions": "Bring a large pot of lightly salted water to a boil. Cook penne pasta for 8 to 10 minutes...",
      "strMealThumb": "https://www.themealdb.com/images/media/meals/ustsqw1468250014.jpg",
      "strTags": "Pasta,Curry",
      "strYoutube": "https://www.youtube.com/watch?v=1IszT_guI08",
      "strIngredient1": "penne rigate",
      "strIngredient2": "olive oil",
      "strIngredient3": "garlic",
      "strIngredient4": "chopped tomatoes",
      "strMeasure1": "1 pound",
      "strMeasure2": "1/4 cup",
      "strMeasure3": "3 cloves",
      "strMeasure4": "1 tin",
      "strSource": null
    }
  ],
  "total_results": 1
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Search Chicken Recipes

Search for general chicken dishes:

```json
{
  "query": "chicken"
}
```

### Example 2: Search Moroccan Couscous

Search for authentic couscous recipes:

```json
{
  "query": "couscous"
}
```

### Example 3: Search Italian Pasta

Find specific pasta recipes:

```json
{
  "query": "Arrabiata"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search recipe ideas and ingredients using TheMealDB
```

### Common Workflow Patterns

- **Chatbot Recipe Lookup:** Telegram / Slack Trigger (`user_message`) → TheMealDB → AI Chat (Format into a clean recipe card) → Reply to User.
- **Daily Meal Recommendation:** Scheduled Trigger (Every day at 11 AM) → TheMealDB (`query: "pasta"`) → Function (Pick random item) → Discord Bot Send (Post Recipe of the Day).
- **Ingredient Extractor:** Manual Trigger → TheMealDB → Function (Extract non-empty `strIngredient` fields) → Create Todo / Shopping List.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Error: "Search query is required"

**Cause:** Neither the `query` parameter was supplied nor did the upstream node pass a valid search string.

**Solution:** Provide a search keyword in the `Query` parameter (e.g. `chicken` or `pizza`) or check your upstream connection.

### Empty results (`total_results: 0`)

**Cause:** No recipes matched the given query keyword in TheMealDB's database.

**Solution:** Try a broader keyword (e.g., search `beef` instead of a complex multi-word title) or verify spelling.

### API request failed

**Cause:** TheMealDB public server is experiencing temporary downtime or network connectivity issues.

**Solution:** Verify internet access or retry the workflow execution after a brief interval.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](../http-request/en.md) - Make custom API calls to external culinary APIs
- [AI Chat](../ai-chat/en.md) - Enhance recipe instructions and suggest ingredient substitutes
- [Discord Bot Send](../discord-bot-send/en.md) - Post recipe cards to Discord channels

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for TheMealDB node |

<!-- /SECTION: changelog -->

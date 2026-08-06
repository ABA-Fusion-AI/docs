---
node_id: "amazon-product-search"
title: "Amazon Product Search"
description: "Search Amazon products without an API key using direct scraping."
category: "web-search"
subcategory: "search-reference"
version: "1.0.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - amazon
  - search
  - marketplace
  - scraping
  - products
related_nodes:
  - ebay-search
  - walmart-search
  - google-dork
---

<!-- SECTION: header -->
# Amazon Product Search

> **Category:** Web Search | **Type:** Action Node

Search Amazon marketplaces without requiring an API key. The node performs direct product searches, extracts product information from search result pages, and returns structured product data for use in automation workflows.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Amazon Product Search** node performs keyword-based product searches across multiple Amazon marketplaces using direct web scraping. It extracts structured product information including titles, prices, ratings, review counts, product links, and images without requiring Amazon Product Advertising API credentials.

The node automatically builds Amazon search URLs, performs HTTP requests using browser-like headers, parses the returned HTML, and extracts product information from the search results.

### Key Features

- **No API Key Required:** Search Amazon products without AWS credentials or Product Advertising API access
- **Multi-Domain Support:** Search across 11 Amazon marketplaces
- **Keyword Search:** Search products using free-text queries
- **Product Filters:** Filter by product category, brand, and price range
- **Structured Results:** Returns product title, price, rating, review count, image, and product URL
- **Configurable Result Limit:** Return between 1 and 50 products
- **Timeout Control:** Configure request timeout from 1 to 30 seconds
- **Automatic Query Normalization:** Accepts configured queries or dynamic workflow input
- **Error Handling:** Handles timeout, HTTP errors, and blocked requests

### Supported Amazon Domains

| Domain | Marketplace |
|---------|-------------|
| `com` | Amazon United States |
| `co.uk` | Amazon United Kingdom |
| `de` | Amazon Germany |
| `fr` | Amazon France |
| `co.jp` | Amazon Japan |
| `ca` | Amazon Canada |
| `it` | Amazon Italy |
| `es` | Amazon Spain |
| `in` | Amazon India |
| `com.mx` | Amazon Mexico |
| `com.br` | Amazon Brazil |

### Use Cases

- Search Amazon products by keyword
- Compare products across Amazon marketplaces
- Build price monitoring workflows
- Collect product metadata
- Retrieve product ratings and review counts
- Integrate Amazon search into automation pipelines
- Feed product information into downstream processing nodes
- Create product recommendation workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration 
### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | `string` | ✅ Yes | — | Product search keywords. Maximum length: 200 characters. |
| `productType` | `string` | ❌ No | — | Optional Amazon product category filter. |
| `brand` | `string` | ❌ No | — | Optional brand filter. |
| `priceRange` | `string` | ❌ No | — | Optional Amazon price range filter. |
| `domain` | `enum` | ❌ No | `com` | Amazon marketplace domain. |
| `limit` | `number` | ❌ No | `10` | Maximum number of returned products (1–50). |
| `timeout` | `number` | ❌ No | `15000` | Request timeout in milliseconds (1000–30000). |

> **Note:** The search query can be provided either through the node configuration or from incoming workflow data.

### Default Values

| Parameter | Default |
|-----------|---------|
| `domain` | `com` |
| `limit` | `10` |
| `timeout` | `15000` |

### Supported Domains

The node supports the following Amazon marketplaces:

| Value | Marketplace |
|-------|-------------|
| `com` | United States |
| `co.uk` | United Kingdom |
| `de` | Germany |
| `fr` | France |
| `co.jp` | Japan |
| `ca` | Canada |
| `it` | Italy |
| `es` | Spain |
| `in` | India |
| `com.mx` | Mexico |
| `com.br` | Brazil |

### Query Resolution

The node resolves the search query using the following priority:

1. The configured `query` parameter.
2. Incoming workflow data when it is a string.
3. Incoming workflow data containing a `query` property.

For example:

```json
{
  "query": "iphone 16"
}
```

or simply

```text
iphone 16
```

### Product Filters

The following filters are optional.

#### Product Type

Limits the search to a specific Amazon category.

Example:

```text
electronics
```

#### Brand

Restricts results to a specific manufacturer.

Example:

```text
Apple
```

#### Price Range

Filters products using Amazon's price filter.

Example:

```text
100-500
```

The exact format depends on Amazon's marketplace filtering rules.

### Result Limit

The node returns between **1** and **50** products.

Example:

```text
Limit = 5
```

Only the first five extracted products are returned.

### Timeout

Defines how long the node waits before cancelling the request.

Minimum:

```text
1000 ms
```

Maximum:

```text
30000 ms
```

Default:

```text
15000 ms
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Optional workflow input containing the search query. |

The node accepts:

- a configured `query`
- a plain string from the previous node
- an object containing a `query` property

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Amazon search results. |
| `error` | `Error` | Emitted when the search fails. |

Successful output structure:

```json
{
  "query": "iphone 16",
  "domain": "com",
  "results": [
    {
      "title": "...",
      "link": "...",
      "review": "...",
      "rating": 4.7,
      "reviewCount": 1284,
      "price": "$999.99",
      "image": "https://..."
    }
  ],
  "resultCount": 10,
  "timestamp": "2026-08-06T10:00:00.000Z"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Search Products

Search Amazon for products using a simple keyword.

**Configuration:**

```text
Query: iphone 16
Domain: com
Limit: 5
Timeout: 15000
```

**Output:**

```json
{
  "query": "iphone 16",
  "domain": "com",
  "results": [
    {
      "title": "Apple iPhone 16",
      "price": "$999.99",
      "rating": 4.8,
      "reviewCount": 1352,
      "link": "https://www.amazon.com/...",
      "image": "https://..."
    }
  ],
  "resultCount": 5,
  "timestamp": "2026-08-06T12:00:00.000Z"
}
```

---

### Example: Search by Brand

Restrict the search to a specific manufacturer.

**Configuration:**

```text
Query: wireless earbuds
Brand: Sony
Domain: com
Limit: 10
```

Only products matching the specified brand are returned.

---

### Example: Search by Product Type

Limit the search to a specific Amazon category.

**Configuration:**

```text
Query: keyboard
Product Type: electronics
Domain: com
```

The generated search URL includes the selected category.

---

### Example: Search by Price Range

Search products within a specified Amazon price range.

**Configuration:**

```text
Query: monitor
Price Range: 100-300
Domain: com
```

Only products matching the configured price filter are returned.

---

### Example: Search Amazon France

Search the French marketplace.

**Configuration:**

```text
Query: ordinateur portable
Domain: fr
Limit: 5
```

The node automatically searches:

```text
https://www.amazon.fr
```

---

### Example: Search Amazon Germany

Search the German marketplace.

**Configuration:**

```text
Query: laptop
Domain: de
```

The node automatically searches:

```text
https://www.amazon.de
```

---

### Example: Limit Returned Results

Return only three products.

**Configuration:**

```text
Query: gaming mouse
Limit: 3
```

Only the first three extracted products are included in the response.

---

### Example: Dynamic Query from Previous Node

Receive the query dynamically.

**Input:**

```json
{
  "query": "mechanical keyboard"
}
```

**Configuration:**

```text
Query: (empty)
Domain: com
```

The node automatically uses:

```text
mechanical keyboard
```

as the search query.

---

### Example: Dynamic String Input

Receive a plain string from the previous node.

**Input:**

```text
airpods pro
```

No configured query is required.

The node automatically searches Amazon for:

```text
airpods pro
```

---

### Example: Product Response Structure

Each returned product may contain:

```json
{
  "title": "Apple AirPods Pro",
  "link": "https://www.amazon.com/...",
  "review": "4.7 out of 5 stars",
  "rating": 4.7,
  "reviewCount": 18342,
  "price": "$199.99",
  "image": "https://m.media-amazon.com/..."
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search Amazon products
```

### Common Patterns

- **Product Lookup:** Manual Trigger → Amazon Product Search → Log
- **Marketplace Search:** HTTP Request → Amazon Product Search → Function
- **Price Monitoring:** Scheduler → Amazon Product Search → Compare Results
- **Product Enrichment:** Database → Amazon Product Search → Store Metadata
- **Marketplace Comparison:** Loop → Amazon Product Search → Aggregate Results

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "Search query is required and cannot be empty"

**Cause:** No search query was provided.

**Solution:** Configure the `query` parameter or provide a string or object containing a `query` property from the previous node.

---

#### "Search query exceeds maximum length of 200 characters"

**Cause:** The configured search query is too long.

**Solution:** Reduce the query length to fewer than 200 characters.

---

#### "Amazon search request timed out"

**Cause:** Amazon did not respond before the configured timeout.

**Solution:** Increase the timeout value or retry later.

---

#### "Amazon blocked the request"

**Cause:** Amazon may temporarily block automated requests.

**Solution:** Wait a few minutes before retrying or try another supported marketplace.

---

#### "Amazon is temporarily unavailable"

**Cause:** Amazon returned HTTP 503.

**Solution:** Retry later.

---

#### No products returned

**Cause:** The search query produced no matching products.

**Solution:** Verify the query, remove restrictive filters, or search a different marketplace.

---

#### Empty product fields

**Cause:** Amazon changed the structure of its search results page.

**Solution:** Update the node selectors to match the latest Amazon HTML structure.

---

### Error Codes

| Error | Description |
|-------|-------------|
| `Search query is required` | No search query was provided. |
| `Search query exceeds maximum length` | Query is longer than 200 characters. |
| `HTTP 403` | Amazon temporarily blocked the request. |
| `HTTP 503` | Amazon service is temporarily unavailable. |
| `Request timeout` | The configured timeout was reached. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Google Dork](./google-dork.md) - Perform advanced Google searches

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-06 | Initial release |

<!-- /SECTION: changelog -->
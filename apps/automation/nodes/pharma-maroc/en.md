---
node_id: "pharma-maroc"
title: "PharmaMaroc"
description: "Search Blink Pharmacie products and retrieve selected product information sections."
category: "search"
subcategory: "specialized"
version: "1.0.0"
language: "en"
last_updated: "2026-11-21"
author: "Fusion Team"
tags:
  - pharma-maroc
  - products
  - pharmacy
related_nodes:
  - manual-trigger
  - function
  - http-request
---

<!-- SECTION: overview -->
# PharmaMaroc

> **Type:** Action Node

Search products through the Blink Pharmacie API, then retrieve information for a product identifier returned by the search. Details can include product data, synthesis, indications and administration methods, patient safety, molecules and adverse effects.

The node calls `https://api.blinkpharmacie.ma/api/v3/`. There is no API-key field. Responses are passed through from the provider without converting prices, stripping HTML or verifying medical content. This page describes data retrieval and workflow behavior.

<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `operation` | enum | Default provided | `searchProducts` | `searchProducts` or `getProductDetails`. |
| `query` | string | Optional in configuration | Empty | Search term. Must resolve from configuration or upstream input for search. |
| `productId` | string | Optional in configuration | Empty | Product identifier. Must resolve for details. Use an ID returned by search. |
| `product_data` | boolean | Optional | `true` | Include general product information. |
| `synthesis` | boolean | Optional | `false` | Include the synthesis section. |
| `indication_methods` | boolean | Optional | `false` | Include indications and administration methods. |
| `patient_safety` | boolean | Optional | `false` | Include patient safety information. |
| `molecules` | boolean | Optional | `false` | Include active molecule information. |
| `adverse_effects` | boolean | Optional | `false` | Include adverse effects grouped by the provider. |

Fields support Expressions. Both Query and ProductId remain visible for both operations. Search uses Query and ignores ProductId and all section switches. Details uses ProductId and the section switches and ignores Query.

There are no pagination or result-limit parameters. Search returns the provider's response. For details, select at least one section. The current implementation returns `{}` without making detail requests when all switches are disabled; it does not raise a validation error for this case.

### Detail endpoints

Each enabled section makes one GET request, sequentially in the order below. Disabled sections are omitted from the output.

| Section | Path relative to `/api/v3/` |
|---|---|
| `product_data` | `product-details/<encoded-id>` |
| `synthesis` | `product_synthesis/<encoded-id>` |
| `indication_methods` | `indecation-methods/<encoded-id>` |
| `patient_safety` | `patient-safety/<encoded-id>` |
| `molecules` | `active_molecules/<encoded-id>` |
| `adverse_effects` | `products_adverse_effects?product_ids=<encoded-id>&molecule_ids=&frequencies=&sort_order=largest_to_smallest` |

The `indecation-methods` spelling above matches the implemented endpoint.

<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input resolution

For each operation, a non-empty configured value after trimming takes priority. If it is empty, the node reads upstream input. It trims resolved strings and accepts numbers by converting them to strings.

| Operation | Direct input | Object aliases, in priority order |
|---|---|---|
| `searchProducts` | string or number | `query`, `search`, `text` |
| `getProductDetails` | string or number | `productId`, `id`, `data` |

Object aliases are selected using truthiness before checking their type. An earlier truthy value of an unsupported type can prevent a later alias from being used. Supply a plain string in the intended field. An object nested under `data` is not automatically unpacked. Section switches always come from configuration, not upstream input.

A search Function can return:

```javascript
return { query: "doliprane" };
```

Keep Query blank to use this input fallback. For a direct string, use `return "doliprane";`.

For a dynamic details lookup:

```javascript
return {
  productId: "52078bb6-5716-4436-8971-c8f62d7082af"
};
```

Keep ProductId blank to use this value, or bind the field through the Expression picker. A configured ProductId overrides the upstream identifier, as verified in Fusion.

Product identifiers containing `..`, `/` or a backslash are rejected. Other non-empty values are URL-encoded and sent to the provider; the node does not enforce UUID syntax. Full product URLs are not accepted as identifiers.

### Search output

Search calls `analysis_suggestion?search=<encoded-query>` and returns the parsed JSON without adding a success wrapper. The observed response contains a `data` array with product entries.

| Observed field | Type | Description |
|---|---|---|
| `id` | string | Identifier to pass to ProductId. |
| `name` | string | Product name and presentation. |
| `type` | string | Observed value `product`. |
| `PPV` | string | Price value supplied by the provider; not converted to a number. |
| `reimbursable` | boolean | Provider reimbursement flag. |
| `is_generic` | string | Provider classification, such as `GENERIQUE`. |

This excerpt contains one entry from the observed 16-result search for `doliprane`:

```json
{
  "data": [
    {
      "id": "52078bb6-5716-4436-8971-c8f62d7082af",
      "name": "DOLIPRANE 100MG NOURISSON 10 SUPPOSITOIRES",
      "type": "product",
      "PPV": "8.10",
      "reimbursable": true,
      "is_generic": "GENERIQUE"
    }
  ]
}
```

An observed unmatched query returned:

```json
{"data": []}
```

### Details output

The result is an object keyed by the enabled section names. The provider's parsed JSON is stored under each key. There is no global success field, and section payloads do not all use the same envelope.

| Section | Observed payload structure |
|---|---|
| `product_data` | `data` object with ID, name, laboratory fields, prices, presentation, form, ATC code, update timestamp, sidebar availability information and molecule summaries. |
| `synthesis` | `data.synthesis` array. |
| `indication_methods` | `data` object with indications, posology and administration fields. Some values contain HTML. |
| `patient_safety` | `data` object with named safety sections; values include arrays and objects. |
| `molecules` | `data.molecules` array. |
| `adverse_effects` | Object whose category names map to arrays of adverse-effect entries, without the `data` envelope observed on other sections. |

A shortened observed product-data response is shown below. The full response contains additional laboratory, availability and update fields.

```json
{
  "product_data": {
    "data": {
      "id": "52078bb6-5716-4436-8971-c8f62d7082af",
      "name": "DOLIPRANE 100MG NOURISSON 10 SUPPOSITOIRES",
      "lab_name": "BOTTU S.A.",
      "lab_address": null,
      "PPV": "8.10",
      "PH": "5.00",
      "presentation": "BOITE DE 10",
      "form": "SUPPOSITOIRE",
      "atc_code": "N02BE01",
      "molecules": [{"atc_code": "N02BE01", "name": "Paracétamol"}]
    }
  }
}
```

Preserve null values, empty arrays, strings and HTML as received. An empty section is not enough to conclude that information does not exist. Inspect section errors and source completeness separately. The node does not independently validate medical claims or interpret the content.

<!-- /SECTION: inputs-outputs -->

<!-- SECTION: workflow-example -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve product information and inspect section sizes
```

The example retains the five-node path from the supplied export: Manual Trigger → Function → PharmaMaroc → Function → Log. Credentials, variables and node parameters have been removed. Configure the nodes after importing.

1. In the upstream Function, paste the dynamic ProductId example above.
2. Set PharmaMaroc Operation to `getProductDetails` and leave ProductId blank to use the Function input.
3. Enable the desired sections. Start with Product_data only; enable all six when inspecting the full response.
4. Paste the summary below into the downstream Function.
5. Run Manual Trigger and inspect Log. Inspect the original PharmaMaroc output for full payloads.

```javascript
if (!input || typeof input !== "object" || Array.isArray(input)) {
  throw new Error("Expected PharmaMaroc section output");
}
return {
  sections: Object.entries(input).map(([name, value]) => {
    const error = value && typeof value === "object" ? value.error : null;
    const payload = value && typeof value === "object" && "data" in value
      ? value.data : value;
    return {
      name,
      error: error || null,
      type: Array.isArray(payload) ? "array"
        : payload === null ? "null" : typeof payload,
      entries: Array.isArray(payload) ? payload.length
        : payload && typeof payload === "object"
          ? Object.keys(payload).length : null
    };
  })
};
```

The Function variable is `input`, not `inputs`. This summary reports shape and section errors; it does not verify medical accuracy or prove that each provider response is complete.

### Reproduce search and details

Set Operation to `searchProducts` and Query to `doliprane`. The observed run returned 16 entries. `  DOLIPRANE  ` returned the same identifiers after trimming. The node trims text but does not force its case; the observed case behavior comes from the provider. Counts, prices and ordering may change.

Choose an ID from `data`, then switch to `getProductDetails`. The observed ID `52078bb6-5716-4436-8971-c8f62d7082af` returned the matching product name and `PPV: "8.10"` in Product_data.

### Inspect long sections

To inspect Patient_safety, enable that section and use this downstream Function:

```javascript
const section = input?.patient_safety;
if (!section || section.error || !section.data) {
  return { received: false, error: section?.error || "patient_safety.data absent" };
}
return {
  received: true,
  sections: Object.entries(section.data).map(([name, value]) => ({
    name,
    type: Array.isArray(value) ? "array" : value === null ? "null" : typeof value,
    count: Array.isArray(value) ? value.length : null
  }))
};
```

For Adverse_effects, use `input.adverse_effects` directly rather than `.data`. A compact run on product `950bcff7-b6a6-4167-9c16-0a8e64116fa0` showed 13 patient-safety sections and, in a separate run, 16 adverse-effect categories totaling 81 entries. These observations apply to those runs, not every product.

### Checks performed in Fusion

| Scenario | Observed behavior |
|---|---|
| Search `doliprane` | 16 matching product entries. |
| Uppercase query with surrounding spaces | Same 16 identifiers. |
| Unmatched query `qzxvjk987654321` | `data: []`. |
| Empty Query without supported input | Required-search error. |
| Empty ProductId without supported input | Required-ID error. |
| ProductId `../test` | Invalid-ID error. |
| Nonexistent UUID-shaped ID | `product_data.error`, with a generic message. |
| No sections selected | `{}`. |
| Query supplied by upstream Function | Same observed search entries. |
| ProductId supplied by upstream Function | Matching product details. |
| Configured ProductId differs from upstream ID | Configured identifier takes priority. |
| Detail sections | Payloads were observed for all six; compact summaries were used for the long safety and adverse-effect sections. |

These checks describe functional behavior observed in the supplied runs. Network outages, timeouts, rate limiting and malformed provider responses were not validated by these runs.

<!-- /SECTION: workflow-example -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Observation | Explanation and action |
|---|---|
| `Le terme de recherche est requis.` | Set Query or supply supported upstream search input. Whitespace is trimmed. |
| `L'ID du produit est requis` | Set ProductId or supply a supported upstream identifier. |
| `ID du produit invalide` | Remove path-like input. Select an identifier from search instead of a URL. |
| `Erreur d'accès à l'API blinkpharmacie.ma: status <code>` | Search returned a non-success HTTP status. The current node does not include the response body. |
| A section contains `{"error":"Erreur sur product_data"}` or another section name | Its details request returned a non-success status. This is a section-level error; other sections may still be returned. The status and provider body are not exposed. |
| `{}` after a details request | Enable at least one section. |
| JSON parsing error or network exception | The current implementation directly parses successful responses and lets these exceptions interrupt execution. Earlier sections are not returned as a partial result when execution throws. |
| Slow execution | Enabled sections are requested sequentially. The node has no explicit fetch timeout or retry policy. Enable fewer sections while investigating. |
| Long output is cut off when copied | Use a downstream Function to summarize sections, then inspect full output in the node. A truncated pasted file alone does not establish an API or node failure. |
| `inputs is not defined` in Function | Use the singular variable `input`. |
| HTML appears in posology fields | HTML is part of the provider response and is not stripped by this node. Use an appropriate downstream transformation or safe renderer. |

The current node does not normalize or validate the full provider response schema. Do not assume that a successful JSON parse guarantees a complete product record. Missing values remain as supplied by the provider.

<!-- /SECTION: troubleshooting -->

<!-- SECTION: related -->
## Related

- [Manual Trigger](/docs/manual-trigger) — Start a product lookup.
- [Function](/docs/function) — Supply identifiers and summarize long responses.
- [HTTP Request](/docs/http-request) — Inspect an API response separately.

<!-- /SECTION: related -->

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0.0 | 2026-11-21 | Document both operations, six detail sections, input precedence, observed outputs, limitations and the sanitized supplied workflow. |

<!-- /SECTION: changelog -->

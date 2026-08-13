---
node_id: "company-search"
title: "Company Search"
description: "Search for Moroccan company information using Maroc Facture and/or Charika.ma APIs."
category: "search"
subcategory: "specialized"
version: "1.0.0"
language: "en"
last_updated: "2026-08-13"
author: "Fusion Team"
tags:
  - company-search
  - search
  - morocco
  - maroc-facture
  - charika
  - business
  - corporate
  - registry
  - specialized
  - action
related_nodes:
  - http-request
  - function
  - if
---

<!-- SECTION: overview -->
# Company Search

> **Category:** Search &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Search for Moroccan company information using **Maroc Facture** and/or **Charika.ma** APIs.

The **Company Search** node allows workflows to query business information, corporate registry details, legal identification (such as ICE - Identifiant Commun de l'Entreprise, RC - Registre du Commerce), registered addresses, activity sectors, and financial status for companies operating in Morocco.

### Supported Data Sources

- **Maroc Facture API:** Queries company data via Maroc Facture registry services.
- **Charika.ma API:** Queries corporate business intelligence records via Charika.ma.
- **Combined Search:** Aggregates and merges results from both Maroc Facture and Charika.ma APIs in a single request.

### Use Cases

- **Know Your Customer (KYC):** Verify Moroccan company credentials during customer or supplier onboarding.
- **B2B Data Enrichment:** Automatically populate company registration numbers (ICE, RC) and official addresses into CRM or ERP systems.
- **Invoice & Billing Validation:** Verify company billing data against official corporate registry records.
- **Due Diligence & Market Intelligence:** Look up business activities, legal form, and official details of Moroccan organizations.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `companyName` | `string` | ✅ Yes | — | The name of the Moroccan company to search for (e.g., `"Maroc Telecom"`). Supports expressions. |
| `source` | `enum` | ❌ No | `"maroc-facture"` | The data provider to query for company records. |

### Data Source Options (`source`)

| Option | Value | Description |
|--------|-------|-------------|
| **Maroc Facture** | `maroc-facture` | Search company records exclusively using the Maroc Facture API. |
| **Charika.ma** | `charika` | Search corporate business data exclusively using the Charika.ma API. |
| **Combined** | `combined` | Perform a concurrent search across both Maroc Facture and Charika.ma APIs and return merged results. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` or `string` | Data trigger or payload containing incoming parameters for company lookup. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Returned when company data is successfully retrieved from the selected source(s). |
| `error` | `object` | Returned if the API request fails, authentication fails, or validation errors occur. |

### Output Data Fields

| Field | Type | Description |
|-------|------|-------------|
| `companyName` | `string` | Official name of the matching company. |
| `ice` | `string` | Identifiant Commun de l'Entreprise (ICE) number, when available. |
| `rc` | `string` | Registre du Commerce (RC) number and registration city. |
| `address` | `string` | Registered office address. |
| `city` | `string` | City of registered headquarters. |
| `activity` | `string` | Primary business activity or sector description. |
| `legalForm` | `string` | Legal structure (e.g., SA, SARL, SARLAU). |
| `source` | `string` | Data provider that returned the record (`maroc-facture`, `charika`, or `combined`). |
| `raw` | `object` | Raw response payload returned by the provider API(s). |

---

### Configuration & Output Examples

#### Example 1: Search via Maroc Facture

**Configuration**
```json
{
  "companyName": "Maroc Telecom",
  "source": "maroc-facture"
}
```

**Output (`success`)**
```json
{
  "companyName": "ITISSALAT AL-MAGHRIB (MAROC TELECOM)",
  "ice": "001527394000088",
  "rc": "48648 Rabat",
  "legalForm": "SA",
  "city": "Rabat",
  "address": "Avenue Annakhil, Hay Riad, Rabat",
  "activity": "Télécommunications",
  "source": "maroc-facture"
}
```

---

#### Example 2: Search via Charika.ma

**Configuration**
```json
{
  "companyName": "Ita telecom maroc",
  "source": "charika"
}
```

**Output (`success`)**
```json
{
  "companyName": "MAROC TELECOM",
  "ice": "001527394000088",
  "rc": "48648 Rabat",
  "legalForm": "Société Anonyme",
  "city": "Rabat",
  "address": "Avenue Annakhil, Hay Riad, Rabat",
  "source": "charika"
}
```

---

#### Example 3: Combined Search (Maroc Facture + Charika.ma)

**Configuration**
```json
{
  "companyName": "Maroc Telecom",
  "source": "combined"
}
```

**Output (`success`)**
```json
{
  "companyName": "MAROC TELECOM",
  "ice": "001527394000088",
  "rc": "48648 Rabat",
  "legalForm": "SA",
  "city": "Rabat",
  "address": "Avenue Annakhil, Hay Riad, Rabat",
  "activity": "Télécommunications",
  "sources": {
    "maroc-facture": {
      "status": "success",
      "found": true
    },
    "charika": {
      "status": "success",
      "found": true
    }
  },
  "source": "combined"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search for Moroccan Company Information
```

### Sample Workflow: Manual Trigger → Company Search → Log Result

```json
{
  "nodes": [
    {
      "id": "trigger",
      "type": "manual-trigger"
    },
    {
      "id": "company-search",
      "type": "company-search",
      "config": {
        "companyName": "Maroc Telecom",
        "source": "maroc-facture"
      }
    },
    {
      "id": "log-result",
      "type": "log"
    }
  ]
}
```

### Common Design Patterns

- **Trigger → Company Search → CRM Enrichment:** Fetch official company registration (ICE) and populate supplier or customer profile data automatically.
- **Company Search → If Node:** Verify whether a company's ICE or RC exists before processing invoices or contracts.
- **Combined Search → Log:** Compare results between Maroc Facture and Charika.ma for complete business intelligence.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Company Name is required`

**Cause:** The `companyName` parameter was left blank or resolved to `null`/`undefined`.

**Solution:** Provide a valid company search string in `companyName` or ensure your expression (`{{input.companyName}}`) resolves to a non-empty string.

---

#### `No company found matching the provided name`

**Cause:** Neither Maroc Facture nor Charika.ma found a record matching the search query.

**Solution:**
- Verify the company name spelling or try searching with keywords (e.g., `"Maroc Telecom"` instead of full legal suffix).
- Switch the `source` parameter to `"combined"` to broaden search coverage across both databases.

---

#### `API Connection Error`

**Cause:** The external provider (Maroc Facture or Charika.ma) is temporarily unreachable or rate-limiting requests.

**Solution:** Check network connectivity and retry the workflow execution.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](./http-request.md) – Perform custom API queries to external business registries
- [Function](./function.md) – Parse and transform company search payload data
- [If](./if.md) – Route workflows based on company lookup outcomes

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-13 | Initial release of Company Search node documentation |

<!-- /SECTION: changelog -->

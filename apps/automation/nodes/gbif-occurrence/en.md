---
node_id: "gbif-occurrence"
title: "GBIF Occurrence"
description: "Search biodiversity occurrence records from the GBIF API."
category: "Biodiversity"
version: "1.0.0"
language: "en"
last_updated: "2026-08-11"
author: "Fusion Team"
tags:

- gbif
- biodiversity
- occurrence
- species
- taxonomy
- wildlife
- ecology
- api
- scientific-name
- country
- pagination
- action

related_nodes:
- function
- if
- http-request

---

# GBIF Occurrence

> **Category:** biodiversity-nodes | **Type:** Action Node

Search biodiversity occurrence records from the **Global Biodiversity Information Facility (GBIF)** API.

The **GBIF Occurrence** node queries the GBIF occurrence search API and retrieves biodiversity occurrence records based on an optional scientific name and country filter.

The node returns structured occurrence data including taxonomy, geographic coordinates, event dates, institution information, collection information, dataset information, and direct GBIF occurrence URLs.

The node supports pagination using the `limit` and `offset` parameters.

### Supported Features

- Search biodiversity occurrence records from GBIF
- Filter occurrences by scientific name
- Filter occurrences by country
- Limit the number of returned records
- Paginate results using `offset`
- Return total matching records
- Detect the end of available records
- Extract taxonomic information
- Extract geographic coordinates
- Extract occurrence dates
- Extract basis of record
- Extract institution information
- Extract collection information
- Extract dataset identifiers
- Generate direct GBIF occurrence URLs
- Accept scientific name from workflow input
- Return normalized structured JSON data

### Use Cases

- Search for species occurrence records
- Find animal sightings in a specific country
- Research species distributions
- Build biodiversity monitoring workflows
- Build wildlife observation applications
- Analyze geographic species distributions
- Create biodiversity dashboards
- Store occurrence records in a database
- Filter biodiversity records using an `If` node
- Transform occurrence data using a `Function` node
- Build ecological research workflows

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| ---------- | ---- | -------- | ------- | ----------- |
| `scientificName` | `string` | ❌ No | `""` | Scientific name of the species or taxon to search for. |
| `country` | `string` | ❌ No | `""` | Country in which to search for occurrences. The value is converted to uppercase before being sent to GBIF. |
| `limit` | `number` | ❌ No | `10` | Maximum number of occurrence records to return. |
| `offset` | `number` | ❌ No | `0` | Number of records to skip for pagination. |

### Configuration Schema

```typescript
const schema: SchemaTypeAny = v.object({
  scientificName: v.string().optional().default(""),
  country: v.string().optional().default(""),
  limit: v.number().optional().default(10),
  offset: v.number().optional().default(0),
});
```

---

## GBIF API

The node uses the following GBIF occurrence search endpoint:

```text
https://api.gbif.org/v1/occurrence/search
```

The request uses the following query parameters:

| Parameter | Description |
| --------- | ----------- |
| `scientificName` | Filters occurrences by scientific name. |
| `country` | Filters occurrences by country code. |
| `limit` | Maximum number of records to return. |
| `offset` | Number of records to skip. |

The node sends the following HTTP header:

```text
Accept: application/json
```

The configured country value is converted to uppercase before being sent to GBIF.

For example:

```text
us
```

is converted to:

```text
US
```

---

## Inputs & Outputs

### Inputs

The node does not require workflow input when `scientificName` is configured.

If `scientificName` is empty and the incoming workflow data is a string, that string is used as the scientific name.

For example, if the previous node returns:

```text
Ursus arctos
```

the GBIF node uses:

```text
Ursus arctos
```

as the `scientificName` query parameter.

The input resolution follows this logic:

1. Use the configured `scientificName` if provided.
2. Otherwise, if workflow input is a string, use the input as the scientific name.
3. Otherwise, perform the search without a scientific name filter.

### Outputs

The node returns a structured biodiversity occurrence search result.

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Indicates whether the GBIF search was successful. |
| `query` | `object` | Contains the parameters used for the search. |
| `total_results` | `number` | Total number of matching occurrence records. |
| `offset` | `number` | Offset used for the current result page. |
| `limit` | `number` | Number of records requested. |
| `end_of_records` | `boolean` | Indicates whether the end of the available records has been reached. |
| `occurrences` | `array` | List of biodiversity occurrence records. |
| `note` | `string` | Description of the returned information. |

### Query Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `scientificName` | `string \| undefined` | Scientific name used in the search. |
| `country` | `string \| undefined` | Country used as the geographic filter. |
| `limit` | `number` | Maximum number of records requested. |
| `offset` | `number` | Number of records skipped. |

### Occurrence Fields

Each occurrence may contain the following fields:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `key` | `number \| null` | GBIF occurrence identifier. |
| `occurrence_id` | `string \| null` | Original occurrence identifier. |
| `scientific_name` | `string \| null` | Scientific name associated with the occurrence. |
| `species` | `string \| null` | Species name. |
| `genus` | `string \| null` | Genus name. |
| `family` | `string \| null` | Family name. |
| `kingdom` | `string \| null` | Biological kingdom. |
| `country` | `string \| null` | Country where the occurrence was recorded. |
| `country_code` | `string \| null` | Country code returned by GBIF. |
| `coordinates` | `object` | Geographic coordinates of the occurrence. |
| `event_date` | `string \| null` | Date associated with the occurrence event. |
| `basis_of_record` | `string \| null` | Basis/source type of the biodiversity record. |
| `institution_code` | `string \| null` | Institution associated with the record. |
| `collection_code` | `string \| null` | Collection associated with the record. |
| `dataset_key` | `string \| null` | GBIF dataset identifier. |
| `gbif_url` | `string \| null` | Direct URL to the GBIF occurrence page. |

### Coordinates

The `coordinates` object contains:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `latitude` | `number \| null` | Decimal latitude of the occurrence. |
| `longitude` | `number \| null` | Decimal longitude of the occurrence. |

---

## Output Example

```json
{
  "success": true,
  "query": {
    "scientificName": "Ursus arctos",
    "country": "US",
    "limit": 10,
    "offset": 0
  },
  "total_results": 1250,
  "offset": 0,
  "limit": 10,
  "end_of_records": false,
  "occurrences": [
    {
      "key": 1234567890,
      "occurrence_id": "OBS-12345",
      "scientific_name": "Ursus arctos",
      "species": "Ursus arctos",
      "genus": "Ursus",
      "family": "Ursidae",
      "kingdom": "Animalia",
      "country": "United States",
      "country_code": "US",
      "coordinates": {
        "latitude": 44.428,
        "longitude": -110.588
      },
      "event_date": "2025-07-15",
      "basis_of_record": "HUMAN_OBSERVATION",
      "institution_code": "GBIF",
      "collection_code": "OBSERVATIONS",
      "dataset_key": "example-dataset-key",
      "gbif_url": "https://www.gbif.org/occurrence/1234567890"
    }
  ],
  "note": "Returns biodiversity occurrence records. Example: Find Brown Bear sightings in the US"
}
```

The exact occurrence fields and values depend on the data returned by the GBIF API.

---

## Configuration Examples

### Default Configuration

```json
{
  "scientificName": "",
  "country": "",
  "limit": 10,
  "offset": 0
}
```

### Search by Scientific Name

```json
{
  "scientificName": "Ursus arctos",
  "country": "",
  "limit": 10,
  "offset": 0
}
```

### Search by Scientific Name and Country

```json
{
  "scientificName": "Ursus arctos",
  "country": "US",
  "limit": 10,
  "offset": 0
}
```

### Search for Lions in Kenya

```json
{
  "scientificName": "Panthera leo",
  "country": "KE",
  "limit": 20,
  "offset": 0
}
```

### Pagination

First page:

```json
{
  "scientificName": "Canis lupus",
  "country": "US",
  "limit": 10,
  "offset": 0
}
```

Second page:

```json
{
  "scientificName": "Canis lupus",
  "country": "US",
  "limit": 10,
  "offset": 10
}
```

Third page:

```json
{
  "scientificName": "Canis lupus",
  "country": "US",
  "limit": 10,
  "offset": 20
}
```

---

## Workflow Integration

### Sample Workflow: Search Species

```json
{
  "nodes": [
    {
      "id": "gbif-occurrence",
      "type": "gbif-occurrence",
      "config": {
        "scientificName": "Ursus arctos",
        "country": "US",
        "limit": 10,
        "offset": 0
      }
    }
  ]
}
```

### Sample Workflow: Input → GBIF Occurrence

```json
{
  "nodes": [
    {
      "id": "input",
      "type": "input"
    },
    {
      "id": "gbif-occurrence",
      "type": "gbif-occurrence",
      "config": {
        "scientificName": "",
        "country": "US",
        "limit": 10,
        "offset": 0
      }
    }
  ]
}
```

If the input node produces:

```text
Ursus arctos
```

the GBIF node searches for:

```text
Ursus arctos
```

### Sample Workflow: GBIF → Function

```json
{
  "nodes": [
    {
      "id": "gbif-occurrence",
      "type": "gbif-occurrence",
      "config": {
        "scientificName": "Panthera leo",
        "country": "KE",
        "limit": 20,
        "offset": 0
      }
    },
    {
      "id": "process-occurrences",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: GBIF → If

```json
{
  "nodes": [
    {
      "id": "gbif-occurrence",
      "type": "gbif-occurrence",
      "config": {
        "scientificName": "Ursus arctos",
        "country": "US",
        "limit": 10,
        "offset": 0
      }
    },
    {
      "id": "filter-occurrences",
      "type": "if"
    }
  ]
}
```

### Common Patterns

- Input → GBIF Occurrence → Function
- GBIF Occurrence → Function → Database
- GBIF Occurrence → If → Notification
- GBIF Occurrence → Function → Data Processing
- GBIF Occurrence → Database → Analytics
- GBIF Occurrence → If → Filter Species
- GBIF Occurrence → Function → Geographic Analysis

---

## Search Process

The node performs the following operations:

1. Reads the node configuration.
2. Determines the scientific name to search for.
3. Uses workflow input as the scientific name when no configured name is provided.
4. Creates the GBIF API query parameters.
5. Adds `scientificName` when a value is provided.
6. Adds `country` when a value is provided.
7. Converts the country value to uppercase.
8. Sends a request to the GBIF occurrence search endpoint.
9. Checks whether the HTTP response was successful.
10. Parses the JSON response.
11. Extracts the returned occurrence records.
12. Maps GBIF fields into the normalized node output.
13. Generates direct GBIF occurrence URLs.
14. Returns the structured result.

---

## Pagination

The node supports pagination through the `limit` and `offset` parameters.

The `limit` parameter controls the maximum number of records requested:

```json
{
  "limit": 10
}
```

The `offset` parameter controls how many records are skipped:

```json
{
  "offset": 20
}
```

For example:

```json
{
  "scientificName": "Ursus arctos",
  "country": "US",
  "limit": 10,
  "offset": 20
}
```

requests up to 10 records starting from offset 20.

The response also includes:

```json
{
  "total_results": 1250,
  "offset": 20,
  "limit": 10,
  "end_of_records": false
}
```

The `end_of_records` value can be used by downstream workflow logic to determine whether additional pages are available.

---

## Scientific Name Search

The `scientificName` parameter can contain a species or taxonomic name.

Example:

```json
{
  "scientificName": "Ursus arctos"
}
```

Another example:

```json
{
  "scientificName": "Panthera leo"
}
```

If the parameter is empty:

```json
{
  "scientificName": ""
}
```

the node does not add a `scientificName` filter to the GBIF request.

If no scientific name is configured but workflow input contains a string, that string is used automatically.

---

## Country Filtering

The `country` parameter restricts occurrences to a specific country.

Example:

```json
{
  "country": "US"
}
```

The node converts the country value to uppercase before sending the request.

For example:

```text
us
```

becomes:

```text
US
```

If no country is configured, the country filter is not included in the GBIF request.

---

## Occurrence Data Mapping

The node normalizes selected GBIF fields into its output structure.

| GBIF Field | Node Output |
| ---------- | ----------- |
| `key` | `key` |
| `occurrenceID` | `occurrence_id` |
| `scientificName` | `scientific_name` |
| `species` | `species` |
| `genus` | `genus` |
| `family` | `family` |
| `kingdom` | `kingdom` |
| `country` | `country` |
| `countryCode` | `country_code` |
| `decimalLatitude` | `coordinates.latitude` |
| `decimalLongitude` | `coordinates.longitude` |
| `eventDate` | `event_date` |
| `basisOfRecord` | `basis_of_record` |
| `institutionCode` | `institution_code` |
| `collectionCode` | `collection_code` |
| `datasetKey` | `dataset_key` |
| `key` | `gbif_url` |

Missing values are converted to `null`.

---

## GBIF Occurrence URL

When a GBIF occurrence contains a valid `key`, the node generates a direct occurrence URL:

```text
https://www.gbif.org/occurrence/<key>
```

For example:

```text
https://www.gbif.org/occurrence/1234567890
```

If the occurrence does not contain a key, `gbif_url` is returned as `null`.

---

## Error Handling

If the GBIF API request fails, the node throws an error.

The HTTP response is checked using:

```typescript
if (!response.ok) {
  throw new Error(`GBIF API error: ${response.status}`);
}
```

The error is then wrapped by `handleTick()` using the following format:

```text
GBIF occurrence search failed: <error message>
```

For example:

```text
GBIF occurrence search failed: GBIF API error: 400
```

### API Error

**Cause**

The GBIF API returned a non-success HTTP status code.

**Solution**

Verify the request parameters and check whether the GBIF API is available.

---

### Invalid Search Parameters

**Cause**

The supplied scientific name, country, limit, or offset may not be accepted by the GBIF API.

**Solution**

Verify the configured values and ensure that numeric parameters contain valid numbers.

---

### No Occurrences Returned

**Cause**

The search did not match any biodiversity occurrence records.

**Solution**

Try a broader scientific name, remove the country filter, or increase the search range.

A successful response can contain:

```json
{
  "success": true,
  "total_results": 0,
  "occurrences": []
}
```

---

### Pagination Reached the End

**Cause**

The requested offset is beyond the available records.

**Solution**

Check the `end_of_records` field and use a smaller offset if additional records are expected.

---

## Notes

The node returns only selected fields from the GBIF API response.

The original GBIF response may contain additional information that is not included in the normalized `occurrences` output.

The `note` field included in the response is:

```text
Returns biodiversity occurrence records. Example: Find Brown Bear sightings in the US
```

---

## Related

- [Function](./function.md) – Transform and process biodiversity occurrence records
- [If](./if.md) – Filter and route occurrence data based on conditions
- [HTTP Request](./http-request.md) – Make requests to external APIs

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-11 | Initial release |

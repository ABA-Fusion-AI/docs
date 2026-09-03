---
node_id: "purpleair"

title: "PurpleAir"

description: "Get real-time air quality sensor data from the PurpleAir API."

category: "Environment / Air Quality"

version: "1.0.0"

language: "en"

last_updated: "2026-09-03"

author: "Fusion Team"

tags:

- purpleair

- air-quality

- sensors

- pm2.5

- environment

- pollution

- real-time-data

- api

related_nodes:

- http-request

- function

- if

---

**# PurpleAir**

> **\*\*Category:\*\*** environment-nodes | **\*\*Type:\*\*** Action Node

Retrieve real-time air quality sensor data using the **\*\*PurpleAir\*\*** API.

The **\*\*PurpleAir\*\*** node queries the PurpleAir `/v1/sensors` endpoint and returns sensor data in a normalized object format. The node allows workflows to select specific sensor fields, specify a location type, and optionally restrict results using geographic bounds.

The raw array-based sensor response returned by PurpleAir is converted into objects where each requested field is mapped to its corresponding value.

**### Supported Features**

\- Retrieve real-time PurpleAir sensor data

\- Select the sensor fields returned by the API

\- Retrieve PM2.5 and other available air-quality measurements

\- Filter sensors using PurpleAir `location_type`

\- Optionally restrict results using geographic bounds

\- Authenticate requests using a PurpleAir API key

\- Convert PurpleAir array-based sensor results into structured objects

\- Return API version and timestamp metadata

\- Return the number of sensors included in the response

**### Use Cases**

\- Monitor real-time PM2.5 levels

\- Build air-quality dashboards

\- Retrieve environmental sensor measurements

\- Monitor pollution levels for a geographic region

\- Feed air-quality measurements into workflow automation

\- Analyze PurpleAir sensor data using downstream Function or AI nodes

\- Filter PurpleAir sensors by location type or geographic bounds

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `fields` | `string` | ❌ No | `"name,pm2.5"` | Comma-separated PurpleAir sensor fields to request. |
| `api_key` | `string` | ✅ Yes | — | PurpleAir API key used to access the sensors endpoint. |
| `location_type` | `number` | ❌ No | `0` | PurpleAir location type value included in the API request. |
| `bounds` | `string` | ❌ No | `""` | Optional geographic bounds value. The parameter is only added to the request when the string is not empty. |

**### Fields Parameter**

The `fields` parameter is sent directly to PurpleAir as the `fields` query parameter.

The default value is:

```text
name,pm2.5
```

Additional PurpleAir fields can be requested by providing a comma-separated string.

For example:

```text
name,pm2.5,pm10,temperature,humidity
```

**### Location Type Parameter**

The configured `location_type` number is converted to a string and included in the API request.

The default value is:

```text
0
```

**### Bounds Parameter**

The `bounds` parameter is optional.

When `bounds` contains a non-empty value, it is appended to the PurpleAir request as the `bounds` query parameter.

When it is empty, the `bounds` parameter is not included in the request.

**---**

**## Operations**

The node exposes a single PurpleAir sensor lookup operation through its configuration.

| Operation | Endpoint | Method | Description |
| --------- | -------- | ------ | ----------- |
| Sensor lookup | `/v1/sensors` | `GET` | Retrieve PurpleAir sensor data using configured fields, API key, location type, and optional bounds. |

The complete request URL is constructed from:

```text
https://api.purpleair.com/v1/sensors
```

with URL query parameters generated using `URLSearchParams`.

**---**

**## Request Body Construction**

The node performs a `GET` request and does not send a request body.

The following query parameters are always constructed:

```text
fields
api_key
location_type
```

For example:

```text
fields=name%2Cpm2.5
api_key=<API_KEY>
location_type=0
```

If `bounds` contains a value, the following parameter is also included:

```text
bounds=<bounds>
```

The resulting request follows this structure:

```text
GET https://api.purpleair.com/v1/sensors?fields=<fields>&api_key=<api_key>&location_type=<location_type>&bounds=<bounds>
```

The request includes the following HTTP header:

```text
Content-Type: application/json
```

**---**

**## Inputs & Outputs**

**### Inputs**

The node does not use the incoming workflow `data` value.

All PurpleAir lookup parameters are read from the node configuration.

**### Outputs**

The node returns a normalized object with the following structure:

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` when the PurpleAir request completes successfully. |
| `api_version` | `string/null` | PurpleAir API version returned by the API, or `null` when unavailable. |
| `time_stamp` | `number/null` | API response timestamp, or `null` when unavailable. |
| `data_time_stamp` | `number/null` | Sensor data timestamp, or `null` when unavailable. |
| `count` | `number` | PurpleAir response count, or the number of returned sensor rows when the response count is unavailable or falsy. |
| `fields` | `string[]` | List of fields returned by PurpleAir. |
| `sensors` | `object[]` | Sensor rows converted from arrays into objects keyed by field name. |
| `note` | `string` | Static description of the returned real-time air-quality data. |

**### Sensor Mapping**

PurpleAir sensor rows are returned as arrays.

The node maps each value to the field at the same index.

For example, if PurpleAir returns:

```json
{
  "fields": ["sensor_index", "name", "pm2.5"],
  "data": [
    [12345, "Sensor A", 12.7]
  ]
}
```

the node converts the sensor row into:

```json
{
  "sensor_index": 12345,
  "name": "Sensor A",
  "pm2.5": 12.7
}
```

If a sensor array does not contain a value for a returned field index, that field is assigned:

```json
null
```

**---**

**## Output Example**

```json
{
  "success": true,
  "api_version": "V1.0.11-0.0.62",
  "time_stamp": 1756915200,
  "data_time_stamp": 1756915140,
  "count": 2,
  "fields": [
    "name",
    "pm2.5"
  ],
  "sensors": [
    {
      "name": "Sensor A",
      "pm2.5": 12.4
    },
    {
      "name": "Sensor B",
      "pm2.5": 18.7
    }
  ],
  "note": "Returns real-time air quality sensor data. Common fields: name, pm2.5, pm10, temperature, humidity"
}
```

The exact fields and sensor values depend on the configured `fields` parameter and the response returned by PurpleAir.

**---**

**## Configuration Examples**

**### Default Sensor Lookup**

```json
{
  "fields": "name,pm2.5",
  "api_key": "YOUR_PURPLEAIR_API_KEY",
  "location_type": 0,
  "bounds": ""
}
```

**### Request Additional Fields**

```json
{
  "fields": "name,pm2.5,pm10,temperature,humidity",
  "api_key": "YOUR_PURPLEAIR_API_KEY",
  "location_type": 0,
  "bounds": ""
}
```

**### Sensor Lookup With Bounds**

```json
{
  "fields": "name,pm2.5",
  "api_key": "YOUR_PURPLEAIR_API_KEY",
  "location_type": 0,
  "bounds": "YOUR_BOUNDS_VALUE"
}
```

**---**

**## Workflow Integration**

**### Sample Workflow: PurpleAir → Function**

```json
{
  "nodes": [
    {
      "id": "purpleair",
      "type": "purpleair",
      "config": {
        "fields": "name,pm2.5",
        "api_key": "YOUR_PURPLEAIR_API_KEY",
        "location_type": 0,
        "bounds": ""
      }
    },
    {
      "id": "process-air-quality",
      "type": "function"
    }
  ]
}
```

**### Sample Workflow: PurpleAir → If**

```json
{
  "nodes": [
    {
      "id": "purpleair",
      "type": "purpleair",
      "config": {
        "fields": "name,pm2.5",
        "api_key": "YOUR_PURPLEAIR_API_KEY",
        "location_type": 0
      }
    },
    {
      "id": "check-air-quality",
      "type": "if"
    }
  ]
}
```

**### Common Patterns**

\- PurpleAir → Function — transform or aggregate returned sensor values

\- PurpleAir → If — branch a workflow based on returned air-quality measurements

\- PurpleAir → Database — store sensor measurements for historical analysis

\- PurpleAir → Dashboard/API Response — expose normalized air-quality sensor data

\- Scheduled Workflow → PurpleAir → Processing — periodically retrieve current air-quality measurements

**---**

**## Error Handling**

**### Missing API Key**

```text
API key is required
```

The node checks `api_key` before calling `getSensorData()`.

The helper method also performs the same API-key validation.

Because `handleTick()` wraps errors generated during `getSensorData()`, helper errors are returned with the PurpleAir lookup prefix.

**### PurpleAir API Error**

If PurpleAir returns a non-success HTTP status, the helper throws:

```text
PurpleAir API error: <status>
```

Inside `handleTick()`, this becomes:

```text
PurpleAir lookup failed: PurpleAir API error: <status>
```

For example:

```text
PurpleAir lookup failed: PurpleAir API error: 401
```

**### General Lookup Error**

Errors thrown while calling the PurpleAir helper are wrapped as:

```text
PurpleAir lookup failed: <error message>
```

When the caught value is an `Error`, the node uses:

```text
error.message
```

Otherwise, the value is converted using:

```text
String(error)
```

**---**

**## Troubleshooting**

**### "API key is required"**

**\*\*Cause\*\***

The `api_key` configuration is empty or missing.

**\*\*Solution\*\***

Provide a PurpleAir API key in the `api_key` parameter.

---

**### "PurpleAir lookup failed: PurpleAir API error: 401"**

**\*\*Cause\*\***

The PurpleAir API rejected the supplied credentials.

**\*\*Solution\*\***

Verify that the configured `api_key` is valid and authorized to access the requested PurpleAir endpoint.

---

**### "PurpleAir lookup failed: PurpleAir API error: 400"**

**\*\*Cause\*\***

One or more query parameters may not be accepted by the PurpleAir API.

**\*\*Solution\*\***

Check the configured `fields`, `location_type`, and `bounds` values.

---

**### Empty Sensors Array**

**\*\*Cause\*\***

PurpleAir returned no sensor rows, or the response did not contain a `data` array.

**\*\*Solution\*\***

Verify the configured location type and optional bounds.

When PurpleAir does not provide `data`, the node uses:

```json
[]
```

as the sensor list.

---

**### Sensor Objects Have Missing Values**

**\*\*Cause\*\***

A sensor row contains fewer values than the number of fields returned by PurpleAir.

**\*\*Solution\*\***

The node automatically assigns `null` to any missing indexed field values.

**---**

**## Security**

The PurpleAir API key is included directly in the request URL query string as:

```text
api_key=<api_key>
```

This behavior is defined by the node implementation.

Because query-string values can appear in request logs, proxies, debugging tools, or monitoring systems, workflow environments should avoid exposing generated PurpleAir request URLs unnecessarily.

For production workflows:

\- Store the PurpleAir API key using secrets or protected configuration

\- Avoid hardcoding API keys directly into reusable workflow definitions

\- Restrict access to workflow execution logs when request URLs may contain credentials

\- Rotate PurpleAir API keys when exposure is suspected

The request is sent to the HTTPS PurpleAir endpoint:

```text
https://api.purpleair.com/v1/sensors
```

**---**

**## Notes**

The node uses the fixed PurpleAir API endpoint:

```text
https://api.purpleair.com/v1/sensors
```

The node does not expose an operation selector. Every execution performs a sensor lookup.

The node does not use incoming workflow `data`.

The default requested fields are:

```text
name,pm2.5
```

The default `location_type` is:

```text
0
```

The default `bounds` value is an empty string.

When `bounds` is empty, it is omitted from the request.

The node converts PurpleAir's array-based sensor rows into objects using the `fields` array returned by the API.

If `data.fields` is missing, the node uses an empty array.

If `data.data` is missing, the node uses an empty sensor array.

For each sensor row, missing indexed values are converted to `null`.

The returned `count` is calculated using:

```text
data.count || sensors.length
```

This means a truthy PurpleAir `count` value is used directly; otherwise, the node falls back to the number of returned sensor rows.

The returned timestamps use:

```text
data.time_stamp || null
data.data_time_stamp || null
```

The node does not:

\- Cache PurpleAir responses

\- Retry failed requests

\- Paginate results

\- Perform PM2.5 calculations

\- Convert measurement units

\- Aggregate sensor measurements

\- Automatically select fields based on downstream workflow requirements

The `stop()` method performs no cleanup or cancellation logic.

**---**

**## Changelog**

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-03 | Initial release |

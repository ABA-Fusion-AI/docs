---
node_id: "quality-maintenance"
title: "Quality Assurance & Maintenance"
description: "Calculates quality assurance and maintenance metrics including Process Capability (Cp, Cpk), AQL Sample Size, Six Sigma Level, MTBF (Mean Time Between Failures), MTTR (Mean Time To Repair), Spare Parts Forecasting, and OSHA Incident Rate."
category: "business-commerce"
subcategory: "logistics-supply-chain"
version: "1.0.0"
language: "en"
last_updated: "2026-08-31"
author: "Fusion Team"
tags:
  - quality
  - maintenance
  - cpk
  - six-sigma
  - mtbf
  - mttr
  - osha
related_nodes:
  - production-metrics
  - inventory-logistics
  - mrp-engine
---

<!-- SECTION: header -->
# Quality Assurance & Maintenance

> **Category:** Business & Commerce | **Type:** Action Node

Calculate quality assurance and maintenance metrics including Process Capability (Cp and Cpk), AQL Sample Size, Six Sigma Level, MTBF, MTTR, Spare Parts Forecasting, and OSHA Incident Rate.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Quality Assurance & Maintenance** node provides common quality control, process capability, reliability, maintenance, forecasting, and workplace safety calculations through a single action node.

The calculation is selected through the `operation` parameter. Each operation exposes its own required configuration fields and returns an operation-specific result.

Supported operations:

- Process Capability (Cp and Cpk)
- AQL Sample Size
- Six Sigma Level
- MTBF (Mean Time Between Failures)
- MTTR (Mean Time To Repair)
- Spare Parts Forecasting
- OSHA Incident Rate

### Key Features

- Calculates process capability indices including Cp, Cpk, Cpu, and Cpl.
- Provides capability interpretations based on calculated Cp and Cpk values.
- Estimates an AQL sample size from the configured lot size.
- Calculates DPMO and an approximated Six Sigma level.
- Calculates Mean Time Between Failures (MTBF).
- Calculates Mean Time To Repair (MTTR).
- Forecasts spare-parts usage using a moving average.
- Calculates OSHA incident rates per 200,000 working hours.
- Supports structured historical usage data for spare-parts forecasting.
- Runs locally without requiring an external API.

### Processing Flow

```text
Select operation
        ↓
Load operation-specific parameters
        ↓
Validate input values
        ↓
Perform quality or maintenance calculation
        ↓
Build operation-specific result
        ↓
Return result
```

### Use Cases

- Manufacturing quality assurance.
- Process capability analysis.
- Quality-control sampling.
- Six Sigma monitoring.
- Equipment reliability analysis.
- Maintenance performance monitoring.
- Spare-parts demand forecasting.
- Workplace incident-rate calculations.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Operation

The `operation` parameter determines which calculation is performed.

Supported values:

```text
cpk
aqlSampleSize
sigmaLevel
mtbf
mttr
sparePartsForecast
oshaRate
```

The default operation is:

```text
cpk
```

---

### Process Capability (Cp and Cpk)

Select:

```text
operation: cpk
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mean` | `number` | Yes | Process mean value. |
| `stdDev` | `number` | Yes | Process standard deviation. |
| `usl` | `number` | Yes | Upper Specification Limit. |
| `lsl` | `number` | Yes | Lower Specification Limit. |

The node calculates:

```text
Cpu = (USL - mean) / (3 × stdDev)
Cpl = (mean - LSL) / (3 × stdDev)
Cp = (USL - LSL) / (6 × stdDev)
Cpk = min(Cpu, Cpl)
```

Example:

```text
mean: 100
stdDev: 2
usl: 106
lsl: 94
```

Output:

```json
{
  "cp": 1,
  "cpk": 1,
  "cpu": 1,
  "cpl": 1,
  "interpretation": {
    "cp": "Marginally Capable",
    "cpk": "Marginally Capable"
  }
}
```

Cp interpretation:

```text
Cp >= 1.33  → Capable
Cp >= 1.00  → Marginally Capable
Cp < 1.00   → Not Capable
```

Cpk interpretation:

```text
Cpk >= 1.33 → Centered and Capable
Cpk >= 1.00 → Marginally Capable
Cpk < 1.00  → Not Capable
```

---

### AQL Sample Size

Select:

```text
operation: aqlSampleSize
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `lotSize` | `number` | Yes | Number of items in the lot. Minimum value is `1`. |

The implementation determines the sample size from the configured lot size.

| Lot Size | Sample Size |
|----------|------------:|
| 1–8 | 2 |
| 9–15 | 3 |
| 16–25 | 5 |
| 26–50 | 8 |
| 51–90 | 13 |
| 91–150 | 20 |
| 151–280 | 32 |
| 281–500 | 50 |
| 501–1200 | 80 |
| Greater than 1200 | 125 |

The sampling rate is calculated as:

```text
samplingRate = (sampleSize / lotSize) × 100
```

Example:

```text
lotSize: 500
```

Output:

```json
{
  "sampleSize": 50,
  "lotSize": 500,
  "samplingRate": 10,
  "unit": "items"
}
```

---

### Six Sigma Level

Select:

```text
operation: sigmaLevel
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `defects` | `number` | Yes | Number of defects. |
| `opportunities` | `number` | Yes | Number of opportunities for defects. Minimum value is `1`. |

The node first calculates Defects Per Million Opportunities:

```text
DPMO = (defects / opportunities) × 1,000,000
```

The implementation then estimates the Sigma level.

For a DPMO value less than or equal to `3.4`, the Sigma level is `6`.

For a DPMO value greater than `690000`, the Sigma level is `1`.

For values between those limits, the implementation uses:

```text
sigmaLevel =
0.8406 + sqrt(29.37 - 2.221 × ln(DPMO))
```

The result is constrained by the implementation and rounded to two decimal places.

Example:

```text
defects: 100
opportunities: 100000
```

Output:

```json
{
  "sigmaLevel": 4.59,
  "dpmo": 1000,
  "defects": 100,
  "opportunities": 100000,
  "defectRate": 0.1
}
```

The defect rate is calculated as:

```text
defectRate = (defects / opportunities) × 100
```

---

### MTBF

Select:

```text
operation: mtbf
```

MTBF represents **Mean Time Between Failures**.

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `totalUptime` | `number` | Yes | Total uptime in hours. |
| `failures` | `number` | Yes | Number of failures. |

When failures are present:

```text
MTBF = totalUptime / failures
```

Example:

```text
totalUptime: 1000
failures: 5
```

Output:

```json
{
  "mtbf": 200,
  "totalUptime": 1000,
  "failures": 5,
  "unit": "hours"
}
```

When `failures` is `0`, the implementation returns the complete `totalUptime` as the MTBF value.

---

### MTTR

Select:

```text
operation: mttr
```

MTTR represents **Mean Time To Repair**.

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `totalDowntime` | `number` | Yes | Total downtime in hours. |
| `failuresMttr` | `number` | Yes | Number of failures used for the MTTR calculation. |

When failures are present:

```text
MTTR = totalDowntime / failuresMttr
```

Example:

```text
totalDowntime: 20
failuresMttr: 4
```

Output:

```json
{
  "mttr": 5,
  "totalDowntime": 20,
  "failures": 4,
  "unit": "hours"
}
```

When `failuresMttr` is `0`, the implementation returns an MTTR value of `0`.

---

### Spare Parts Forecast

Select:

```text
operation: sparePartsForecast
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `history` | `number[]` | Yes | Historical spare-parts usage values. At least one value is required. |
| `period` | `number` | No | Number of recent historical values used for the moving average. Default is `3`. |

The node takes the last `period` values from `history` and calculates their arithmetic mean.

Calculation:

```text
forecast = sum(last period values) / period
```

Example:

```text
history: [10, 12, 14, 16, 18]
period: 3
```

The values used are:

```text
[14, 16, 18]
```

Calculation:

```text
(14 + 16 + 18) / 3 = 16
```

Output:

```json
{
  "forecast": 16,
  "period": 3,
  "historyUsed": [
    14,
    16,
    18
  ],
  "unit": "units"
}
```

The forecast is rounded to two decimal places.

If the available history contains fewer values than the configured period, the node returns a result containing `forecast: null` and information about the insufficient history.

---

### OSHA Incident Rate

Select:

```text
operation: oshaRate
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `incidents` | `number` | Yes | Number of incidents. |
| `totalHours` | `number` | Yes | Total hours worked. |

Calculation:

```text
OSHA Rate = (incidents × 200000) / totalHours
```

Example:

```text
incidents: 3
totalHours: 100000
```

Output:

```json
{
  "oshaRate": 6,
  "incidents": 3,
  "totalHours": 100000,
  "unit": "incidents per 200,000 hours"
}
```

The OSHA incident rate is rounded to two decimal places.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses operation-specific configured parameters.

The `operation` parameter determines which additional parameters are required.

| Operation | Required Parameters |
|-----------|---------------------|
| `cpk` | `mean`, `stdDev`, `usl`, `lsl` |
| `aqlSampleSize` | `lotSize` |
| `sigmaLevel` | `defects`, `opportunities` |
| `mtbf` | `totalUptime`, `failures` |
| `mttr` | `totalDowntime`, `failuresMttr` |
| `sparePartsForecast` | `history` |
| `oshaRate` | `incidents`, `totalHours` |

For `sparePartsForecast`, `period` is optional and defaults to `3`.

### Outputs

The returned object depends on the selected operation.

| Operation | Main Output |
|-----------|-------------|
| `cpk` | `cp`, `cpk`, `cpu`, `cpl`, `interpretation` |
| `aqlSampleSize` | `sampleSize`, `lotSize`, `samplingRate`, `unit` |
| `sigmaLevel` | `sigmaLevel`, `dpmo`, `defects`, `opportunities`, `defectRate` |
| `mtbf` | `mtbf`, `totalUptime`, `failures`, `unit` |
| `mttr` | `mttr`, `totalDowntime`, `failures`, `unit` |
| `sparePartsForecast` | `forecast`, `period`, `historyUsed`, `unit` |
| `oshaRate` | `oshaRate`, `incidents`, `totalHours`, `unit` |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Process Capability

**Configuration**

```text
operation: cpk
mean: 100
stdDev: 2
usl: 106
lsl: 94
```

**Output**

```json
{
  "cp": 1,
  "cpk": 1,
  "cpu": 1,
  "cpl": 1,
  "interpretation": {
    "cp": "Marginally Capable",
    "cpk": "Marginally Capable"
  }
}
```

### Example 2: AQL Sample Size

**Configuration**

```text
operation: aqlSampleSize
lotSize: 500
```

**Output**

```json
{
  "sampleSize": 50,
  "lotSize": 500,
  "samplingRate": 10,
  "unit": "items"
}
```

### Example 3: Six Sigma Level

**Configuration**

```text
operation: sigmaLevel
defects: 100
opportunities: 100000
```

**Output**

```json
{
  "sigmaLevel": 4.59,
  "dpmo": 1000,
  "defects": 100,
  "opportunities": 100000,
  "defectRate": 0.1
}
```

### Example 4: MTBF

**Configuration**

```text
operation: mtbf
totalUptime: 1000
failures: 5
```

**Output**

```json
{
  "mtbf": 200,
  "totalUptime": 1000,
  "failures": 5,
  "unit": "hours"
}
```

### Example 5: MTTR

**Configuration**

```text
operation: mttr
totalDowntime: 20
failuresMttr: 4
```

**Output**

```json
{
  "mttr": 5,
  "totalDowntime": 20,
  "failures": 4,
  "unit": "hours"
}
```

### Example 6: Spare Parts Forecast

**Configuration**

```json
{
  "operation": "sparePartsForecast",
  "history": [
    10,
    12,
    14,
    16,
    18
  ],
  "period": 3
}
```

**Output**

```json
{
  "forecast": 16,
  "period": 3,
  "historyUsed": [
    14,
    16,
    18
  ],
  "unit": "units"
}
```

### Example 7: OSHA Incident Rate

**Configuration**

```text
operation: oshaRate
incidents: 3
totalHours: 100000
```

**Output**

```json
{
  "oshaRate": 6,
  "incidents": 3,
  "totalHours": 100000,
  "unit": "incidents per 200,000 hours"
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Quality Maintenance Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Operation-Specific Fields Are Missing

Each operation requires different parameters.

Ensure that all required fields for the selected `operation` are configured.

### Standard Deviation Cannot Be Zero

For the `cpk` operation, the standard deviation is used as a divisor.

If:

```text
stdDev = 0
```

the node throws:

```text
Standard deviation cannot be zero
```

Use a standard deviation greater than `0`.

### Invalid Lot Size

The `aqlSampleSize` operation requires:

```text
lotSize >= 1
```

Values below `1` are rejected by schema validation.

### Invalid Opportunities

The `sigmaLevel` operation requires:

```text
opportunities >= 1
```

Values below `1` are rejected by schema validation.

### Insufficient Spare Parts History

The `sparePartsForecast` operation requires enough history values for the configured moving-average period.

For example:

```text
history: [10, 12]
period: 3
```

The node returns an object containing:

```json
{
  "forecast": null,
  "error": "Insufficient history data. Need at least 3 data points, but got 2",
  "historyLength": 2,
  "requiredPeriod": 3
}
```

Provide at least as many history values as the configured `period`.

### Total Hours Cannot Be Zero

For the `oshaRate` operation, `totalHours` is used as a divisor.

If:

```text
totalHours = 0
```

the node throws:

```text
Total hours worked cannot be zero
```

Use a value greater than `0`.

### Negative Values

Parameters representing standard deviation, lot size, defects, opportunities, uptime, failures, downtime, historical usage, incidents, and working hours are constrained by schema validation.

Where a minimum value of `0` is defined, negative values are rejected before calculation.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Production & Efficiency Metrics** — Calculate production, manufacturing, utilization, capacity, and efficiency metrics.
- **Inventory Logistics** — Support inventory and logistics-oriented workflows.
- **MRP Engine** — Support material requirements and production-planning workflows.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-31 | Initial documentation for the Quality Assurance & Maintenance node. |

<!-- /SECTION: changelog -->
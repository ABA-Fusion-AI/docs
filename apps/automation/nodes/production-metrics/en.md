---
node_id: "production-metrics"
title: "Production & Efficiency Metrics"
description: "Calculates various production and efficiency metrics including OEE, Takt Time, WIP, Scrap Rate, FPY, Cycle Time Variance, Utilization, SMED Analysis, Bottleneck Detection, and Shift Capacity Planning."
category: "mathematical-statistical-analysis"
subcategory: "statistics-experiments"
version: "1.0.0"
language: "en"
last_updated: "2026-08-31"
author: "Fusion Team"
tags:
  - production
  - efficiency
  - oee
  - takt-time
  - wip
  - smed
  - manufacturing
related_nodes:
  - quality-maintenance
  - probability
  - quantiles
---

<!-- SECTION: header -->
# Production & Efficiency Metrics

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate common production and manufacturing efficiency metrics including OEE, Takt Time, WIP, Scrap Rate, FPY, Cycle Time Variance, Utilization, SMED, Bottleneck Detection, and Shift Capacity.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Production & Efficiency Metrics** node provides multiple production and manufacturing calculations through a single action node.

The operation is selected through the `operation` parameter. Each operation exposes its own required configuration fields.

Supported operations:

- OEE
- Takt Time
- WIP
- Scrap Rate
- First Pass Yield (FPY)
- Cycle Time Variance
- Machine Utilization
- SMED Analysis
- Bottleneck Detection
- Shift Capacity Planning

### Key Features

- Calculates Overall Equipment Effectiveness (OEE).
- Calculates production Takt Time.
- Calculates Work in Progress (WIP).
- Calculates Scrap Rate.
- Calculates First Pass Yield (FPY).
- Calculates Cycle Time Variance.
- Calculates machine utilization.
- Separates internal and external SMED setup time.
- Detects the lowest-capacity production station.
- Calculates available shift capacity.
- Supports structured arrays for SMED steps and production stations.
- Runs entirely locally without external APIs.

### Processing Flow

```text
Select operation
        ↓
Load operation-specific parameters
        ↓
Validate input values
        ↓
Perform production metric calculation
        ↓
Build operation-specific result
        ↓
Return metric
```

### Use Cases

- Manufacturing KPI calculations.
- Production planning.
- Equipment effectiveness analysis.
- Lean manufacturing workflows.
- Quality monitoring.
- Production capacity estimation.
- Setup-time analysis.
- Bottleneck identification.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Operation

The `operation` parameter determines which calculation is performed.

Supported values:

```text
oee
taktTime
wip
scrapRate
fpy
cycleTimeVariance
utilization
smed
bottleneck
shiftCapacity
```

The default operation is:

```text
oee
```

---

### OEE

Select:

```text
operation: oee
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `runTime` | `number` | Yes | Actual machine run time in minutes. |
| `plannedTime` | `number` | Yes | Planned production time in minutes. |
| `idealCycleTime` | `number` | Yes | Ideal cycle time in minutes per part. |
| `totalParts` | `number` | Yes | Total number of produced parts. |
| `goodParts` | `number` | Yes | Number of good parts produced. |

The node calculates:

```text
availability = runTime / plannedTime
performance = (totalParts × idealCycleTime) / runTime
quality = goodParts / totalParts
oee = availability × performance × quality
```

Example:

```text
runTime: 400
plannedTime: 480
idealCycleTime: 0.5
totalParts: 700
goodParts: 665
```

Output:

```json
{
  "availability": 0.8333333333333334,
  "performance": 0.875,
  "quality": 0.95,
  "oee": 0.6927083333333334
}
```

If `plannedTime` is `0`, the implementation returns:

```json
{
  "availability": 0,
  "performance": 0,
  "quality": 0,
  "oee": 0
}
```

---

### Takt Time

Select:

```text
operation: taktTime
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `minutesAvailable` | `number` | Yes | Available production time in minutes. |
| `demandUnits` | `number` | Yes | Customer demand in units. |

Calculation:

```text
taktTime = minutesAvailable / demandUnits
```

Example:

```text
minutesAvailable: 480
demandUnits: 120
```

Output:

```json
{
  "taktTime": 4,
  "unit": "minutes per unit"
}
```

If `demandUnits` is `0`, the node returns `0`.

---

### WIP

Select:

```text
operation: wip
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `throughputRate` | `number` | Yes | Throughput rate in parts per time unit. |
| `leadTime` | `number` | Yes | Lead time in corresponding time units. |

Calculation:

```text
WIP = throughputRate × leadTime
```

Example:

```text
throughputRate: 50
leadTime: 2.5
```

Output:

```json
{
  "wip": 125,
  "unit": "parts"
}
```

---

### Scrap Rate

Select:

```text
operation: scrapRate
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `totalProduced` | `number` | Yes | Total number of produced parts. |
| `rejected` | `number` | Yes | Number of rejected parts. |

Calculation:

```text
scrapRate = (rejected / totalProduced) × 100
```

Example:

```text
totalProduced: 1000
rejected: 50
```

Output:

```json
{
  "scrapRate": 5,
  "unit": "percentage"
}
```

If `totalProduced` is `0`, the node returns `0`.

---

### First Pass Yield

Select:

```text
operation: fpy
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `totalEntered` | `number` | Yes | Total items entering the process. |
| `passedFirstTime` | `number` | Yes | Items passing without rework. |

Calculation:

```text
FPY = (passedFirstTime / totalEntered) × 100
```

Example:

```text
totalEntered: 1000
passedFirstTime: 920
```

Output:

```json
{
  "fpy": 92,
  "unit": "percentage"
}
```

If `totalEntered` is `0`, the node returns `0`.

---

### Cycle Time Variance

Select:

```text
operation: cycleTimeVariance
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `actualTime` | `number` | Yes | Actual cycle time. |
| `standardTime` | `number` | Yes | Standard cycle time. |

Calculation:

```text
variance = ((actualTime - standardTime) / standardTime) × 100
```

Example:

```text
actualTime: 12
standardTime: 10
```

Output:

```json
{
  "variance": 20,
  "unit": "percentage",
  "interpretation": "Slower than standard"
}
```

When the calculated ratio is greater than zero, the interpretation is:

```text
Slower than standard
```

Otherwise:

```text
Faster than standard
```

---

### Utilization

Select:

```text
operation: utilization
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `runTimeUtil` | `number` | Yes | Machine run time in minutes. |
| `totalAvailableTime` | `number` | Yes | Total available machine time in minutes. |

Calculation:

```text
utilization = (runTimeUtil / totalAvailableTime) × 100
```

Example:

```text
runTimeUtil: 360
totalAvailableTime: 480
```

Output:

```json
{
  "utilization": 75,
  "unit": "percentage"
}
```

---

### SMED Analysis

Select:

```text
operation: smed
```

`steps` must contain at least one setup step.

Each step contains:

| Field | Type | Description |
|-------|------|-------------|
| `type` | `internal` or `external` | Indicates whether the step requires the machine to be stopped. |
| `duration` | `number` | Duration in minutes. |

Example:

```json
[
  {
    "type": "internal",
    "duration": 20
  },
  {
    "type": "external",
    "duration": 15
  },
  {
    "type": "internal",
    "duration": 10
  }
]
```

Output:

```json
{
  "internal": 30,
  "external": 15,
  "totalSetup": 45,
  "unit": "minutes"
}
```

---

### Bottleneck Detection

Select:

```text
operation: bottleneck
```

`stations` must contain at least one station.

Each station contains:

| Field | Type | Description |
|-------|------|-------------|
| `name` | `string` | Production station name. |
| `capacity` | `number` | Capacity in parts per hour. |

The node selects the station with the lowest capacity.

Example:

```json
[
  {
    "name": "Cutting",
    "capacity": 120
  },
  {
    "name": "Assembly",
    "capacity": 80
  },
  {
    "name": "Packaging",
    "capacity": 100
  }
]
```

Output:

```json
{
  "bottleneck": "Assembly",
  "capacity": 80,
  "unit": "parts per hour",
  "allStations": [
    {
      "name": "Cutting",
      "capacity": 120
    },
    {
      "name": "Assembly",
      "capacity": 80
    },
    {
      "name": "Packaging",
      "capacity": 100
    }
  ]
}
```

---

### Shift Capacity

Select:

```text
operation: shiftCapacity
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `shiftHours` | `number` | Yes | Shift duration in hours. |
| `breakMinutes` | `number` | Yes | Total break time in minutes. |
| `partsPerMinute` | `number` | Yes | Production speed in parts per minute. |

Calculation:

```text
availableMinutes = shiftHours × 60 - breakMinutes
shiftCapacity = availableMinutes × partsPerMinute
```

Example:

```text
shiftHours: 8
breakMinutes: 60
partsPerMinute: 2
```

Output:

```json
{
  "shiftCapacity": 840,
  "availableMinutes": 420,
  "unit": "parts"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses operation-specific configured parameters.

The `operation` parameter determines which other parameters are required.

### Outputs

The returned object depends on the selected operation.

| Operation | Main Output |
|-----------|-------------|
| `oee` | `availability`, `performance`, `quality`, `oee` |
| `taktTime` | `taktTime`, `unit` |
| `wip` | `wip`, `unit` |
| `scrapRate` | `scrapRate`, `unit` |
| `fpy` | `fpy`, `unit` |
| `cycleTimeVariance` | `variance`, `unit`, `interpretation` |
| `utilization` | `utilization`, `unit` |
| `smed` | `internal`, `external`, `totalSetup`, `unit` |
| `bottleneck` | `bottleneck`, `capacity`, `unit`, `allStations` |
| `shiftCapacity` | `shiftCapacity`, `availableMinutes`, `unit` |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: OEE

**Configuration**

```text
operation: oee
runTime: 400
plannedTime: 480
idealCycleTime: 0.5
totalParts: 700
goodParts: 665
```

**Output**

```json
{
  "availability": 0.8333333333333334,
  "performance": 0.875,
  "quality": 0.95,
  "oee": 0.6927083333333334
}
```

### Example 2: Takt Time

**Configuration**

```text
operation: taktTime
minutesAvailable: 480
demandUnits: 120
```

**Output**

```json
{
  "taktTime": 4,
  "unit": "minutes per unit"
}
```

### Example 3: SMED

**Configuration**

```json
{
  "operation": "smed",
  "steps": [
    {
      "type": "internal",
      "duration": 20
    },
    {
      "type": "external",
      "duration": 15
    },
    {
      "type": "internal",
      "duration": 10
    }
  ]
}
```

**Output**

```json
{
  "internal": 30,
  "external": 15,
  "totalSetup": 45,
  "unit": "minutes"
}
```

### Example 4: Bottleneck Detection

**Configuration**

```json
{
  "operation": "bottleneck",
  "stations": [
    {
      "name": "Cutting",
      "capacity": 120
    },
    {
      "name": "Assembly",
      "capacity": 80
    },
    {
      "name": "Packaging",
      "capacity": 100
    }
  ]
}
```

**Output**

```json
{
  "bottleneck": "Assembly",
  "capacity": 80,
  "unit": "parts per hour",
  "allStations": [
    {
      "name": "Cutting",
      "capacity": 120
    },
    {
      "name": "Assembly",
      "capacity": 80
    },
    {
      "name": "Packaging",
      "capacity": 100
    }
  ]
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Production Metrics Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Operation-Specific Fields Are Missing

Each operation requires different parameters.

Ensure the required fields for the selected `operation` are configured.

### SMED Requires Steps

The `steps` parameter must contain at least one item.

Each item requires:

```text
type
duration
```

Supported types:

```text
internal
external
```

### Bottleneck Requires Stations

The `stations` parameter must contain at least one station.

Each station requires:

```text
name
capacity
```

### No Bottleneck Station

The schema requires at least one station.

The implementation also contains the fallback error:

```text
At least one station is required
```

if an empty station array reaches the calculation.

### Negative Values

Production metric numeric parameters use schema validation with a minimum value of `0`.

Negative values are rejected before calculation.

### Takt Time with Zero Demand

When:

```text
demandUnits = 0
```

the node returns:

```text
taktTime = 0
```

instead of dividing by zero.

### Scrap Rate with Zero Production

When:

```text
totalProduced = 0
```

the node returns:

```text
scrapRate = 0
```

### FPY with Zero Input

When:

```text
totalEntered = 0
```

the node returns:

```text
fpy = 0
```

### Planned Time Is Zero

When OEE uses:

```text
plannedTime = 0
```

the node returns all OEE metrics as `0`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Quality Maintenance** — Support quality and maintenance-oriented workflows.
- **Probability** — Calculate probability distribution functions.
- **Quantiles / Percentiles / IQR** — Calculate percentile and quartile statistics.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-31 | Initial documentation for the Production & Efficiency Metrics node. |

<!-- /SECTION: changelog -->

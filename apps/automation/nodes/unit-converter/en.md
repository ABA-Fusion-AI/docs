---
node_id: "unit-converter"
title: "Unit Converter"
description: "Universal unit converter for logistics and general use. Converts between weight, length, volume, area, temperature, time, and speed units with maximum precision."
category: "business-commerce"
subcategory: "logistics-supply-chain"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - unit-converter
  - conversion
  - logistics
  - weight
  - length
  - volume
  - temperature
  - speed
  - area
  - time
related_nodes:
  - bom-utils
  - currency-converter
  - geospatial-utils
  - math-expression
---

<!-- SECTION: header -->
# Unit Converter

> **Category:** Business & Commerce | **Subcategory:** Logistics & Supply Chain | **Type:** Action Node

Convert measurements across weight, distance, volume, surface area, temperature, time, and speed with high precision.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Unit Converter** node is a universal measurement conversion utility designed for international logistics, supply chain calculations, engineering workflows, and general data normalization.

It supports conversions across **7 physical measurement categories** (Weight, Length, Volume, Area, Temperature, Time, and Speed), featuring automatic unit category detection, affine temperature transformations (Celsius, Fahrenheit, Kelvin), and adaptive decimal precision formatting.

### Key Features

- **7 Measurement Categories:** Converts seamlessly between metric, imperial, and scientific units for Weight, Length, Volume, Area, Temperature, Time, and Speed.
- **Adaptive Precision Formatting:** Returns both the exact unrounded floating-point `value` and an automatically rounded `formatted` result with optimal precision (`precision`).
- **Affine Temperature Conversions:** Accurately converts between Celsius (`c`), Fahrenheit (`f`), and Kelvin (`k`) using scale and offset transformations.
- **Flexible Numeric Input:** Accepts raw numbers (e.g., `5`) or numeric strings (e.g., `"120"`) passed statically or dynamically from upstream nodes.
- **Category Validation:** Automatically verifies that source and target units belong to the same dimension, preventing invalid transformations (e.g., converting kilograms to kilometers).
- **Zero Latency:** Executes synchronously in memory with zero external API dependencies.

### Processing Flow

```text
Incoming Value & Units (Parameter or Workflow Trigger)
  ↓
Parse & Validate Value as Numeric
  ↓
Detect Category of fromUnit and toUnit
  ↓
Validate Category Compatibility (fromCategory === toCategory)
  ↓
Execute Conversion:
  ├─ Temperature → Affine conversion: (value - offset) / scale → Celsius → Target
  └─ Standard    → Factor conversion: value * (fromFactor / toFactor)
  ↓
Compute Optimal Precision & Formatted Number
  ↓
Emit Output Object { value, fromUnit, toUnit, category, precision, formatted }
```

### Use Cases

- **Cross-Border Logistics & Freight:** Convert package weights (lbs to kg) and dimensions (inches to cm) before submitting shipping manifests to DHL, FedEx, or UPS.
- **International eCommerce Stores:** Standardize product volume, dimensions, and weights for international storefronts.
- **IoT & Sensor Data Ingestion:** Normalize raw weather and industrial sensor telemetry (e.g., Fahrenheit to Celsius, knots to km/h, m/s).
- **Warehouse & Real Estate Calculations:** Convert land and warehouse storage surface areas between hectares, acres, and square meters.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `value` | `number \| string` | Yes | — | The numeric quantity to convert. Accepts numbers or numeric strings (e.g., `5`, `"100"`). |
| `fromUnit` | `string` | Yes | — | The source unit symbol (e.g., `kg`, `km`, `c`, `l`, `h`, `km/h`). |
| `toUnit` | `string` | Yes | — | The target unit symbol (must belong to the same category as `fromUnit`). |

---

### Supported Units by Category

#### 1. Weight / Mass (`weight`)

| Unit | Symbol | Base Factor (to kg) |
|------|--------|---------------------|
| Kilogram | `kg` | $1$ |
| Gram | `g` | $0.001$ |
| Milligram | `mg` | $0.000001$ |
| Metric Ton | `ton` | $1000$ |
| Pound | `lb` | $0.453592$ |
| Ounce | `oz` | $0.0283495$ |

#### 2. Length / Distance (`length`)

| Unit | Symbol | Base Factor (to m) |
|------|--------|--------------------|
| Meter | `m` | $1$ |
| Kilometer | `km` | $1000$ |
| Centimeter | `cm` | $0.01$ |
| Millimeter | `mm` | $0.001$ |
| Mile | `mi` | $1609.34$ |
| Yard | `yd` | $0.9144$ |
| Foot | `ft` | $0.3048$ |
| Inch | `in` | $0.0254$ |

#### 3. Volume (`volume`)

| Unit | Symbol | Base Factor (to l) |
|------|--------|-------------------|
| Liter | `l` | $1$ |
| Milliliter | `ml` | $0.001$ |
| Cubic Meter | `m3` | $1000$ |
| Cubic Centimeter | `cm3` | $0.001$ |
| US Gallon | `gal` | $3.78541$ |
| US Quart | `qt` | $0.946353$ |
| US Pint | `pt` | $0.473176$ |
| Fluid Ounce | `floz` | $0.0295735$ |

#### 4. Area / Surface (`area`)

| Unit | Symbol | Base Factor (to m²) |
|------|--------|---------------------|
| Square Meter | `m2` | $1$ |
| Square Kilometer | `km2` | $1000000$ |
| Square Centimeter | `cm2` | $0.0001$ |
| Hectare | `ha` | $10000$ |
| Acre | `acre` | $4046.86$ |
| Square Foot | `sqft` | $0.092903$ |
| Square Inch | `sqin` | $0.00064516$ |

#### 5. Temperature (`temperature`)

| Unit | Symbol | Conversion Formula to Celsius |
|------|--------|--------------------------------|
| Celsius | `c` | $T_C = T$ |
| Fahrenheit | `f` | $T_C = (T_F - 32) / 1.8$ |
| Kelvin | `k` | $T_C = T_K - 273.15$ |

#### 6. Time (`time`)

| Unit | Symbol | Base Factor (to seconds) |
|------|--------|--------------------------|
| Second | `s` | $1$ |
| Millisecond | `ms` | $0.001$ |
| Minute | `min` | $60$ |
| Hour | `h` | $3600$ |
| Day | `day` | $86400$ |
| Week | `week` | $604800$ |

#### 7. Speed (`speed`)

| Unit | Symbol | Base Factor (to m/s) |
|------|--------|----------------------|
| Meters per second | `m/s` | $1$ |
| Kilometers per hour | `km/h` | $0.277778$ |
| Miles per hour | `mi/h` | $0.44704$ |
| Knot | `knot` | $0.514444$ |

### Default Configuration

```json
{
  "value": 5,
  "fromUnit": "kg",
  "toUnit": "g"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow trigger or previous step payload. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Returned on successful conversion containing the converted value and metadata. |
| `error` | `object` | Returned if units are invalid, non-numeric, or from different categories. |

### Output Schema

```typescript
{
  value: number;          // Full floating-point converted value
  fromUnit: string;       // Source unit
  toUnit: string;         // Target unit
  originalValue: number;  // Initial input value
  category: string;       // Unit category ("weight", "length", "temperature", etc.)
  precision: number;      // Optimal decimal places used for formatting
  formatted: number;      // Value rounded to optimal precision
}
```

### Response Example

```json
{
  "value": 5000,
  "fromUnit": "kg",
  "toUnit": "g",
  "originalValue": 5,
  "category": "weight",
  "precision": 0,
  "formatted": 5000
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Weight Conversion (Kilograms to Grams)

```json
{
  "value": 5,
  "fromUnit": "kg",
  "toUnit": "g"
}
```

**Output:**
```json
{
  "value": 5000,
  "fromUnit": "kg",
  "toUnit": "g",
  "originalValue": 5,
  "category": "weight",
  "precision": 0,
  "formatted": 5000
}
```

### Example 2: Distance Conversion (Kilometers to Meters)

```json
{
  "value": 3,
  "fromUnit": "km",
  "toUnit": "m"
}
```

**Output:**
```json
{
  "value": 3000,
  "fromUnit": "km",
  "toUnit": "m",
  "originalValue": 3,
  "category": "length",
  "precision": 0,
  "formatted": 3000
}
```

### Example 3: Temperature Conversion (Celsius to Fahrenheit)

```json
{
  "value": 100,
  "fromUnit": "c",
  "toUnit": "f"
}
```

**Output:**
```json
{
  "value": 212,
  "fromUnit": "c",
  "toUnit": "f",
  "originalValue": 100,
  "category": "temperature",
  "precision": 1,
  "formatted": 212
}
```

### Example 4: Area Conversion (Hectares to Square Meters)

```json
{
  "value": 5,
  "fromUnit": "ha",
  "toUnit": "m2"
}
```

**Output:**
```json
{
  "value": 50000,
  "fromUnit": "ha",
  "toUnit": "m2",
  "originalValue": 5,
  "category": "area",
  "precision": 0,
  "formatted": 50000
}
```

### Example 5: Speed Conversion (km/h to m/s)

```json
{
  "value": 120,
  "fromUnit": "km/h",
  "toUnit": "m/s"
}
```

**Output:**
```json
{
  "value": 33.33336,
  "fromUnit": "km/h",
  "toUnit": "m/s",
  "originalValue": 120,
  "category": "speed",
  "precision": 1,
  "formatted": 33.3
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Convert units across weight, distance, temperature, volume, area, time, and speed
```

### Common Workflow Patterns

- **Logistics Package Standardization:** Webhook (Shopify Order: `weight: 12, unit: "lb"`) → Unit Converter (`fromUnit: "lb"`, `toUnit: "kg"`) → DHL / UPS Shipping API.
- **Weather Sensor Telemetry Normalization:** MQTT Trigger (`temperature: 75, unit: "f"`) → Unit Converter (`fromUnit: "f"`, `toUnit: "c"`) → InfluxDB / PostgreSQL Storage.
- **Warehouse Space Calculation:** Form Trigger → Unit Converter (`fromUnit: "sqft"`, `toUnit: "m2"`) → ERP Quote Generator.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Error: "Cannot convert between different categories: weight and length"

**Cause:** You attempted to convert between incompatible physical dimensions (for instance, converting `kg` to `km`).

**Solution:** Ensure both `fromUnit` and `toUnit` belong to the same category (e.g. both are weight units or both are length units).

### Error: "Value must be a valid number"

**Cause:** The `value` parameter contains non-numeric text (such as `"abc"` or `NaN`).

**Solution:** Verify that the incoming value is a valid integer or floating-point number.

### Error: "Both fromUnit and toUnit are required"

**Cause:** One or both unit fields were left blank.

**Solution:** Select valid source and target unit symbols from the dropdown list.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Product & BOM Management](../bom-utils/en.md) - Bill of materials and cost rollups
- [Currency Converter](../currency-converter/en.md) - Real-time foreign exchange conversions
- [Geospatial Utils](../geospatial-utils/en.md) - Geographic distance and coordinate transformations
- [Math Expression](../math-expression/en.md) - Evaluate custom mathematical formulas

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for Unit Converter node |

<!-- /SECTION: changelog -->

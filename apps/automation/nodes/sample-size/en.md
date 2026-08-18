---
node_id: "sample-size-power"
title: "Sample Size / Power"
description: "Calculates required sample size per group for A/B tests (proportions or means) with specified power and significance level."
category: "Statistics"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:

- statistics
- sample-size
- power-analysis
- ab-test
- experimentation
- proportions
- means
- significance
- hypothesis-testing

related_nodes:
- ab-test
- function
- if

---

# Sample Size / Power

> **Category:** statistics-nodes | **Type:** Action Node

Calculates the **required sample size per group** for an A/B test, before running the experiment — either for **proportions** (conversion-rate style tests) or **means** (continuous-metric tests).

The **Sample Size / Power** node performs a standard power-analysis calculation, given the expected effect size, desired significance level (`alpha`), and desired statistical power, and returns the approximate number of samples needed per group.

### Supported Features

- Sample size calculation for two-proportion tests (`ab_proportions`)
- Sample size calculation for two-sample mean tests (`means`)
- Configurable significance level (`alpha`)
- Configurable statistical power
- Two-sided or one-sided test support (proportions only)
- Input validation for proportions range, zero effect size, and power bounds
- Conditional parameter display based on the selected `type`

### Use Cases

- Plan the required sample size before launching an A/B test
- Determine how long an experiment needs to run to reach adequate statistical power
- Validate whether a planned experiment has enough traffic/users to detect a meaningful effect
- Compare sample size requirements across different expected effect sizes
- Feed the result into a scheduling or reporting workflow ahead of running [A/B Test Calculator](./ab-test.md)

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `type` | `enum` | ❌ No | `"ab_proportions"` | Calculation type: `ab_proportions` or `means`. |
| `p1` | `number` | ✅ Yes (if `type = ab_proportions`) | — | Proportion in group 1 (between 0 and 1). |
| `p2` | `number` | ✅ Yes (if `type = ab_proportions`) | — | Proportion in group 2 (between 0 and 1). |
| `sigma` | `number` | ✅ Yes (if `type = means`) | — | Standard deviation, assumed equal for both groups. |
| `delta` | `number` | ✅ Yes (if `type = means`) | — | Effect size — the expected difference in means. |
| `alpha` | `number` | ❌ No | `0.05` | Significance level, between 0 and 1. |
| `power` | `number` | ❌ No | `0.8` | Desired statistical power, between 0 and 1 (exclusive of 1). |
| `twoSided` | `boolean` | ❌ No | `true` | Whether the test is two-sided. Only applied for `ab_proportions`. |

Parameters for the `ab_proportions` branch (`p1`, `p2`) and the `means` branch (`sigma`, `delta`) are shown or hidden conditionally based on the selected `type`.

---

## Calculation Types

### A/B Proportions (`type: "ab_proportions"`)

Uses the standard two-proportion sample-size formula:

```text
delta = |p2 - p1|
pBar = (p1 + p2) / 2
n = (zAlpha * sqrt(2 * pBar * (1 - pBar)) + zBeta * sqrt(p1*(1-p1) + p2*(1-p2)))² / delta²
```

`zAlpha` is computed from `alpha`, adjusted for `twoSided` (uses `alpha / 2` for two-sided tests, `alpha` for one-sided). `zBeta` is computed directly from `power`.

**Validation:**
- `p1` and `p2` must each be strictly between 0 and 1 (exclusive).
- `p1` and `p2` must differ — a zero effect size is rejected.

### Means (`type: "means"`)

Uses the standard two-sample mean sample-size formula, always **two-sided**:

```text
n = 2 * ((zAlpha + zBeta) * sigma / delta)²
```

`zAlpha` always uses `alpha / 2` here — `twoSided` is not applied to this branch.

**Validation:**
- `delta` must be greater than 0.

### Common Validation

`power` must be strictly less than 1 for both types — the node throws if `power >= 1`.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input.

All configuration is provided through the node configuration.

### Outputs — Proportions

| Output | Type | Description |
| ------ | ---- | ----------- |
| `nPerGroupApprox` | `number` | Required sample size per group, rounded up. |
| `alpha` | `number` | Significance level used. |
| `power` | `number` | Statistical power used. |
| `p1` | `number` | Proportion in group 1, as configured. |
| `p2` | `number` | Proportion in group 2, as configured. |
| `delta` | `number` | Absolute difference between `p1` and `p2`, rounded to 6 decimal places. |
| `twoSided` | `boolean` | Whether the calculation used a two-sided test. |

### Outputs — Means

| Output | Type | Description |
| ------ | ---- | ----------- |
| `nPerGroupApprox` | `number` | Required sample size per group, rounded up. |
| `alpha` | `number` | Significance level used. |
| `power` | `number` | Statistical power used. |
| `sigma` | `number` | Standard deviation, as configured. |
| `delta` | `number` | Effect size, as configured (not rounded). |

---

## Output Example

### Proportions

```json
{
  "nPerGroupApprox": 3843,
  "alpha": 0.05,
  "power": 0.8,
  "p1": 0.1,
  "p2": 0.12,
  "delta": 0.02,
  "twoSided": true
}
```

### Means

```json
{
  "nPerGroupApprox": 64,
  "alpha": 0.05,
  "power": 0.8,
  "sigma": 8.5,
  "delta": 3.2
}
```

---

## Configuration Examples

### Default Configuration (Proportions)

```json
{
  "type": "ab_proportions",
  "p1": 0.1,
  "p2": 0.12,
  "alpha": 0.05,
  "power": 0.8,
  "twoSided": true
}
```

### Means Test

```json
{
  "type": "means",
  "sigma": 8.5,
  "delta": 3.2,
  "alpha": 0.05,
  "power": 0.8
}
```

### One-Sided Proportions Test

```json
{
  "type": "ab_proportions",
  "p1": 0.05,
  "p2": 0.07,
  "alpha": 0.05,
  "power": 0.8,
  "twoSided": false
}
```

### Higher Power Requirement

```json
{
  "type": "ab_proportions",
  "p1": 0.1,
  "p2": 0.12,
  "alpha": 0.01,
  "power": 0.9
}
```

---

## Workflow Integration

### Sample Workflow: Plan Sample Size

```json
{
  "nodes": [
    {
      "id": "sample-size-signup",
      "type": "sample-size-power",
      "config": {
        "type": "ab_proportions",
        "p1": 0.1,
        "p2": 0.12
      }
    }
  ]
}
```

### Sample Workflow: Sample Size → Function → Notification

```json
{
  "nodes": [
    {
      "id": "sample-size",
      "type": "sample-size-power",
      "config": {
        "type": "means",
        "sigma": 8.5,
        "delta": 3.2
      }
    },
    {
      "id": "estimate-duration",
      "type": "function"
    },
    {
      "id": "notify-team",
      "type": "notification"
    }
  ]
}
```

### Sample Workflow: Sample Size → A/B Test (planning-to-analysis pipeline)

```json
{
  "nodes": [
    {
      "id": "sample-size",
      "type": "sample-size-power",
      "config": {
        "type": "ab_proportions",
        "p1": 0.1,
        "p2": 0.12
      }
    },
    {
      "id": "ab-test-result",
      "type": "ab-test"
    }
  ]
}
```

### Common Patterns

- Sample Size / Power → Function (convert to days of traffic) → Notification — experiment planning
- Sample Size / Power → Database — log required sample sizes for experiment tracking
- Sample Size / Power → If (feasibility check) → Notification — flag underpowered experiments before launch

---

## Error Handling

### Power Too High

```text
power must be < 1
```

Raised for both calculation types when `power >= 1`.

### Invalid Proportions Range

```text
p1 and p2 must be between 0 and 1
```

Raised for `ab_proportions` when either `p1` or `p2` is not strictly between 0 and 1.

### Identical Proportions

```text
p1 and p2 must be different
```

Raised for `ab_proportions` when `p1` equals `p2` (zero effect size).

### Zero Effect Size (Means)

```text
delta must be > 0
```

Raised for `means` when `delta` is 0.

---

## Troubleshooting

### "power must be < 1"

**Cause**

`power` was set to `1` or higher, which is not a valid target (100% power is not achievable in a finite sample).

**Solution**

Use a realistic power target, commonly `0.8` or `0.9`.

---

### "p1 and p2 must be between 0 and 1"

**Cause**

`p1` or `p2` was set to `0`, `1`, or a value outside the `(0, 1)` range.

**Solution**

Use proportions strictly between 0 and 1, e.g. `0.1` for 10%.

---

### "p1 and p2 must be different"

**Cause**

`p1` and `p2` are equal, meaning there is no effect to detect — sample size is undefined (infinite) in this case.

**Solution**

Set `p2` to the smallest meaningful difference from `p1` you want the test to be able to detect (the Minimum Detectable Effect).

---

### "delta must be > 0"

**Cause**

`delta` (the expected difference in means) was set to `0` for a `means` calculation.

**Solution**

Set `delta` to the smallest meaningful difference in means you want the test to be able to detect.

---

### Very Large `nPerGroupApprox`

**Cause**

A small expected effect size (`delta`, or the gap between `p1` and `p2`) relative to the variance requires a much larger sample to detect reliably — this is expected statistical behavior, not an error.

**Solution**

Reconsider the Minimum Detectable Effect, lower the desired `power`, or accept a longer experiment duration.

---

## Security

The node performs no outbound HTTP requests and does not access external services.

All computation is performed locally from the provided configuration values.

No API key or authentication credential is required.

---

## Notes

The node returns a **planning estimate**, not an analysis of collected data — pair it with [A/B Test Calculator](./ab-test.md) once the experiment has run.

The node does not:

- Account for multiple comparisons or sequential testing corrections
- Support unequal group sizes (allocation ratio is assumed 1:1)
- Support a one-sided calculation for the `means` type (always two-sided)
- Validate that `sigma` reflects the actual metric's variance
- Round `nPerGroupApprox` down — it is always rounded **up** (`Math.ceil`) to ensure adequate power

---

## Related

- [A/B Test Calculator](./ab-test.md) – Analyze results once the planned experiment has run
- [Function](./function.md) – Convert sample size into estimated experiment duration
- [If](./if.md) – Route workflows based on feasibility of the required sample size

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-18 | Initial release |
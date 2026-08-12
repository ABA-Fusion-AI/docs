---
node_id: "ab-test"
title: "A/B Test Calculator"
description: "Performs A/B tests for proportions or means, calculating p-values, significance, and confidence intervals."
category: "Statistics"
version: "1.0.0"
language: "en"
last_updated: "2026-08-12"
author: "Fusion Team"
tags:

- statistics
- ab-test
- hypothesis-testing
- proportions
- means
- p-value
- confidence-interval
- significance
- z-test
- experimentation

related_nodes:
- function
- if
- http-request

---

# A/B Test Calculator

> **Category:** statistics-nodes | **Type:** Action Node

Performs statistical **A/B tests** on experiment data, either for **proportions** (conversion-rate style tests) or **means** (continuous-metric tests).

The **A/B Test Calculator** node runs a two-sample z-test on the provided group data, computes the z-statistic, p-value, statistical significance against a configurable alpha threshold, and an approximate confidence interval for the difference between groups.

### Supported Features

- A/B test for proportions (two-proportion z-test)
- A/B test for means (two-sample z-test)
- Configurable significance level (`alpha`)
- One-sided and two-sided alternative hypotheses
- Automatic p-value computation
- Automatic significance determination
- Confidence interval calculation for the difference
- Input validation (`xA <= nA`, `xB <= nB`)
- Degenerate-data protection (zero standard error)
- Conditional parameter display based on test `kind`

### Use Cases

- Evaluate conversion-rate experiments (e.g. button color, landing page variant)
- Compare click-through or signup rates between two groups
- Compare average revenue, session duration, or other continuous metrics between two variants
- Validate whether an experiment result is statistically significant
- Build automated experimentation / growth workflows
- Feed test results into a `If` node to trigger a rollout decision
- Log experiment outcomes to a database or notification channel

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `kind` | `enum` | ❌ No | `"proportions"` | A/B test type: `proportions` or `means`. |
| `xA` | `number` | ✅ Yes (if `kind = proportions`) | — | Successes in group A. |
| `nA` | `number` | ✅ Yes (if `kind = proportions`) | — | Sample size in group A. |
| `xB` | `number` | ✅ Yes (if `kind = proportions`) | — | Successes in group B. |
| `nB` | `number` | ✅ Yes (if `kind = proportions`) | — | Sample size in group B. |
| `meanA` | `number` | ✅ Yes (if `kind = means`) | — | Mean of group A. |
| `meanB` | `number` | ✅ Yes (if `kind = means`) | — | Mean of group B. |
| `sdA` | `number` | ✅ Yes (if `kind = means`) | — | Standard deviation of group A. |
| `sdB` | `number` | ✅ Yes (if `kind = means`) | — | Standard deviation of group B. |
| `nAMeans` | `number` | ✅ Yes (if `kind = means`) | — | Sample size in group A. |
| `nBMeans` | `number` | ✅ Yes (if `kind = means`) | — | Sample size in group B. |
| `alpha` | `number` | ❌ No | `0.05` | Significance level, between 0 and 1. |
| `alternative` | `enum` | ❌ No | `"two-sided"` | Alternative hypothesis: `two-sided`, `greater`, or `less`. |

Parameters for the `proportions` branch (`xA`, `nA`, `xB`, `nB`) and the `means` branch (`meanA`, `meanB`, `sdA`, `sdB`, `nAMeans`, `nBMeans`) are shown or hidden conditionally based on the selected `kind`.

---

## Test Types

### Proportions (`kind: "proportions"`)

Runs a **two-proportion z-test**, commonly used for conversion-rate experiments.

```text
pA = xA / nA
pB = xB / nB
diff = pB - pA
```

The pooled standard error is used for the z-statistic and p-value, while an **unpooled** standard error is used for the confidence interval.

**Validation:** the node throws an error if `xA > nA` or `xB > nB`, and if the pooled standard error is zero (degenerate data).

### Means (`kind: "means"`)

Runs a **two-sample z-test for means**, used for continuous metrics.

```text
diff = meanB - meanA
se = sqrt(sdA² / nAMeans + sdB² / nBMeans)
```

---

## Alternative Hypotheses

| Value | Description |
| ----- | ----------- |
| `two-sided` | Tests whether group B differs from group A in either direction. |
| `greater` | Tests whether group B is greater than group A. |
| `less` | Tests whether group B is less than group A. |

The p-value formula changes depending on the selected alternative:

```text
two-sided: pValue = 2 * (1 - normalCdf(|z|))
greater:   pValue = 1 - normalCdf(z)
less:      pValue = normalCdf(z)
```

---

## Inputs & Outputs

### Inputs

The node does not require workflow input.

All configuration is provided through the node configuration.

### Outputs — Proportions

| Output | Type | Description |
| ------ | ---- | ----------- |
| `test` | `string` | Always `"two_proportion_z"`. |
| `pA` | `number` | Observed proportion in group A. |
| `pB` | `number` | Observed proportion in group B. |
| `diff` | `number` | Difference between `pB` and `pA`. |
| `z` | `number` | Z-statistic. |
| `pValue` | `number` | Computed p-value. |
| `alpha` | `number` | Significance level used. |
| `significant` | `boolean` | Whether `pValue < alpha`. |
| `ciDiffApprox` | `number[]` | Approximate confidence interval for the difference (unpooled SE). |

### Outputs — Means

| Output | Type | Description |
| ------ | ---- | ----------- |
| `test` | `string` | Always `"two_sample_z_means"`. |
| `diff` | `number` | Difference between `meanB` and `meanA`. |
| `z` | `number` | Z-statistic. |
| `pValue` | `number` | Computed p-value. |
| `alpha` | `number` | Significance level used. |
| `significant` | `boolean` | Whether `pValue < alpha`. |
| `ciDiff` | `number[]` | Confidence interval for the difference. |

All numeric outputs are rounded (6 decimal places for values, 9 for `pValue`).

---

## Output Example

### Proportions

```json
{
  "test": "two_proportion_z",
  "pA": 0.1,
  "pB": 0.125,
  "diff": 0.025,
  "z": 1.34246,
  "pValue": 0.179457312,
  "alpha": 0.05,
  "significant": false,
  "ciDiffApprox": [-0.011473, 0.061473]
}
```

### Means

```json
{
  "test": "two_sample_z_means",
  "diff": 3.2,
  "z": 2.145123,
  "pValue": 0.031943211,
  "alpha": 0.05,
  "significant": true,
  "ciDiff": [0.284, 6.116]
}
```

---

## Configuration Examples

### Default Configuration (Proportions)

```json
{
  "kind": "proportions",
  "xA": 100,
  "nA": 1000,
  "xB": 125,
  "nB": 1000,
  "alpha": 0.05,
  "alternative": "two-sided"
}
```

### Means Test

```json
{
  "kind": "means",
  "meanA": 42.1,
  "meanB": 45.3,
  "sdA": 8.5,
  "sdB": 9.1,
  "nAMeans": 200,
  "nBMeans": 210,
  "alpha": 0.05,
  "alternative": "two-sided"
}
```

### One-Sided Test (Greater)

```json
{
  "kind": "proportions",
  "xA": 80,
  "nA": 800,
  "xB": 100,
  "nB": 800,
  "alpha": 0.01,
  "alternative": "greater"
}
```

### Stricter Significance Level

```json
{
  "kind": "means",
  "meanA": 10.0,
  "meanB": 10.8,
  "sdA": 2.0,
  "sdB": 2.1,
  "nAMeans": 500,
  "nBMeans": 500,
  "alpha": 0.01,
  "alternative": "two-sided"
}
```

---

## Workflow Integration

### Sample Workflow: Proportions Test

```json
{
  "nodes": [
    {
      "id": "ab-test-signup",
      "type": "ab-test",
      "config": {
        "kind": "proportions",
        "xA": 100,
        "nA": 1000,
        "xB": 125,
        "nB": 1000
      }
    }
  ]
}
```

### Sample Workflow: A/B Test → If

```json
{
  "nodes": [
    {
      "id": "ab-test-revenue",
      "type": "ab-test",
      "config": {
        "kind": "means",
        "meanA": 42.1,
        "meanB": 45.3,
        "sdA": 8.5,
        "sdB": 9.1,
        "nAMeans": 200,
        "nBMeans": 210
      }
    },
    {
      "id": "check-significant",
      "type": "if"
    }
  ]
}
```

### Sample Workflow: A/B Test → Function → Notification

```json
{
  "nodes": [
    {
      "id": "ab-test",
      "type": "ab-test",
      "config": {
        "kind": "proportions",
        "xA": 100,
        "nA": 1000,
        "xB": 125,
        "nB": 1000
      }
    },
    {
      "id": "format-result",
      "type": "function"
    }
  ]
}
```

### Common Patterns

- Schedule → Database (fetch experiment data) → A/B Test → If → Notification
- A/B Test → Function → Format report
- A/B Test → Database → Store experiment history
- A/B Test → If → Trigger rollout / rollback decision

---

## Validation & Error Handling

### Proportions Validation

The node throws an error if:

```text
xA <= nA and xB <= nB required
```

when either `xA > nA` or `xB > nB`.

### Degenerate Data

If the pooled standard error is zero (e.g. identical, degenerate group data), the node throws:

```text
Zero standard error (degenerate data)
```

---

## Troubleshooting

### "xA <= nA and xB <= nB required"

**Cause**

The number of successes exceeds the sample size for group A or B in a proportions test.

**Solution**

Verify that `xA` does not exceed `nA`, and `xB` does not exceed `nB`.

---

### "Zero standard error (degenerate data)"

**Cause**

The pooled proportion produces a standard error of zero — typically when both groups have 0% or 100% success rates with the same pooled proportion.

**Solution**

Check the input data for degenerate cases (no variance between groups) before running the test.

---

### Result Never Significant

**Cause**

The observed difference between groups is small relative to the sample size, or `alpha` is set too low.

**Solution**

Increase the sample size, run the experiment longer, or review whether `alpha` matches your experimentation standard (commonly `0.05`).

---

### Unexpected Confidence Interval Width

**Cause**

Proportions tests use an **unpooled** standard error for the confidence interval, which differs from the **pooled** standard error used for the z-statistic and p-value. This is expected behavior and not a bug.

**Solution**

No action needed — this is standard practice for two-proportion z-tests.

---

## Security

The node performs no outbound HTTP requests and does not access external services.

All computation is performed locally from the provided configuration values.

No API key or authentication credential is required.

---

## Notes

The node returns computed statistical results rather than raw data.

The node does not:

- Fetch experiment data itself
- Store test results
- Generate visualizations
- Determine practical significance (only statistical significance)
- Adjust for multiple comparisons

It is intended to compute a single two-sample statistical test result for downstream workflow processing.

---

## Related

- [Function](./function.md) – Transform and format test results
- [If](./if.md) – Route workflows based on `significant` output
- [HTTP Request](./http-request.md) – Fetch or post experiment data externally

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-12 | Initial release |
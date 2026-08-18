---
node_id: "salary-converter"
title: "Salary Converter"
description: "Convert between annual, monthly, weekly, and hourly salary rates"
category: "Business & Commerce"
subcategory: "Finance & Accounting"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - salary
  - payroll
  - compensation
  - hourly
  - annual
  - monthly
  - weekly
  - converter
  - finance
  - hr
related_nodes:
  - currency-converter
  - loan-calculator
  - date-calculator
  - function
  - log
  - http-request
---

<!-- SECTION: header -->
# Salary Converter

> **Category:** Business & Commerce | **Type:** Action Node

Convert compensation amounts seamlessly across annual, monthly, weekly, and hourly salary rates with customizable working hours and annual work week schedules.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Salary Converter** node performs bidirectional compensation conversions across multiple payment frequencies. Whether you are standardizing job offer packages, budgeting contractor costs, calculating payroll schedules, or converting hourly wages into equivalent annual salaries, this node automates all wage calculations inside your workflow.

The node takes a baseline salary `amount` and a source payment frequency `from` (`annual`, `monthly`, `weekly`, or `hourly`). It allows you to specify custom working schedules through `hoursPerWeek` (default: 40 hours) and `weeksPerYear` (default: 52 weeks), returning a complete, normalized breakdown of equivalent rates across all standard time horizons.

### Key Features

- **Bidirectional Rate Conversion:** Convert from any base frequency (`annual`, `monthly`, `weekly`, `hourly`) to all equivalent frequencies in a single step.
- **Customizable Work Schedules:** Configure standard 40-hour full-time workweeks, part-time hours (e.g. 20–35 hrs/week), or seasonal contract schedules (e.g. 48 weeks/year).
- **Comprehensive Pay Frequency Breakdown:** Generates annual, monthly, bi-weekly, semi-monthly, weekly, daily, and hourly rate breakdowns.
- **Standardized Payroll Normalization:** Handles standard corporate payroll rules (12 months/year, 52 weeks/year, 26 bi-weekly periods, 24 semi-monthly periods).
- **High-Precision Calculations:** Returns clean numeric amounts rounded to two decimal places for currency compatibility.
- **Dynamic Expression Support:** Bind candidate compensation fields, timesheet hours, or ERP invoice lines directly from incoming workflow variables.

### Conversion Logic & Flow

```text
Input Parameters (amount: 50000, from: "annual", hours: 40, weeks: 52)
                               ↓
                 Normalize to Annual Base Salary
       ┌───────────────────────┼───────────────────────┐
       ↓                       ↓                       ↓
From Annual:             From Monthly:            From Hourly:
annual = amount          annual = amount * 12    annual = amount * hours * weeks
                               ↓
              Compute All Equivalent Frequency Rates
  ┌───────────────┬───────────────┬───────────────┬───────────────┐
  ↓               ↓               ↓               ↓               ↓
Monthly:        Bi-Weekly:      Weekly:         Daily:          Hourly:
annual / 12     annual / 26     annual / weeks  weekly / 5      annual / (weeks * hours)
                               ↓
         Structured JSON Output (All Rates & Metadata)
```

### Use Cases

- **HR Recruitment & Job Offers:** Convert an applicant's requested hourly wage into equivalent annual or monthly compensation packages during onboarding.
- **Contractor & Freelancer Budgeting:** Project the annual and monthly expenditure for external contractors billed at an hourly or weekly rate.
- **Payroll & Benefits Automation:** Calculate bi-weekly and semi-monthly pay stubs from base annual salaries.
- **Part-Time & Flexible Work Modeling:** Accurately estimate payroll costs for part-time employees working non-standard hours (e.g. 30 hours/week across 48 weeks).
- **ERP & Accounting Integration:** Standardize salary representations across HRIS platforms (e.g. BambooHR, Workday, Deel) and finance systems.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `amount` | `number` | ✅ Yes | `50000` | The compensation amount to convert (e.g., `50000`, `5000`, `30`). |
| `from` | `string` | ✅ Yes | `annual` | Source frequency of the input amount: `annual`, `monthly`, `weekly`, or `hourly`. |
| `hoursPerWeek` | `number` | ❌ No | `40` | Average working hours per week (e.g. `40` for full-time, `20`–`35` for part-time). |
| `weeksPerYear` | `number` | ❌ No | `52` | Number of paid work weeks per year (typically `52`, or `48`–`50` for contract/seasonal work). |

---

### Parameter Details

#### `amount`
The monetary value to be converted.
- **Type:** `number`
- **Required:** Yes
- **Allowed Values:** Any non-negative number (`>= 0`).
- **Example:** `50000` (for annual salary), `5000` (for monthly pay), `800` (for weekly pay), `35` (for hourly rate).

#### `from`
Specifies the payment interval of the provided `amount`.
- **Type:** `string` (Dropdown)
- **Options:**
  - `annual` — Input amount represents total yearly compensation (1 year = 12 months = 52 weeks).
  - `monthly` — Input amount represents monthly pay (12 pay periods per year).
  - `weekly` — Input amount represents pay per week (`weeksPerYear` periods per year).
  - `hourly` — Input amount represents pay per single worked hour.
- **Default:** `annual`

#### `hoursPerWeek`
Defines the number of billable or scheduled working hours in a standard work week.
- **Type:** `number`
- **Default:** `40`
- **Recommended Values:** `40` (Standard Full-Time), `37.5` / `35` (Standard European/UK Full-Time), `20`–`30` (Part-Time).

#### `weeksPerYear`
Defines the number of active work weeks in a year used for annualized conversions.
- **Type:** `number`
- **Default:** `52`
- **Recommended Values:** `52` (Standard Annual Payroll), `50` (2 weeks unpaid leave), `48` (Seasonal or educational contracts).

---

### Conversion Formulas

| Output Frequency | Formula from Annual Base (`A`) | Description |
|:-----------------|:-------------------------------|:------------|
| **Annual** | $A$ | Total baseline yearly compensation |
| **Monthly** | $A / 12$ | 12 monthly pay periods per year |
| **Semi-Monthly** | $A / 24$ | Twice per month (24 pay periods per year) |
| **Bi-Weekly** | $A / 26$ | Every two weeks (26 pay periods per year) |
| **Weekly** | $A / \text{weeksPerYear}$ | Pay per standard calendar week |
| **Daily** | $\text{Weekly} / 5$ or $\text{Hourly} \times 8$ | Standard 8-hour workday equivalent |
| **Hourly** | $A / (\text{weeksPerYear} \times \text{hoursPerWeek})$ | Individual hourly rate equivalent |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream workflow trigger or data payload. Parameters can dynamically bind values via expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when salary conversion succeeds. Contains the full multi-frequency breakdown and calculation metadata. |
| `error` | `Error` | Emitted when input parameters are invalid or calculation fails. |

---

### Output Data Structure

The `success` output returns structured JSON containing normalized rates across all standard time horizons:

```json
{
  "success": true,
  "input": {
    "amount": 50000,
    "from": "annual",
    "hoursPerWeek": 40,
    "weeksPerYear": 52
  },
  "rates": {
    "annual": 50000,
    "monthly": 4166.67,
    "semiMonthly": 2083.33,
    "biweekly": 1923.08,
    "weekly": 961.54,
    "daily": 192.31,
    "hourly": 24.04
  },
  "summary": {
    "hoursPerWeek": 40,
    "weeksPerYear": 52,
    "totalHoursPerYear": 2080,
    "formatted": {
      "annual": "$50,000.00",
      "monthly": "$4,166.67",
      "weekly": "$961.54",
      "hourly": "$24.04"
    }
  }
}
```

---

### Output Fields Reference

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates successful execution. |
| `input.amount` | `number` | The original amount provided to the converter. |
| `input.from` | `string` | The source frequency mode (`annual`, `monthly`, `weekly`, `hourly`). |
| `rates.annual` | `number` | Total annualized salary amount. |
| `rates.monthly` | `number` | Monthly compensation amount (Annual / 12). |
| `rates.semiMonthly` | `number` | Semi-monthly pay amount (Annual / 24). |
| `rates.biweekly` | `number` | Bi-weekly pay amount (Annual / 26). |
| `rates.weekly` | `number` | Weekly pay amount (Annual / weeksPerYear). |
| `rates.daily` | `number` | Daily rate based on an 8-hour workday equivalent. |
| `rates.hourly` | `number` | Base hourly pay rate (Annual / totalHoursPerYear). |
| `summary.totalHoursPerYear` | `number` | Total productive working hours per year (`hoursPerWeek * weeksPerYear`). |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Convert Annual Salary to Hourly & Monthly

Convert a standard annual salary of `$50,000` into monthly, weekly, and hourly equivalents.

**Parameter Configuration:**

```text
Amount: 50000
From: annual
HoursPerWeek: 40
WeeksPerYear: 52
```

**Calculation Results:**
- **Annual:** `$50,000.00`
- **Monthly:** `$4,166.67`
- **Weekly:** `$961.54`
- **Hourly:** `$24.04` (based on 2,080 annual work hours)

---

### Example 2: Convert Monthly Compensation to Annual & Weekly

Calculate annualized earnings from a monthly base pay of `$5,000`.

**Parameter Configuration:**

```text
Amount: 5000
From: monthly
HoursPerWeek: 40
WeeksPerYear: 52
```

**Calculation Results:**
- **Annual:** `$60,000.00` (`5000 * 12`)
- **Monthly:** `$5,000.00`
- **Weekly:** `$1,153.85`
- **Hourly:** `$28.85`

---

### Example 3: Convert Contractor Hourly Rate to Annualized Income

Estimate the annual earnings and monthly budget for a freelance software engineer billing `$30/hour`.

**Parameter Configuration:**

```text
Amount: 30
From: hourly
HoursPerWeek: 40
WeeksPerYear: 52
```

**Calculation Results:**
- **Hourly:** `$30.00`
- **Weekly:** `$1,200.00` (`30 * 40`)
- **Monthly:** `$5,200.00`
- **Annual:** `$62,400.00` (`30 * 40 * 52`)

---

### Example 4: Part-Time Schedule with Custom Work Weeks

Convert a weekly wage of `$800` for a part-time role with 35 hours per week across 48 weeks per year.

**Parameter Configuration:**

```text
Amount: 800
From: weekly
HoursPerWeek: 35
WeeksPerYear: 48
```

**Calculation Results:**
- **Weekly:** `$800.00`
- **Annual:** `$38,400.00` (`800 * 48`)
- **Monthly:** `$3,200.00` (`38400 / 12`)
- **Hourly:** `$22.86` (`800 / 35`)

---

### Example 5: Automated Candidate Offer Pipeline

Calculate salary breakdowns dynamically upon receiving a candidate submission via webhook.

**Workflow Pipeline:**

```text
Job Application Webhook (candidate hourly expectation: $45)
  → Salary Converter (amount: {{outputs.Webhook.success.hourly_rate}}, from: "hourly")
  → Function (build formal offer letter payload)
  → PDF Generator (create offer letter PDF with monthly & annual rates)
  → SendGrid / Gmail (email offer letter to hiring manager)
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Convert between annual, monthly, weekly, and hourly salary rates
```

### Common Workflow Patterns

- **Recruitment Offer Generator:** `Manual / Webhook Trigger` → `Salary Converter` (`from: "hourly"`) → `Function` → `Generate Contract / Document`.
- **Contractor Timesheet Auditing:** `Timecard Webhook` → `Salary Converter` (`from: "hourly"`, `hoursPerWeek: 35`) → `Log` → `QuickBooks / Xero Invoice Creation`.
- **HR Compensation Review:** `Airtable / Postgres Read` (Employee records) → `For Each` → `Salary Converter` → `Financial Report Export`.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Zero or Negative Amounts
- **Symptom:** Node returns `$0.00` for all rates or raises a validation warning.
- **Cause:** Input `amount` is set to `0` or a negative value.
- **Solution:** Verify upstream dynamic expressions ensure `amount` is a positive number.

#### Unexpected Hourly Rates on Part-Time Contracts
- **Symptom:** Hourly rate seems lower than expected for part-time staff.
- **Cause:** The default `hoursPerWeek` of `40` was left unchanged for a part-time employee working fewer hours.
- **Solution:** Explicitly set `hoursPerWeek` (e.g. `20` or `30`) to reflect the correct working schedule.

#### Mismatched Annual Weeks
- **Symptom:** Annual calculation does not match regional payroll tables.
- **Cause:** Some European or contractor payroll models use 48 or 50 active working weeks instead of the 52 calendar weeks.
- **Solution:** Adjust `weeksPerYear` to match the exact contractual paid weeks.

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Amount parameter is required` | The `amount` field was omitted or undefined | Supply a valid numeric salary amount |
| `Invalid frequency type` | `from` frequency was not one of the allowed options | Select `annual`, `monthly`, `weekly`, or `hourly` |
| `HoursPerWeek must be greater than 0` | `hoursPerWeek` was set to `0` or negative | Set a positive weekly hour count (e.g. `40`) |
| `WeeksPerYear must be greater than 0` | `weeksPerYear` was set to `0` or negative | Set a positive annual weeks count (e.g. `52`) |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Currency Converter](../currency-converter/en.md) — Convert monetary amounts across international exchange rates
- [Loan Calculator](../loan-calculator/en.md) — Calculate loan payments, interest, and amortization schedules
- [Date Calculator](../date-calculator/en.md) — Compute working days, payroll intervals, and date offsets
- [Function](../function/en.md) — Perform custom mathematical logic and tax deductions on salary payloads
- [Log](../log/en.md) — Print converted salary outputs to the debug console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial release supporting bidirectional conversions across annual, monthly, weekly, and hourly frequencies with custom schedule hours/weeks |

<!-- /SECTION: changelog -->

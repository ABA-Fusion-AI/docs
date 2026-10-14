---
node_id: "options-pricing"
title: "Options Pricing"
description: "Calculate option prices using Black-Scholes model with Greeks (delta, gamma, theta, vega)"
category: "Mathematical & Statistical Analysis"
subcategory: "Calculators & Models"
version: "1.0.0"
language: "en"
last_updated: "2026-10-14"
author: "Fusion Team"
tags:
  - options
  - black-scholes
  - greeks
  - delta
  - gamma
  - theta
  - vega
  - derivatives
  - finance
  - trading
  - calculators
related_nodes:
  - bond-pricing
  - loan-calculator
  - financial-ratio-analyzer
  - function
  - log
---

<!-- SECTION: overview -->
# Options Pricing

> **Category:** Mathematical & Statistical Analysis&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Calculate theoretical option prices and core risk sensitivities (The Greeks: Delta, Gamma, Theta, Vega) using the standard Black-Scholes-Merton model for European call and put options.

### What is the Options Pricing Node?

The **Options Pricing** node provides analytical valuation for European-style equity and index options. By taking real-time or historical market variables—current stock price ($S$), option strike price ($K$), time to expiry ($T$), risk-free interest rate ($r$), and implied volatility ($\sigma$)—the node solves the closed-form Black-Scholes differential equation and outputs:

1. **Option Premium (`price`):** The fair economic value of the contract.
2. **The Greeks (`delta`, `gamma`, `theta`, `vega`):** First- and second-order partial derivatives quantifying price sensitivity and risk exposure.
3. **Value Decomposition (`intrinsicValue`, `timeValue`):** Separation of the premium into immediate realizable payoff versus speculative time/volatility premium.

### Mathematical Framework (Black-Scholes-Merton)

The node computes the normalized standardized parameters $d_1$ and $d_2$:

$$d_1 = \frac{\ln(S / K) + \left(r + \frac{1}{2}\sigma^2\right)T}{\sigma \sqrt{T}}$$

$$d_2 = d_1 - \sigma \sqrt{T}$$

Where:
- $S$ = Current Stock / Underlying Price (`stockPrice`)
- $K$ = Strike Price (`strikePrice`)
- $T$ = Time to Expiration in years (`timeToExpiry`)
- $r$ = Risk-free rate expressed as a decimal ($r = \text{riskFreeRate} / 100$)
- $\sigma$ = Volatility expressed as a decimal ($\sigma = \text{volatility} / 100$)
- $N(x)$ = Cumulative standard normal distribution function (evaluated using high-precision polynomial approximation $|\epsilon| < 7.5 \times 10^{-8}$)

Using $d_1$ and $d_2$, the pricing formulas are:
- **European Call:** $C = S \cdot N(d_1) - K \cdot e^{-rT} \cdot N(d_2)$
- **European Put:** $P = K \cdot e^{-rT} \cdot N(-d_2) - S \cdot N(-d_1)$

### Key Features

- **Standard European Call & Put Pricing:** Closed-form analytical solution compliant with global quantitative finance benchmarks.
- **Full Analytical Greeks Suite:**
  - **Delta ($\Delta$):** Rate of change of the option price with respect to changes in the underlying asset's price ($\partial V / \partial S$).
  - **Gamma ($\Gamma$):** Curvature and rate of change of Delta ($\partial^2 V / \partial S^2$). Identical for both Calls and Puts.
  - **Theta ($\Theta$):** Annualized rate of time decay ($\partial V / \partial T$). Shows the expected premium drop as expiration approaches.
  - **Vega ($\nu$):** Sensitivity to shifts in implied volatility ($\partial V / \partial \sigma$). Identical for both Calls and Puts.
- **Intrinsic & Extrinsic Breakdown:** Automatically isolates $\max(0, S-K)$ or $\max(0, K-S)$ from remaining extrinsic time value.
- **Workflow Pipeline Ready:** Accepts static values, JSON payloads from webhooks, upstream market data APIs, or variables.

### Common Use Cases

- **Automated Options Screener:** Compare theoretical Black-Scholes fair values against live exchange order books (e.g., Yahoo Finance, Polygon, Alpaca) to flag mispriced contracts.
- **Delta-Neutral Portfolio Hedging:** Continuously recalculate portfolio Delta to determine how many underlying shares must be bought or sold to maintain a net-zero market directional exposure.
- **Risk Management & Greeks Dashboards:** Feed option parameters into automated reporting flows to monitor aggregate Gamma and Vega exposure across derivatives portfolios.
- **Trading Education & Algorithmic Simulators:** Construct interactive workflows that teach options pricing dynamics or test trading strategies before live capital deployment.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Bounds | Description |
|-----------|------|----------|---------|--------|-------------|
| `stockPrice` | `number` | ❌ No | `100` | $\ge 0$ | Current spot price of the underlying asset ($S$). |
| `strikePrice` | `number` | ❌ No | `100` | $\ge 0$ | Exercise/strike price of the option contract ($K$). |
| `timeToExpiry` | `number` | ❌ No | `0.25` | $0$ to $10$ | Time remaining until contract expiration expressed in **years** ($T$). For example: 3 months = `0.25`, 6 months = `0.5`, 1 month $\approx$ `0.0833`. |
| `riskFreeRate` | `number` | ❌ No | `5` | $0$ to $100$ | Annualized risk-free interest rate ($r$) entered as a **percentage** (e.g., `5` corresponds to 5% or 0.05). |
| `volatility` | `number` | ❌ No | `20` | $0$ to $500$ | Annualized volatility ($\sigma$) entered as a **percentage** (e.g., `20` corresponds to 20% or 0.20 implied volatility). |
| `type` | `enum` | ❌ No | `"call"` | `call`, `put` | Type of option contract: `"call"` (right to buy) or `"put"` (right to sell). |

> [!TIP]
> **Unit Conventions:**
> - Enter `volatility` and `riskFreeRate` directly as whole percentages (e.g., `20` for 20%, not `0.20`). The node automatically converts them to decimals internally.
> - Enter `timeToExpiry` as a fractional year. For daily options, divide remaining calendar days by 365 (e.g., 30 days = $30/365 \approx 0.0822$).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Trigger or upstream payload. Parameters configured on the node are evaluated when `input` arrives. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when calculation succeeds. Contains the option price, Greeks, and intrinsic/time value breakdown. |
| `error` | `object` | Emitted if calculation fails (e.g., missing configuration). |

### Output Data Schema (`success`)

When calculation succeeds, the node emits a structured payload:

```json
{
  "success": true,
  "price": 4.614988613066949,
  "delta": 0.5694603016957197,
  "gamma": 0.03928800094473793,
  "theta": -10.474152266772839,
  "vega": 19.644000472368965,
  "type": "call",
  "intrinsicValue": 0,
  "timeValue": 4.614988613066949
}
```

### Output Field Descriptions

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | `true` when the Black-Scholes calculation completed successfully. |
| `price` | `number` | Theoretical Black-Scholes option price (fair value per share). |
| `delta` | `number` | Option Delta ($\Delta$). For Calls: ranges from $0.0$ to $1.0$. For Puts: ranges from $-1.0$ to $0.0$. |
| `gamma` | `number` | Option Gamma ($\Gamma$). Acceleration of Delta per \$1 shift in stock price. Always positive for long options. |
| `theta` | `number` | Annualized Option Theta ($\Theta$). Represents the expected time decay of the option position over a 1-year horizon. Divide by 365 to obtain daily decay. |
| `vega` | `number` | Option Vega ($\nu$). Absolute change in option price for a 1.0 unit (100%) change in volatility (or $\nu / 100$ per 1% IV shift). |
| `type` | `string` | The contract type evaluated (`"call"` or `"put"`). |
| `intrinsicValue` | `number` | The immediate exercise value: $\max(0, S - K)$ for Call, $\max(0, K - S)$ for Put. |
| `timeValue` | `number` | Extrinsic time and volatility premium: $\text{price} - \text{intrinsicValue}$. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Standard At-the-Money (ATM) 3-Month Call Option

Evaluate an ATM European Call on a \$100 stock with 20% volatility and 5% risk-free rate expiring in 3 months ($T = 0.25$ years).

**Node Parameters:**
```json
{
  "stockPrice": 100,
  "strikePrice": 100,
  "timeToExpiry": 0.25,
  "riskFreeRate": 5,
  "volatility": 20,
  "type": "call"
}
```

**Output Payload:**
```json
{
  "success": true,
  "price": 4.614988613066949,
  "delta": 0.5694603016957197,
  "gamma": 0.03928800094473793,
  "theta": -10.474152266772839,
  "vega": 19.644000472368965,
  "type": "call",
  "intrinsicValue": 0,
  "timeValue": 4.614988613066949
}
```

> **Interpretation:** The fair value of the call option is \$4.61. Because the option is ATM ($S = K = 100$), intrinsic value is \$0.00 and all \$4.61 is time value. Delta is $+0.57$, indicating that for every \$1 increase in stock price, the option increases by $\approx \$0.57$.

---

### Example 2: At-the-Money (ATM) European Put (Put-Call Parity)

Evaluate the corresponding Put option using the same underlying parameters.

**Node Parameters:**
```json
{
  "stockPrice": 100,
  "strikePrice": 100,
  "timeToExpiry": 0.25,
  "riskFreeRate": 5,
  "volatility": 20,
  "type": "put"
}
```

**Output Payload:**
```json
{
  "success": true,
  "price": 3.372768662455094,
  "delta": -0.4305396983042803,
  "gamma": 0.03928800094473793,
  "theta": -5.536263264303431,
  "vega": 19.644000472368965,
  "type": "put",
  "intrinsicValue": 0,
  "timeValue": 3.372768662455094
}
```

> **Interpretation:** Notice that $\text{Gamma} = 0.0393$ and $\text{Vega} = 19.644$ are identical to the Call. Furthermore, $\Delta_{\text{call}} - \Delta_{\text{put}} = 0.5695 - (-0.4305) = 1.0$, exactly satisfying Put-Call parity.

---

### Example 3: In-the-Money (ITM) Call with High Intrinsic Value

Stock has rallied to \$120 while holding a \$100 strike call with 6 months to expiration ($T = 0.5$).

**Node Parameters:**
```json
{
  "stockPrice": 120,
  "strikePrice": 100,
  "timeToExpiry": 0.5,
  "riskFreeRate": 4,
  "volatility": 25,
  "type": "call"
}
```

**Output Payload:**
```json
{
  "success": true,
  "price": 24.129321481141384,
  "delta": 0.8522731853478954,
  "gamma": 0.01258671842013149,
  "theta": -9.184307684078877,
  "vega": 18.124874524989345,
  "type": "call",
  "intrinsicValue": 20,
  "timeValue": 4.129321481141384
}
```

> **Interpretation:** The option is deeply In-The-Money with an intrinsic value of \$20.00 ($120 - 100$) and \$4.13 of time value. Delta is $+0.85$, demonstrating high sensitivity to the stock's movements.

---

### Example 4: Out-of-the-Money (OTM) Put Option

Evaluating a downside protective put with strike \$85 on a \$100 stock.

**Node Parameters:**
```json
{
  "stockPrice": 100,
  "strikePrice": 85,
  "timeToExpiry": 0.25,
  "riskFreeRate": 5,
  "volatility": 20,
  "type": "put"
}
```

**Output Payload:**
```json
{
  "success": true,
  "price": 0.2073948754127816,
  "delta": -0.0528472483811802,
  "gamma": 0.01048624817420489,
  "theta": -1.458921847182945,
  "vega": 5.243124087102445,
  "type": "put",
  "intrinsicValue": 0,
  "timeValue": 0.2073948754127816
}
```

> **Interpretation:** Because the strike is far below the current market price, the option has a low premium (\$0.21) and a small negative Delta ($-0.05$).

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Calculate option prices using Black-Scholes model with Greeks (delta, gamma, theta, vega)
```

### How the Workflow Operates

1. **Manual Trigger (`manual-trigger`):** Initiates execution when triggered from the canvas or an API call.
2. **Options Pricing (`options-pricing`):** Receives input parameters, runs the Black-Scholes equations, and computes fair value with Delta, Gamma, Theta, and Vega.
3. **Log (`log`):** Logs the calculated pricing object and Greeks directly to the execution inspection console.

```
┌──────────────────┐       ┌───────────────────┐       ┌─────────────┐
│  Manual Trigger  │ ────> │  Options Pricing  │ ────> │     Log     │
│ (manual-trigger) │       │ (options-pricing) │       │    (log)    │
└──────────────────┘       └───────────────────┘       └─────────────┘
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Questions & Issues

#### Result returns `NaN` for Price or Greeks
- **Cause:** `timeToExpiry` or `volatility` was set to `0`. Division by zero occurs in $d_1 = \dots / (\sigma \sqrt{T})$.
- **Solution:** Ensure `timeToExpiry` is strictly greater than `0` (e.g., `0.0027` for 1 day remaining) and `volatility` is strictly positive (e.g., `1` for 1%).

#### Option Price looks unrealistically large or small
- **Cause:** Inputting volatility or interest rates as decimals (e.g., entering `0.20` instead of `20` for 20% volatility).
- **Solution:** Enter values as natural percentages: `20` for 20% volatility, and `5` for 5% risk-free rate.

#### How to convert annualized Theta to daily decay?
- **Cause:** Standard Black-Scholes produces annualized Theta ($\Theta_{\text{annual}}$).
- **Solution:** To calculate daily dollar decay, divide `theta` by 365:
  $$\Theta_{\text{daily}} = \frac{\text{theta}}{365}$$
  For example, with $\text{theta} = -10.47$, the daily time decay is $-10.47 / 365 \approx -\$0.0287$ per day.

#### Put-Call Parity appears off
- **Check:** Ensure the continuous compounding formula is used:
  $$C - P = S - K \cdot e^{-r \cdot T}$$
  Where $r = \text{riskFreeRate} / 100$ and $T = \text{timeToExpiry}$.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Bond Pricing](../bond-pricing/en.md) — Calculate bond yields, dirty prices, and duration
- [Loan Calculator](../loan-calculator/en.md) — Amortization schedules and payment calculations
- [Financial Ratio Analyzer](../financial-ratio-analyzer/en.md) — Financial liquidity, leverage, and profitability ratios
- [Function](../function/en.md) — Run custom multi-leg spread logic, iron condors, and portfolio aggregations
- [Log](../log/en.md) — Inspect output payloads and Greeks in the execution console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-14 | Initial documentation release with complete Black-Scholes formulas, analytical Greeks (Delta, Gamma, Theta, Vega), value decomposition, and example workflow |

<!-- /SECTION: changelog -->

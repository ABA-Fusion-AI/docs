---
  node_id: "bourrage-raw-sample"
  title: "Bourrage Raw Sample"
  description: "Send one timestamped row of raw SAG mill signals to the bourrage prediction server as JSON, keyed by the model's own column names."
  category: "mathematical-statistical-analysis"
  subcategory: "calculators-models"
  version: "0.1.0"
  language: "en"
  last_updated: "2026-08-12"
  author: "Fusion Team"
  tags:
    - bourrage
    - sag-mill
    - mining
    - prediction
    - industrial
    - telemetry
  related_nodes:
    - opc
    - scada-read
    - interval
    - http-request
---

<!-- SECTION: overview -->
  # Bourrage Raw Sample

  > **Category:** Mathematical & Statistical Analysis&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

  Sends one timestamped row of **raw** SAG mill measurements to the bourrage
  (mill-clogging) prediction server as JSON.

The node carries the upstream signals only — the 17 process tags plus the three
HG/MG/LG blend proportions, with a `Horodates` stamp. It does **not** compute the
155 model inputs. Those (5-minute statistics, retards and differences, 30-minute
rolling trends, and the causal covariance/correlation pairs) need a rolling
history that a single workflow tick does not have, so they are built server-side
where the model and its accumulated window live.

### Use Cases

- Poll a SAG mill every minute over OPC/SCADA and stream each sample to the risk model
- Raise an operator alarm when the returned risk level crosses a threshold
- Replay historical rows through the same endpoint used in production
- Fan one sample out to both the model and a historian

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Server

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `url` | `string` | ✅ Yes | — | Endpoint receiving the sample. Accepts an expression. |
| `authType` | `enum` | ❌ No | `none` | `none`, `bearer`, or `basic`. Builds the `Authorization` header for you. |
| `bearerToken` | `string` | ✅ Yes* | — | Sent as `Authorization: Bearer <token>`. Shown only when `authType` is `bearer`. |
| `username` | `string` | ✅ Yes* | — | HTTP basic username. Shown only when `authType` is `basic`. |
| `password` | `string` | ✅ Yes* | — | HTTP basic password. Shown only when `authType` is `basic`. |
| `headers` | `record` | ❌ No | — | Extra headers. `Content-Type` defaults to `application/json`; your value wins. |
| `timeoutMs` | `number` | ❌ No | `10000` | Abort after this many milliseconds. `0` disables the timeout. |
| `ignoreHttpErrors` | `boolean` | ❌ No | `false` | When on, every status resolves to `success` so the graph can branch on `status`. |

\* Required only while the matching `authType` is selected; the field is hidden and unchecked otherwise.

### Sample Timestamp

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `timestampMode` | `enum` | ❌ No | `now` | `now` stamps with the current UTC clock as `YYYY-MM-DD HH:MM:SS`. `custom` sends a value you supply. |
| `horodates` | `string` | ✅ Yes* | — | Sample time, sent verbatim as `Horodates`. Shown only when `timestampMode` is `custom`. |

### Raw Signals

All 20 are numbers and all accept expressions, so each can be bound to an
upstream tag. The **Sent as** column is the exact key written to the JSON body.

| Parameter | Sent as | Description |
|-----------|---------|-------------|
| `speed` | `SPEED (rpm)` | Mill rotation speed |
| `weightSag` | `Weight SAG (t)` | Mill charge weight |
| `unmilledMaterialFlowRate` | `Unmilled Material Flow Rate (t/h)` | Unmilled material returning to the mill |
| `feedFlowRate` | `Feed Flow Rate (t/h)` | Fresh feed into the mill |
| `motorPowerSagMillLeft` | `Motor Power sag mill left(kw)` | Left mill motor active power |
| `motorPowerSagMillRight` | `Motor Power sag mill right(kw)` | Right mill motor active power |
| `sagMillPower` | `SAG Mill Power(kw)` | Total mill power |
| `frequenceMoteurRightSag` | `Frequence moteur right SAG(hertz)` | Right motor drive frequency |
| `frequenceMoteurLeftSag` | `Frequence moteur left SAG(hertz)` | Left motor drive frequency |
| `sagMillWater` | `Sag Mill water (m3/h)` | Water addition |
| `puissanceActiveJawCrusherMotor` | `Puissance Active jaw crusher motor(kw)` | Jaw crusher motor active power |
| `puissanceActiveSagMillScatConv` | `Puissance Active sag mill scat conv(kw)` | Scats conveyor active power |
| `crushedOreTonnageFlowRate` | `Crushed Ore Tonnage Flow Rate (t/h)` | Crushed ore tonnage |
| `torqueJawCrusherMotor` | `Torque jaw crusher motor (N.m)` | Jaw crusher motor torque |
| `aprf1` | `APRF1` | Process signal APRF1 |
| `aprf2` | `APRF2` | Process signal APRF2 |
| `aprf3` | `APRF3` | Process signal APRF3 |
| `natureTvHgPct` | `Nature_TV_HG_pct` | High-grade share of the blend |
| `natureTvMgPct` | `Nature_TV_MG_pct` | Medium-grade share of the blend |
| `natureTvLgPct` | `Nature_TV_LG_pct` | Low-grade share of the blend |

> Schema field names are camelCase because they must be valid identifiers. The
> node maps them back to the model's original column names — spacing and casing
> included — before sending.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Triggers one request. Bind parameters to upstream values with expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The server's reply, plus the request that produced it |
| `error` | `Error` | Transport failure, timeout, or a non-2xx status when `ignoreHttpErrors` is off |

### Output Schema (`success`)

| Field | Type | Description |
|-------|------|-------------|
| `status` | `number` | HTTP status code |
| `statusText` | `string` | HTTP status text |
| `sentAt` | `string` | The `Horodates` value actually sent |
| `request` | `object` | The exact JSON body sent, keyed by model column names |
| `body` | `any` | Parsed response from the server |

### Request Body

```json
{
  "Horodates": "2026-08-12 10:01:00",
  "SPEED (rpm)": 8.1,
  "Weight SAG (t)": 320,
  "Unmilled Material Flow Rate (t/h)": 45,
  "Feed Flow Rate (t/h)": 780,
  "Motor Power sag mill left(kw)": 4200,
  "Motor Power sag mill right(kw)": 4210,
  "SAG Mill Power(kw)": 8410,
  "Frequence moteur right SAG(hertz)": 48.5,
  "Frequence moteur left SAG(hertz)": 48.4,
  "Sag Mill water (m3/h)": 260,
  "Puissance Active jaw crusher motor(kw)": 150,
  "Puissance Active sag mill scat conv(kw)": 22,
  "Crushed Ore Tonnage Flow Rate (t/h)": 690,
  "Torque jaw crusher motor (N.m)": 980,
  "APRF1": 1.1,
  "APRF2": 2.2,
  "APRF3": 3.3,
  "Nature_TV_HG_pct": 0.35,
  "Nature_TV_MG_pct": 0.45,
  "Nature_TV_LG_pct": 0.2
}
```

This is exactly the dictionary the predictor's `update()` accepts, so the server
can hand it straight to the model without remapping.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Score a SAG mill sample every minute
```

**How it flows:**

1. An **Interval** trigger fires once a minute — the sampling rate the model expects.
2. **OPC** reads the mill tags for that minute.
3. **Bourrage Raw Sample** binds each signal to a tag from step 2 and POSTs the row.
4. **Log** receives the reply. Swap it for an **If/Else** to branch on the risk level and drive an alarm.

### Common Patterns

- **Continuous scoring:** interval → read tags → this node → branch on risk level
- **Backfill:** iterate historical rows with **Loop**, `timestampMode: custom`, one request each
- **Dual write:** fan the same sample out to the model and to a historian

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### The response has no prediction, but the status is 2xx

**Cause:** Expected. The model scores a five-minute window only once that window
closes, so early samples return an empty or null result.

**Solution:** Branch on the returned risk fields rather than treating an empty
reply as a failure. If the server signals "not ready" with a non-2xx status
instead, turn on `ignoreHttpErrors` and branch on `status`.

#### Predictions never appear, or the server rejects the history

**Cause:** Mixed timestamp conventions. A stamp carrying a UTC offset and a
naive one cannot be compared, so the server's history index breaks once both
have arrived.

**Solution:** Pick one convention for the whole run. `timestampMode: now` emits
the naive `YYYY-MM-DD HH:MM:SS` form; if you supply `Horodates` yourself, match
that shape.

#### Blend proportions look wrong or the risk jumps unexpectedly

**Cause:** HG/MG/LG sent as `0–1` for some samples and `0–100` for others. The
server rescales a batch it sees above `1.5`, so a mixed batch is rescaled
inconsistently. This node passes the values through untouched.

**Solution:** Normalize upstream so every sample uses the same scale.

#### `Unknown column` or the model ignores a signal

**Cause:** A proxy or gateway between the node and the model renamed or
lower-cased the JSON keys. The model matches the original column spelling exactly.

**Solution:** Compare the `request` field of the `success` output against what
the server received, and stop the intermediary from rewriting keys.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `ECONNREFUSED` | Server unreachable at `url` | Check the host, port, and that the model service is up |
| `ETIMEDOUT` | No reply within `timeoutMs` | Raise the timeout, or check whether the server is rebuilding its window |
| `401` / `403` | Missing or wrong credentials | Check `authType` and the bound token or username/password |
| `422` | Server rejected the payload shape | Confirm all 20 signals are bound and `Horodates` parses |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [OPC](./opc.md) – Read the mill tags that feed this node
- [Interval](./interval.md) – Drive the one-sample-per-minute cadence
- [If/Else](./if-else.md) – Branch on the returned risk level
- [HTTP Request](./http-request.md) – Generic alternative when you need a different payload shape

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 0.1.0 | 2026-08-12 | Initial release |

<!-- /SECTION: changelog -->

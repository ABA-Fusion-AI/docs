---
node_id: "delay"
title: "Delay"
description: "Delays the emission of items from the source by a given timeout or until a given Date."
category: "orchestration"
subcategory: "Timing & Rate Control"
version: "1.1.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - delay
  - timing
  - rate-control
  - orchestration
  - pause
  - throttle
related_nodes:
  - debounce-time
  - interval
  - function
  - http-request
---

<!-- SECTION: overview -->
# Delay

> **Category:** Orchestration &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Utility Node

Hold each incoming item for a fixed duration (in **milliseconds**) before forwarding it unchanged downstream. The **Delay** node does not alter the data payload in any way — it only shifts its timing. Every item is delayed by the same amount, preserving arrival order.

### Use Cases

- **API Rate Limiting:** Insert a pause between outbound HTTP requests to respect external API rate limits and avoid `429 Too Many Requests` errors.
- **Pipeline Pacing:** Stagger items in a processing pipeline to prevent overwhelming a slow consumer node or database.
- **Timed Triggers:** Wait a fixed period after a trigger fires before executing a follow-up action (e.g., send a reminder 5 seconds after user registration).
- **Retry Backoff:** Add a deliberate delay in an error branch before retrying a failed operation.
- **AI Pipeline Throttling:** Pace requests to LLM nodes (Mistral, Groq, OpenAI) to stay within per-minute token limits.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `duration` | `number` | Yes | `5000` | The delay in **milliseconds** before the item is forwarded. Must be a non-negative integer. |

### Duration Reference Table

| Duration value | Equivalent time |
|----------------|-----------------|
| `500` | 0.5 seconds |
| `1000` | 1 second |
| `5000` | 5 seconds *(default)* |
| `10000` | 10 seconds |
| `60000` | 1 minute |
| `0` | No delay (pass-through) |

> **Note:** The `duration` parameter accepts expressions, so you can dynamically compute the delay from upstream data — for example: `{{ outputs.Function.success.waitTime * 1000 }}`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | The item to delay. The node accepts any data type — objects, strings, arrays, or primitives. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | The original item, emitted **unchanged** after the configured `duration` has elapsed. |
| `error` | `Error` | Emitted if the input stream errors or if `duration` is invalid. |

### Passthrough Behavior

The Delay node is transparent to the data — it forwards the **exact same payload** it receives. No fields are added, removed, or modified. The only change is the timing of emission.

```json
// Input received
{ "userId": "abc123", "action": "signup" }

// Output after 5000ms — identical payload
{ "userId": "abc123", "action": "signup" }
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: AI Interview Question Generator with Throttled LLM
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **Function Node:** Returns a candidate profile string (e.g., `"devloppeur java"`).
3. **Delay Node:** Holds the data for **5000 ms** (5 seconds) before passing it forward — preventing the Agent from being invoked too quickly and giving upstream processes time to settle.
4. **Agent Node:** Receives the profile and uses the Groq LLM to generate 5 targeted technical interview questions based on the candidate's experience.
5. **Log Node:** Displays the generated interview questions.

### Common Patterns

- **Before an LLM Call:** Place a Delay node before Agent or AI Chat nodes when running batch pipelines to respect token-per-minute limits.
- **Retry Loops:** In an error branch, add a Delay before reconnecting to the retry input to implement exponential or fixed backoff.
- **Sequence Pacing:** Chain multiple action nodes with Delay nodes between them to create timed sequences (e.g., onboarding email series).
- **Rate-Limited APIs:** Add a 1000ms Delay before each HTTP Request node when looping over a list of items to avoid hitting API quotas.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Items appear later than expected
- **Cause:** The `duration` parameter is in **milliseconds**, not seconds.
- **Solution:** Multiply the desired seconds by 1000. For example, a 5-second delay requires `duration: 5000`, not `duration: 5`.

#### `Delay must be a non-negative integer`
- **Cause:** The `duration` value is missing, negative, a float, or resolves to `NaN` when using an expression.
- **Solution:** Ensure `duration` is a whole number greater than or equal to `0`. If using an expression, add a guard: `{{ Math.max(0, Math.round(myValue)) }}`.

#### Workflow appears frozen / no output
- **Cause:** A very large `duration` value (e.g., `9999999`) is causing the node to hold items indefinitely.
- **Solution:** Review the `duration` value. Use `0` to temporarily disable the delay and test the rest of the pipeline.

#### Delay not working when `duration` is an expression
- **Cause:** The expression references an output that hasn't been set yet at the time the Delay node is configured.
- **Solution:** Ensure the upstream node that provides the delay value is connected and runs before the Delay node in the workflow graph.

### Error Codes Summary

| Error | Cause | Solution |
|-------|-------|----------|
| Invalid `duration` | Negative, float, or undefined value | Use a non-negative integer in milliseconds |
| Node emits nothing | Duration too large or upstream disconnected | Check `duration` value and input connection |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Debounce Time](./debounce-time.md) – Emit only after a quiet period with no new items
- [Interval](./interval.md) – Emit items on a recurring timer
- [Function](./function.md) – Dynamically compute the delay duration based on workflow data
- [HTTP Request](./http-request.md) – Pair with Delay to pace outbound API calls

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.1.0 | 2026-08-06 | Expanded documentation with workflow example, common patterns, and dynamic expression support |
| 1.0.0 | 2026-06-30 | Initial release |

<!-- /SECTION: changelog -->

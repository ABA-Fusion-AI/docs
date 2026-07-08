---
  node_id: "stg-once"
  title: "STG Once"
  description: "A staging copy of the Once trigger node for testing workflow behavior in the STG environment."
  category: "utilities"
  subcategory: "stg"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-07-08"
  author: "Fusion Team"
  tags:
    - stg
    - once
    - trigger
    - testing
    - staging
---

# STG Once

> **Category:** Utilities&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

This node is a staging version of the built-in Once trigger. It emits once and then stops, which makes it useful for validating one-time workflow execution behavior in the STG environment.

## Configuration

This node has no configurable parameters.

## Behavior

- Emits once after a short delay
- Stops further emissions after the first run
- Does not require additional setup

## Example

Use this node at the start of a workflow when you want to test that a single execution path runs exactly once.

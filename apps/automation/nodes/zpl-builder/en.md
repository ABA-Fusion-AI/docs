---
node_id: "zpl-builder"
title: "ZPL Builder"
description: "Generates ZPL (Zebra Programming Language) commands for label printing"
category: "Logistics & Supply Chain"
version: "1.0.0"
language: "en"
last_updated: "2026-09-02"
author: "Fusion Team"
tags:
  - zpl
  - zebra
  - label
  - printing
  - barcode
  - logistics
related_nodes:
  - esc-pos-builder
  - barcode-generator
  - log
---

<!-- SECTION: header -->

# ZPL Builder

> **Category:** Logistics & Supply Chain | **Type:** Action Node

Generate ZPL (Zebra Programming Language) commands for Zebra-compatible label printing using configurable printer settings and label elements.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **ZPL Builder** node generates ZPL commands for Zebra-compatible label printing. It builds a complete ZPL label from printer configuration and an ordered list of label elements.

The node supports text, Code 128 barcodes, boxes, and reverse text. Each generated label starts with the ZPL start-format command, applies the configured printer settings, adds the configured elements in order, and ends with the ZPL end-format command.

### Key Features

- **ZPL Generation:** Generate complete Zebra Programming Language output
- **Printer Configuration:** Configure darkness, label width, and print speed
- **Text Elements:** Add positioned text with configurable font size
- **Barcode Elements:** Generate Code 128 barcodes with configurable height
- **Barcode Text:** Show or hide human-readable text below a barcode
- **Box Elements:** Draw boxes with configurable dimensions and thickness
- **Reverse Text:** Generate reversed text over a black background
- **Multiple Elements:** Add several label elements in a defined order
- **Output Information:** Return both the generated ZPL and its character length

### Supported Elements

The node supports four element types:

1. `text`
2. `barcode`
3. `box`
4. `reverseText`

Elements are processed in the same order in which they appear in the `elements` array.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Printer Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `darkness` | `number` | ❌ No | `15` | Print darkness level used by the `~SD` command |
| `width` | `number` | ❌ No | `800` | Label width in dots used by the `^PW` command |
| `speed` | `number` | ❌ No | `3` | Print speed used by the `^PR` command |

With the default configuration, the generated label begins with:

```text
^XA
~SD15
^PW800
^PR3
```

and ends with:

```text
^XZ
```

### Zero Values

The builder applies its defaults using fallback logic.

When `darkness`, `width`, or `speed` is configured as `0`, the value is replaced by its corresponding default:

```text
darkness: 0 → 15
width: 0 → 800
speed: 0 → 3
```

This behavior was confirmed during functional testing.

### Elements

The `elements` parameter is an array of label elements.

Each element requires:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | `string` | ✅ Yes | Element type: `text`, `barcode`, `box`, or `reverseText` |
| `x` | `number` | ✅ Yes | Horizontal position in dots |
| `y` | `number` | ✅ Yes | Vertical position in dots |

Additional parameters depend on the selected element type.

### Text Element

A `text` element adds positioned text to the label.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `text` | `string` | Conditional | — | Text content |
| `fontSize` | `number` | ❌ No | `30` | Font height and width |

Example:

```text
Type: text
X: 50
Y: 100
Text: Hello Zebra
Font Size: 30
```

Generated command:

```text
^FO50,100^A0N,30,30^FDHello Zebra^FS
```

If `text` is not provided, the element is skipped by the builder.

### Barcode Element

A `barcode` element generates a Code 128 barcode.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | Conditional | — | Barcode data |
| `height` | `number` | ❌ No | `100` | Barcode height in dots |
| `showText` | `boolean` | ❌ No | `true` | Show human-readable text below the barcode |

Example:

```text
Type: barcode
X: 50
Y: 100
Data: 123456789
Height: 100
Show Text: true
```

Generated command:

```text
^BY2,3,100^FO50,100^BCN,100,Y,N,N^FD123456789^FS
```

When `showText` is `false`, the barcode command uses:

```text
^BCN,100,N,N,N
```

If `data` is not provided, the barcode element is skipped.

### Box Element

A `box` element draws a rectangular box.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `boxWidth` | `number` | Conditional | — | Box width in dots |
| `boxHeight` | `number` | Conditional | — | Box height in dots |
| `thickness` | `number` | ❌ No | `2` | Border thickness |

Example:

```text
Type: box
X: 50
Y: 100
Box Width: 300
Box Height: 150
Thickness: 4
```

Generated command:

```text
^FO50,100^GB300,150,4^FS
```

Both `boxWidth` and `boxHeight` must be available for the builder to add the box.

### Reverse Text Element

A `reverseText` element creates a black box and renders reversed text over it.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `text` | `string` | Conditional | Text displayed inside the reversed area |
| `reverseWidth` | `number` | Conditional | Width of the reverse text area |
| `reverseHeight` | `number` | Conditional | Height of the reverse text area |

Example:

```text
Type: reverseText
X: 50
Y: 100
Text: IMPORTANT
Reverse Width: 300
Reverse Height: 50
```

Generated commands:

```text
^FO50,100^GB300,50,50^FS
^FO50,100^A0N,40,40^FR^FDIMPORTANT^FS
```

The font dimensions are calculated from:

```text
reverseHeight × 0.8
```

For `reverseHeight: 50`, the calculated font size is `40`.

If `text`, `reverseWidth`, or `reverseHeight` is missing, the reverse text element is skipped.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

The node does not use incoming workflow data to build the label. Label generation is controlled by the node configuration and the `elements` array.

### Outputs

| Field | Type | Description |
|-------|------|-------------|
| `zpl` | `string` | Complete generated ZPL label |
| `length` | `number` | Character length of the generated ZPL string |

Example output structure:

```json
{
  "zpl": "^XA\n~SD15\n^PW800\n^PR3\n^XZ",
  "length": 25
}
```

### Minimal Output

With default printer settings and no elements:

```text
^XA
~SD15
^PW800
^PR3
^XZ
```

The returned length is:

```text
25
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Basic Label

Configuration:

```text
Darkness: 15
Width: 800
Speed: 3
```

With no elements, the node generates:

```text
^XA
~SD15
^PW800
^PR3
^XZ
```

---

### Custom Printer Configuration

Configuration:

```text
Darkness: 20
Width: 600
Speed: 5
```

Output:

```text
^XA
~SD20
^PW600
^PR5
^XZ
```

---

### Text Label

Element:

```text
Type: text
X: 50
Y: 100
Text: Hello Zebra
Font Size: 30
```

Generated ZPL:

```text
^XA
~SD15
^PW800
^PR3
^FO50,100^A0N,30,30^FDHello Zebra^FS
^XZ
```

---

### Barcode with Visible Text

Element:

```text
Type: barcode
X: 50
Y: 100
Data: 123456789
Height: 100
Show Text: true
```

Generated barcode command:

```text
^BY2,3,100^FO50,100^BCN,100,Y,N,N^FD123456789^FS
```

---

### Barcode without Visible Text

Using the same barcode with:

```text
Show Text: false
```

generates:

```text
^BY2,3,100^FO50,100^BCN,100,N,N,N^FD123456789^FS
```

---

### Box

Element:

```text
Type: box
X: 50
Y: 100
Box Width: 300
Box Height: 150
Thickness: 4
```

Generated command:

```text
^FO50,100^GB300,150,4^FS
```

---

### Reverse Text

Element:

```text
Type: reverseText
X: 50
Y: 100
Text: IMPORTANT
Reverse Width: 300
Reverse Height: 50
```

Generated commands:

```text
^FO50,100^GB300,50,50^FS
^FO50,100^A0N,40,40^FR^FDIMPORTANT^FS
```

---

### Product Label with Multiple Elements

The example workflow uses three elements in this order:

```text
1. Text
2. Barcode
3. Box
```

Configuration:

```text
Darkness: 15
Width: 800
Speed: 3
```

Text:

```text
Type: text
X: 50
Y: 40
Text: Product A
Font Size: 30
```

Barcode:

```text
Type: barcode
X: 50
Y: 100
Data: 123456789
Height: 80
Show Text: true
```

Box:

```text
Type: box
X: 20
Y: 20
Box Width: 500
Box Height: 250
Thickness: 3
```

Generated ZPL:

```text
^XA
~SD15
^PW800
^PR3
^FO50,40^A0N,30,30^FDProduct A^FS
^BY2,3,80^FO50,100^BCN,80,Y,N,N^FD123456789^FS
^FO20,20^GB500,250,3^FS
^XZ
```

Elements are generated in the same order in which they are configured.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate a ZPL product label with text, barcode, and box elements
```

### Workflow Structure

```text
Manual Trigger → ZPL Builder → Log
```

The example generates a product label containing:

- product text
- a Code 128 barcode
- a surrounding box

The Log node can be used to inspect the generated `zpl` value and its `length`.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### An element does not appear in the generated ZPL

**Cause:** One or more values required by the selected element type are missing.

The builder skips:

- `text` without `text`
- `barcode` without `data`
- `box` without `boxWidth` or `boxHeight`
- `reverseText` without `text`, `reverseWidth`, or `reverseHeight`

**Solution:** Verify that all values required by the selected element type are configured.

---

#### Zero printer values use the defaults

**Cause:** The builder uses fallback values for printer configuration.

For example:

```text
Darkness: 0
Width: 0
Speed: 0
```

produces:

```text
~SD15
^PW800
^PR3
```

**Solution:** Use non-zero values when configuring these printer settings.

---

#### Barcode text is still displayed

**Cause:** `showText` defaults to `true`.

**Solution:** Explicitly configure:

```text
Show Text: false
```

The generated barcode command will use `N` for the text flag.

---

#### Reverse text size is different from the configured height

**Cause:** Reverse text font dimensions are calculated automatically from `reverseHeight`.

The calculation is:

```text
reverseHeight × 0.8
```

For example:

```text
Reverse Height: 50
```

produces a font size of:

```text
40
```

---

#### Elements appear in an unexpected order

**Cause:** Elements are processed sequentially according to their order in the `elements` array.

**Solution:** Reorder the elements in the node configuration to control the generated ZPL command order.

### Behavior Reference

| Behavior | Cause | Solution |
|----------|-------|----------|
| Element missing from output | Required element-specific value is missing | Complete the element configuration |
| Printer value `0` becomes default | Builder fallback logic | Use a non-zero value |
| Barcode text visible | `showText` defaults to `true` | Set `showText` to `false` |
| Reverse text font size differs | Size is calculated from `reverseHeight × 0.8` | Adjust `reverseHeight` |
| Unexpected element order | Elements follow array order | Reorder the configured elements |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- **ESC/POS Builder** - Generate commands for ESC/POS-compatible printing
- **Barcode Generator** - Generate barcode data for workflows
- **Log** - Inspect generated ZPL output

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-02 | Initial documentation and validated workflow example |

<!-- /SECTION: changelog -->
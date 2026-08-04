---
node_id: "icalendar"
title: "iCalendar"
description: "Generate, extend, and parse iCalendar content and individual events."
category: "data-transformation"
subcategory: "date-time"
version: "1.0.0"
language: "en"
last_updated: "2026-08-04"
author: "Fusion Team"
tags: [icalendar, ics, calendar, events]
related_nodes: [google-calendar, outlook-calendar]
---

<!-- SECTION: overview -->
# iCalendar

> **Category:** Data Transformation&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Generate complete `.ics` calendars or individual events, append events to existing calendar content, and parse calendar events into workflow data.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `operation` | enum | No | `generateCalendar`, `addEvent`, `parseCalendar`, or `generateEvent`. |
| `calendarName` | string | No | Generated calendar name. |
| `timezone` | string | No | IANA timezone for the calendar. |
| `events` | string | Conditional | JSON array of events for calendar generation. |
| `icsContent` | string | Conditional | Existing ICS content to parse or extend. |
| `title`, `start`, `end` | string | Conditional | Event title and ISO datetimes. |
| `description`, `location` | string | No | Optional event details. |
| `organizerName`, `organizerEmail` | string | No | Optional organizer details. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Success:** Generated `icsContent`, or a normalized event list and count when parsing.
- **Error:** Invalid JSON, missing required event fields, or malformed calendar content.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate an iCalendar event
```
<!-- /SECTION: examples -->

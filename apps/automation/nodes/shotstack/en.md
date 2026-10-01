---
node_id: "shotstack"
title: "Shotstack: Video Editing"
description: "Programmatically generate, edit, and render automated videos, animations, and image sequences via the Shotstack cloud video editing API."
category: "Generative AI & LLMs"
subcategory: "LLM Providers"
version: "1.0.0"
language: "en"
last_updated: "2026-10-01"
author: "Fusion Team"
tags:
  - shotstack
  - video-editing
  - video-rendering
  - generative-ai
  - multimedia
  - automated-video
  - cloud-rendering
related_nodes:
  - manual-trigger
  - log
  - function
  - did
  - synthesia
  - hey-gen
---

<!-- SECTION: overview -->
# Shotstack: Video Editing

> **Category:** Generative AI & LLMs&nbsp;&nbsp;|&nbsp;&nbsp;**Subgroup:** LLM Providers&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **Shotstack: Video Editing** node connects Fusion workflows directly to the [Shotstack Video Editing API](https://shotstack.io/docs/guide/). It allows you to programmatically assemble, composite, and render high-definition videos, animated banners, motion graphics, and audio compositions in the cloud without needing local desktop video editing software.

Using a JSON-driven multi-track timeline model, you can arrange video clips, static images, dynamic text titles, HTML/CSS snippets, and soundtrack audio across layered tracks. Once configured, the node sends your composition to Shotstack's distributed rendering cloud and returns the queued render job details.

### Key Features

- **Multi-Track Timeline Compositing:** Layer visual and audio assets across multiple tracks with precise control over foreground/background stacking, positioning, scaling, and start/duration timings.
- **Diverse Asset Support:** Render video files (`mp4`), images (`jpg`, `png`), audio clips (`mp3`), pre-styled titles, custom HTML/CSS cards, luma mattes, and image sequences.
- **Built-in Motion & Transitions:** Apply visual transitions (such as `fade`, `wipeLeft`, `wipeRight`, `slideUp`) and dynamic motion animations (`zoomIn`, `slideLeft`) to any clip.
- **Flexible Aspect Ratios & Formats:** Export directly to modern social formats including vertical 9:16 (TikTok, Instagram Reels, YouTube Shorts), square 1:1, landscape 16:9, or custom resolutions up to 4K.
- **Soundtrack Integration:** Combine your video tracks with background audio, featuring volume controls and automatic fade-in/fade-out effects.
- **Sandbox & Production Environments:** Safely prototype and test your compositions using the free Shotstack Sandbox environment before switching to Production for finalized high-throughput renders.

### Use Cases

- **Automated Social Media Video Generation:** Automatically generate TikTok, Reels, or YouTube Shorts videos from incoming RSS feeds, blog posts, or AI-generated scripts.
- **Personalized E-Commerce Video Ads:** Dynamically combine product photos, promotional pricing text overlays, and upbeat background music into tailored marketing ads.
- **Automated Real Estate Showcases:** Assemble photo galleries of new property listings with animated captions, agent branding, and transitions into a finished video tour.
- **Dynamic Watermarking & Branding:** Overlay corporate logos, animated lower-thirds, or subtitle captions onto existing video assets.
- **Animated GIF & Social Banner Creation:** Transform static announcements and marketing banners into looping animated GIFs or short teaser clips.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Configure your Shotstack credentials, rendering environment, timeline structure, and output specifications in the node parameters panel.

### General Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | ✅ Yes | — | Your Shotstack API Key. Available from the [Shotstack Dashboard](https://dashboard.shotstack.io/). Supports workflow expressions. |
| `environment` | `enum` | ❌ No | `sandbox` | Rendering environment:<br>• `sandbox`: Routes to `https://api.shotstack.io/edit/stage/render` for free prototyping and testing.<br>• `production`: Routes to `https://api.shotstack.io/edit/v1/render` for final production exports. |
| `timeline` | `object` | ✅ Yes | — | The root timeline object specifying background styling, custom fonts, audio soundtracks, and visual tracks. |
| `output` | `object` | ❌ No | — | Optional configuration defining the export format, resolution, aspect ratio, frame rate, and compression quality. |

---

### Timeline Configuration (`timeline`)

The `timeline` object represents your editing canvas and sequence:

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `timeline.background` | `string` | ❌ No | `#000000` | Hexadecimal background color for the canvas (e.g. `#FFFFFF`, `#000000`, `#1E1E2F`). |
| `timeline.fonts` | `array` | ❌ No | `[]` | List of custom web fonts to download and register for text rendering. Each item is an object: `{ "src": "https://example.com/font.ttf" }`. |
| `timeline.soundtrack` | `object` | ❌ No | — | Global audio soundtrack played across the entire video duration (see Soundtrack Options below). |
| `timeline.tracks` | `array` | ✅ Yes | — | Array of track objects. Tracks are rendered from top to bottom (the first track is on top of subsequent tracks). |

#### Soundtrack Options (`timeline.soundtrack`)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `src` | `string` | ✅ Yes | Publicly accessible URL to an audio file (`.mp3` or `.wav`). |
| `effect` | `string` | ❌ No | Audio transition effect: `fadeIn`, `fadeOut`, or `fadeInFadeOut`. |
| `volume` | `number` | ❌ No | Audio volume multiplier between `0.0` (muted) and `1.0` (100% volume). Defaults to `1.0`. |

---

### Tracks & Clips Configuration (`timeline.tracks[]`)

Each entry in `timeline.tracks` contains a `clips` array. Clips define when, where, and how media assets appear on the timeline.

> [!NOTE]
> **Track Stacking Order:** Track 1 (index 0) is the foreground layer. Any clips placed in Track 1 will render on top of clips placed in Track 2, 3, etc. Use upper tracks for watermarks, lower-thirds, and text overlays, and lower tracks for background videos or images.

#### Clip Object Properties

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `asset` | `object` | ✅ Yes | — | The media asset to display (see Asset Types below). |
| `start` | `number` | ❌ No | `0` | The start time of the clip along the timeline, measured in seconds (e.g. `0`, `2.5`). |
| `length` | `number` | ❌ No | — | The duration of the clip in seconds (e.g. `5`). |
| `fit` | `enum` | ❌ No | `cover` | How the asset scales to fit the viewport: `cover`, `contain`, `crop`, or `none`. |
| `scale` | `number` | ❌ No | `1.0` | Scaling multiplier (e.g. `1.0` for 100%, `0.5` for 50%, `1.2` for 120%). |
| `position` | `string` | ❌ No | `center` | Anchor placement on screen: `center`, `top`, `bottom`, `left`, `right`, `topLeft`, `topRight`, `bottomLeft`, `bottomRight`. |
| `offset` | `object` | ❌ No | `{ x: 0, y: 0 }` | Fine-grained position adjustment with horizontal (`x`) and vertical (`y`) offsets between `-1.0` and `1.0`. |
| `transition` | `object` | ❌ No | — | In/out visual transitions: `{ "in": "fade", "out": "fade" }`. Supports `fade`, `wipeLeft`, `wipeRight`, `slideUp`, `slideDown`, `carouselLeft`, `carouselRight`. |
| `width` | `number` | ❌ No | — | Explicit display width in pixels. |
| `height` | `number` | ❌ No | — | Explicit display height in pixels. |

---

### Asset Types (`asset`)

The `asset.type` parameter determines which properties are applicable:

| Asset Type | Primary Purpose | Required Fields | Optional Fields |
|------------|-----------------|-----------------|-----------------|
| `video` | Video clip playback | `src` (URL to `.mp4` / `.mov`) | `style`, `animation` |
| `image` | Static image overlay or slide | `src` (URL to `.png` / `.jpg`) | `style`, `animation` |
| `audio` | Individual voiceover or sound effect clip | `src` (URL to `.mp3` / `.wav`) | — |
| `title` | Animated or styled text header / caption | `text` | `font`, `style`, `animation` |
| `rich-text` | Multi-line formatted typography | `text` | `font`, `style` |
| `html` | Custom HTML layout rendered via headless browser | `html` | `css`, `width`, `height` |
| `luma` | Luma matte transition video | `src` | — |
| `imageSequence` | Stop-motion or animated sequence of frames | `src` or `url` | — |

#### Font Styling Options (`asset.font`)

When using `title` or `rich-text` assets, configure typography using the `font` object:

| Field | Type | Description |
|-------|------|-------------|
| `family` | `string` | Font family name (e.g. `"Montserrat"`, `"Open Sans"`, `"Roboto"`). |
| `size` | `number` | Font size in points (e.g. `36`, `48`). |
| `weight` | `number` | Font weight (e.g. `300`, `400`, `700`, `900`). |
| `color` | `string` | Hexadecimal font color code (e.g. `#FFFFFF`, `#FFD700`). |
| `opacity` | `number` | Text opacity from `0.0` (invisible) to `1.0` (fully opaque). |

#### Animation Options (`asset.animation`)

Add motion graphics to images, video clips, or titles:

| Field | Type | Description |
|-------|------|-------------|
| `preset` | `string` | Motion preset: `zoomIn`, `zoomOut`, `slideLeft`, `slideRight`, `slideUp`, `slideDown`. |
| `style` | `string` | Motion easing curve (e.g. `linear`, `easeIn`, `easeOut`). |
| `direction` | `string` | Direction modifier: `left`, `right`, `up`, `down`. |
| `duration` | `number` | Duration of the animation in seconds. |

---

### Output Settings (`output`)

Customize the generated video file format, resolution, aspect ratio, and encoding parameters:

| Parameter | Type | Default | Allowed Values | Description |
|-----------|------|---------|----------------|-------------|
| `output.format` | `enum` | `mp4` | `mp4`, `gif`, `mp3`, `jpg`, `png`, `bmp` | Target file format. Use `mp4` for video, `gif` for animations, or `jpg`/`png` for thumbnail snapshots. |
| `output.resolution` | `enum` | `hd` | `preview`, `mobile`, `sd`, `hd`, `1080`, `4k` | Standard resolution presets:<br>• `preview`: Fast low-res preview (approx. 360p)<br>• `mobile`: Optimized for mobile devices (480p)<br>• `sd`: Standard definition (576p)<br>• `hd`: High definition (720p)<br>• `1080`: Full HD (1080p)<br>• `4k`: Ultra HD (2160p) |
| `output.aspectRatio` | `enum` | `16:9` | `16:9`, `9:16`, `1:1`, `4:5`, `4:3` | Frame proportions:<br>• `16:9`: YouTube, standard widescreen TV and monitors<br>• `9:16`: TikTok, Instagram Reels, YouTube Shorts<br>• `1:1`: Instagram square feed posts<br>• `4:5`: Instagram portrait feed posts<br>• `4:3`: Classic standard display |
| `output.size` | `object` | — | `{ "width": number, "height": number }` | Custom pixel dimensions override (e.g. `{ "width": 1200, "height": 630 }`). |
| `output.fps` | `number` | `25` | `12` – `60` | Output frame rate in frames per second (e.g. `24`, `25`, `30`, `60`). |
| `output.quality` | `enum` | `medium` | `low`, `medium`, `high` | Compression quality level. Higher quality produces cleaner video with larger file sizes. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming data payload from upstream nodes. Enables using dynamic expressions to inject parameters such as `{{input.videoUrl}}`, `{{input.title}}`, or `{{input.duration}}` into the timeline. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when Shotstack successfully validates and queues the render request (HTTP 200/201). Contains the render job ID and dispatch status. |
| `error` | `object` | Emitted when the request fails due to invalid API credentials, malformed timeline parameters, or unserviceable media asset URLs. |

---

### Output Payload Examples

#### 1. Successful Render Request (`success`)

When a render is submitted, Shotstack responds with confirmation and a unique render `id`:

```json
{
  "success": true,
  "message": "Created",
  "response": {
    "message": "Render Successfully Queued",
    "id": "2bfa8024-9b16-43b5-9d51-177259163e79"
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates whether the render dispatch succeeded. |
| `message` | `string` | HTTP response status message from the Shotstack API. |
| `response.id` | `string` | The unique Render ID generated for the job. Use this ID to check render progress or retrieve the rendered video URL. |
| `response.message` | `string` | Status description confirming the video is queued in the render pipeline. |

> [!TIP]
> **Understanding Cloud Rendering:** Shotstack processes video renders asynchronously. Submitting a render queues the job immediately and returns a Render ID. Depending on your video duration, resolution, and effects, the cloud render typically completes within a few seconds to a couple of minutes.

#### 2. Error Response (`error`)

If the API rejects the composition, an error message is emitted:

```json
{
  "error": "Shotstack API error (Status: 400): The track 0 clip 0 asset src URL is not accessible."
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

The following example workflow demonstrates triggering a workflow manually, submitting a video editing composition to the **Shotstack: Video Editing** node, and logging the queued render response:

```fusion-workflow
src: example.workflow.json
title: Render an automated video with Shotstack
```

### Complete Workflow Definition

```json
{
    "name": "Shotstack: Video Editing",
    "variables": {},
    "secrets": {},
    "nodes": [
        {
            "position": {
                "x": 778,
                "y": 41
            },
            "width": 72,
            "height": 72,
            "selected": true,
            "id": "y66rhbm1uued5vc53ppflryf",
            "type": "action",
            "data": {
                "description": "Edit and render videos using Shotstack API.",
                "showRunningStatus": true,
                "_morphing": false,
                "name": "shotstack",
                "label": "Shotstack: Video Editing",
                "inputs": {
                    "input": {
                        "label": "Input"
                    }
                },
                "outputs": {
                    "success": {
                        "label": "Success",
                        "isConnectable": null
                    },
                    "error": {
                        "label": "Error",
                        "isConnectable": null
                    }
                },
                "parameters": {},
                "defaultOutput": null
            },
            "inputs": null,
            "outputs": null
        },
        {
            "position": {
                "x": 596.5,
                "y": 23.5
            },
            "width": 72,
            "height": 72,
            "id": "uzvbfawp071u4vkedxrkw8xs",
            "type": "trigger",
            "data": {
                "description": "Triggers the workflow manually.",
                "showRunningStatus": false,
                "_morphing": false,
                "name": "manual-trigger",
                "label": "Manual Trigger",
                "inputs": {},
                "outputs": {
                    "success": {
                        "label": "Success",
                        "isConnectable": null
                    },
                    "error": {
                        "label": "Error",
                        "isConnectable": null
                    }
                },
                "parameters": {},
                "defaultOutput": null
            },
            "inputs": null,
            "outputs": null
        },
        {
            "position": {
                "x": 933.5,
                "y": 23.5
            },
            "width": 256,
            "height": 100,
            "dragHandle": ".drag-handle__custom",
            "id": "wkf7oabdw2etow42mo3q0fv4",
            "type": "display",
            "data": {
                "description": "Logs the input data to the console.",
                "showRunningStatus": false,
                "_morphing": false,
                "name": "log",
                "label": "Log",
                "inputs": {
                    "input": {
                        "label": "Input"
                    }
                },
                "outputs": {
                    "success": {
                        "label": "Success",
                        "isConnectable": false
                    }
                },
                "parameters": {},
                "defaultOutput": null
            },
            "inputs": null,
            "outputs": null
        }
    ],
    "connections": [
        {
            "type": "directed",
            "id": "xy-edge__y66rhbm1uued5vc53ppflryfsuccess-wkf7oabdw2etow42mo3q0fv4input",
            "data": {
                "isAnimated": false
            },
            "source": "y66rhbm1uued5vc53ppflryf",
            "target": "wkf7oabdw2etow42mo3q0fv4",
            "sourceHandle": "success",
            "targetHandle": "input"
        },
        {
            "type": "directed",
            "id": "xy-edge__uzvbfawp071u4vkedxrkw8xssuccess-y66rhbm1uued5vc53ppflryfinput",
            "data": {
                "isAnimated": false
            },
            "source": "uzvbfawp071u4vkedxrkw8xs",
            "target": "y66rhbm1uued5vc53ppflryf",
            "sourceHandle": "success",
            "targetHandle": "input"
        }
    ],
    "status": "stopped",
    "tracingEnabled": true,
    "userId": "593a70dc-76b4-4abf-ac4f-de40c6d89861",
    "tenantId": "c4e72c92-6d3f-4ac3-b029-b722160d1088",
    "workspaceId": "9aae95bf-7bb8-4ec9-b268-e9337a62381f",
    "folderId": null,
    "version": 1,
    "createdAt": 1790839855006,
    "updatedAt": 1790839906005
}
```

---

### Practical Configuration Examples

#### Example 1: Vertical Social Media Video (TikTok / Reels / Shorts in 9:16)

Create a 10-second vertical video featuring a background video, a floating animated title, and a background soundtrack:

```json
{
  "apiKey": "{{secrets.SHOTSTACK_API_KEY}}",
  "environment": "sandbox",
  "timeline": {
    "background": "#000000",
    "soundtrack": {
      "src": "https://shotstack-assets.s3.amazonaws.com/music/freepd/chill-background.mp3",
      "effect": "fadeInFadeOut",
      "volume": 0.8
    },
    "tracks": [
      {
        "clips": [
          {
            "asset": {
              "type": "title",
              "text": "5 Tips for Workflow Automation",
              "font": {
                "family": "Montserrat",
                "size": 42,
                "weight": 700,
                "color": "#FFFFFF"
              },
              "animation": {
                "preset": "zoomIn",
                "duration": 1
              }
            },
            "start": 1,
            "length": 6,
            "position": "center",
            "transition": {
              "in": "fade",
              "out": "fade"
            }
          }
        ]
      },
      {
        "clips": [
          {
            "asset": {
              "type": "video",
              "src": "https://shotstack-assets.s3.amazonaws.com/footage/mountain-drone.mp4"
            },
            "start": 0,
            "length": 10,
            "fit": "cover"
          }
        ]
      }
    ]
  },
  "output": {
    "format": "mp4",
    "resolution": "1080",
    "aspectRatio": "9:16",
    "fps": 30,
    "quality": "high"
  }
}
```

#### Example 2: Branded Watermark & Video Collage

Layer a corporate logo over a background video with a lower-third text banner:

- **Track 1 (Top Layer):** Brand logo image in the top-right corner (`position: "topRight"`, `scale: 0.25`, `offset: { "x": -0.05, "y": -0.05 }`).
- **Track 2 (Middle Layer):** Lower-third title text (`position: "bottom"`, `start: 2`, `length: 5`).
- **Track 3 (Base Layer):** Main interview or screen recording MP4 video (`start: 0`, `length: 15`, `fit: "cover"`).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Shotstack API error (Status: 401)`

- **Cause:** The provided `apiKey` is missing, expired, or invalid.
- **Solution:** 
  1. Verify your API key in the [Shotstack Dashboard](https://dashboard.shotstack.io/).
  2. Confirm you are using the correct environment: Sandbox keys require `environment: "sandbox"`, while Production keys require `environment: "production"`.

#### `Shotstack API error (Status: 400): asset src URL not reachable`

- **Cause:** One of the media URLs in `src` (video, audio, image, font) is private, returns a 404/403 status, or requires authentication.
- **Solution:** Ensure all media assets are publicly accessible over HTTPS (e.g. hosted on AWS S3 with public read permissions, Cloudinary, or a public CDN).

#### `Black frames or unexpected gaps in video`

- **Cause:** Misaligned `start` and `length` timestamps between clips across the timeline.
- **Solution:** Verify that the `start` time of each clip matches your intended sequence. For continuous playback, ensure clip N+1 starts at `clip N start + clip N length`.

#### `Text styling or custom font fails to render`

- **Cause:** The custom font URL in `timeline.fonts` is invalid or CORS-restricted.
- **Solution:** Supply a direct download link to a `.ttf` or `.woff` file, or use one of Shotstack's built-in standard fonts (e.g. `Montserrat`, `Roboto`, `Open Sans`).

---

### Error Reference

| Status Code | Error Message | Possible Cause | Recommended Resolution |
|-------------|---------------|----------------|------------------------|
| `400` | `Bad Request` | Malformed JSON timeline structure, missing required fields, or invalid asset types | Review the timeline structure against the configuration guide. |
| `401` | `Unauthorized` | Missing or invalid `apiKey` | Double-check API key credentials in the Shotstack console. |
| `403` | `Forbidden` | Account subscription limit reached or quota exceeded | Check your Shotstack usage quotas and account plan. |
| `422` | `Unprocessable Entity` | Parameter validation failure (e.g. negative clip duration or conflicting resolution settings) | Ensure `start` and `length` values are positive numbers. |
| `429` | `Too Many Requests` | Shotstack API rate limit exceeded | Add rate-limiting or delay nodes between rapid sequential render calls. |
| `500` | `Internal Server Error` | Shotstack rendering cluster disruption | Check the [Shotstack Status Page](https://status.shotstack.io/). |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Manual Trigger](./manual-trigger.md) — Trigger workflows manually for testing and development
- [Function](./function.md) — Dynamically construct and transform Shotstack timeline JSON payloads
- [Log](./log.md) — Inspect queued render IDs and diagnostic outputs
- [D-ID](./did.md) — AI avatar video generation
- [Synthesia](./synthesia.md) — AI video presentation generation
- [HeyGen](./hey-gen.md) — AI spokesperson and avatar video rendering

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-01 | Initial release of Shotstack Video Editing node with multi-track timeline compositing, rich media asset rendering, transitions, audio soundtrack controls, and sandbox/production environments. |

<!-- /SECTION: changelog -->

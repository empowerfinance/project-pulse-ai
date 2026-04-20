# Pre-Demo Slides — Structure & Visuals

Brand aesthetic (from the problem-definition template):
- **Background:** cream / warm off-white (`#F1ECE0` ish)
- **Primary text:** near-black
- **Accent:** lime/chartreuse pill highlight (`#C6E812` ish)
- **Type:** bold sans-serif for display, italic emphasis on key words
- **Layout:** generous whitespace, left-heavy text, right-side content blocks

Keep all five slides in this system so the deck feels unified.

---

## Slide 1 — The Question

**Purpose:** Set up the hook. Audience should be curious, not yet know the punchline.

**Layout:** Mostly empty. One big question, left-aligned.

**Copy:**

> Pill tag (top left): `AI SPRINT · PROJECT PULSE`
>
> Display headline (huge, italic emphasis):
> **Of EPD's *~32 active initiatives*, how many have a health status set in Linear?**
>
> Footer: `2026 · Tilt`

**No visual needed.** The emptiness is the point — it should feel like the room is waiting.

**Presenter cue:** Pause. Let the audience guess. Don't answer. Click to next slide.

---

## Slide 2 — The Answer

**Purpose:** Reveal + visual metaphor. The number lands, the image tells them *why* that number is bad.

**Layout:** Two-column. Left: giant number + one-line caption. Right: the noise→signal visual.

**Copy:**

> Pill tag: `THE GAP`
>
> Left column — enormous numeral: **6**
> Just below, smaller: `out of ~32 active EPD initiatives`
>
> Below that, one line in italic: *Multiple active projects haven't had a status update in 30+ days.*
>
> Right column: **the noise-to-signal visual** (see image notes below)

---

## Slide 3 — Define the Problem

Already drafted — use the four-field fill-in from `offsite-presentation.md`. No new visuals needed; the template handles it.

---

## Slide 4 — What I Built

**Purpose:** One glance = "I get it." Four messy inputs resolve into one clean pulse.

**Layout:** Full-bleed diagram. Title top-left.

**Copy:**

> Pill tag: `PROJECT PULSE AI`
>
> Headline: **Four noisy signals. *One clean pulse.***
>
> Subhead (small, under headline): *Synthesizes Linear, GitHub, Slack, and Amplitude into evidence-backed status — from a single project up to the whole company.*

**Visual concept — "signal convergence":**

Four horizontal waveforms stacked on the left side, each styled to evoke its source:

| Source | Waveform style |
|---|---|
| **Linear** | Square-wave / digital step pattern (tickets, discrete events) |
| **GitHub** | Sawtooth or spiky bursts (PR merges, commits) |
| **Slack** | Dense, chatty AM-modulation-looking sine noise (constant chatter) |
| **Amplitude** | Smooth analog sine wave, slowly varying (metric trends) |

Each wave is labeled on the left with its source name + a small monochrome icon. All four waves flow rightward into a **convergence node** (a glowing lime dot or wedge) roughly center-right. Out of the node emerges **one bold, clean, lime-green ECG/heartbeat pulse line** extending to the right edge, labeled `PULSE`.

Optional detail: a faint vertical dashed line at the convergence point labeled `SYNTHESIS` or `CLAUDE + MCP`.

**Why this works:** The four-into-one is instantly readable. The waveform styles make each source feel distinct without needing to read labels. The output pulse line matches the brand accent color, tying the visual into the deck's DNA.

---

## Slide 5 — Demo

**Purpose:** Transition beat. Gives you 2 seconds to switch from Keynote to browser.

**Layout:** Single word on cream background. Pill tag optional.

**Copy (pick one):**

> Option A — restrained:
> `DEMO.`
>
> Option B — matches the slide-1 italic treatment:
> *Let's watch it run.*
>
> Option C — call-back to the hook:
> *From **8** to **45**.*

**Visual:** None, or the same faint pulse line from Slide 4 running across the bottom as a brand tie-in.

---

## How to generate the actual images

You have three practical paths. Ranked by fit for this deck.

### Path 1 — SVG (best fit, I can generate now)

The Slide 2 and Slide 4 visuals are geometric/abstract — waveforms, pulses, convergence. These render beautifully as SVG, will perfectly match the brand palette (exact hex codes, not approximate), scale losslessly, and can be dropped into Keynote/Google Slides/Figma as images.

**I can generate these SVGs directly in this session** if you want — just say the word and I'll write them to `docs/slides/` as standalone files you can export to PNG or drop straight in.

### Path 2 — AI image generation (DALL-E / Midjourney / Claude)

Good for more photorealistic or textured visuals. Paste these prompts:

**Slide 2 — noise and pulse:**
> Abstract editorial illustration, minimalist, cream off-white background (#F1ECE0), thin line art. Top two-thirds: four overlapping noisy horizontal waveforms in muted grey, chaotic and low-contrast, evoking scattered data signals. Bottom third: one clean, bright chartreuse-green (#C6E812) ECG heartbeat pulse line running horizontally, sharp and glowing, cutting through the noise. No text. Flat vector style, generous whitespace, wide aspect ratio 16:9.

**Slide 4 — signal convergence:**
> Editorial infographic, cream off-white background (#F1ECE0), flat vector style, minimalist. Left side: four horizontal waveform lines stacked vertically, each in a distinct style — a square wave, a jagged sawtooth, a dense sine-wave noise pattern, and a smooth slow sine curve — all in muted dark grey. Each waveform labeled with a small monochrome icon. All four waves flow rightward, converging at a bright chartreuse-green (#C6E812) glowing node in the center-right. From the node, a single bold clean green ECG heartbeat pulse line emerges and extends to the right edge. 16:9 aspect ratio, no text, lots of whitespace.

Tip: generate 3-4 variations and pick the one where the chartreuse matches your deck accent. You may need to color-correct in post.

### Path 3 — Figma (you have MCP access)

If you want the visuals fully editable and on-brand, build them in Figma. You have the Figma MCP connector active, so:

- Create a new Figma file with `create_new_file`
- Sketch each visual as simple shapes + strokes
- Use the exact Tilt accent hex once you sample it from an existing Tilt deck
- Export as PNG/SVG and drop into your slide tool

This is the most work but gives you full editorial control and native brand consistency.

---

## Recommendation

1. Use the **SVG path** for Slides 2 and 4 — it's the cleanest match for the abstract waveform concepts and I can generate them now.
2. Slides 1, 3, and 5 are type-only — build those directly in your slide tool using the template you already have.
3. If the SVGs feel too clinical, fall back to Path 2 with the prompts above.

# Pre-Demo Slides — Structure & Visuals

Brand aesthetic (from the problem-definition template):
- **Background:** cream / warm off-white (`#F1ECE0` ish)
- **Primary text:** near-black
- **Accent:** lime/chartreuse pill highlight (`#C6E812` ish)
- **Type:** bold sans-serif for display, italic emphasis on key words
- **Layout:** generous whitespace, left-heavy text, right-side content blocks

Keep all four slides in this system so the deck feels unified.

---

## Slide 1 — The Status Tax

**Purpose:** Open cold. Establish the problem upfront by making the cost of status reporting tangible and showing how it compounds with the org hierarchy.

**Layout:** Full-bleed. Left/center: hierarchy pyramid (32 → 4 → 2 → 1). Right: big number. Bottom: one-line caption.

**Copy (all rendered inside the SVG):**

> Pill tag (top left): `THE STATUS TAX`
>
> Subtitle (below pill, italic): *A slice of Tilt · 2 BUs · 4 pods · 32 initiatives*
>
> Row labels (left edge): `COMPANY ×1`, `BUs ×2`, `PODS ×4`, `INITIATIVES ×32`
>
> Right column: **~50** / HRS / MONTH / *a full work week, every month*
>
> Bottom caption: *At every level, a human reads the level below and writes for the level above.*

**Visual:** Use `docs/slides/slide-2-status-tax.svg` as the full-bleed slide image.

**Presenter cue:** Open on this slide for the full 1:30 opening. Walk the pyramid bottom-up as you narrate. Repeat the "slice of Tilt" framing three times: up front, when the number lands, and as the exit ("across the whole company, it's materially more"). Acknowledge the 50-hour number is a ballpark — the point is that it's non-trivial.

---

## Slide 2 — Define the Problem

Use the four-field fill-in from `offsite-presentation.md`. No new visuals needed; the template handles it.

---

## Slide 3 — What I Built

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

Each wave is labeled on the left with its source name. All four waves flow rightward into a **convergence node** (a glowing lime dot) roughly center-right. Out of the node emerges **one bold, clean, lime-green ECG/heartbeat pulse line** extending to the right edge, labeled `PULSE`.

Optional detail: a faint label at the convergence point reading `SYNTHESIS · CLAUDE + MCP`.

**Visual:** Use `docs/slides/slide-4-convergence.svg`.

**Why this works:** The four-into-one is instantly readable. The waveform styles make each source feel distinct. The output pulse line matches the brand accent color. And it visually inverts the hierarchy from Slide 1 — instead of humans writing upward, signals flow into one synthesized output.

---

## Slide 4 — Demo

**Purpose:** Transition beat. Gives you 2 seconds to switch from Keynote to browser.

**Layout:** Single word on cream background. Pill tag optional.

**Copy (pick one):**

> Option A — restrained:
> `DEMO.`
>
> Option B — matches the slide-1 italic treatment:
> *Let's watch it run.*

**Visual:** None, or the same faint pulse line from Slide 3 running across the bottom as a brand tie-in.

---

## How to generate the actual images

The SVGs for Slides 1 and 3 are already generated at `docs/slides/`:

- `slide-2-status-tax.svg` — hierarchy pyramid for Slide 1
- `slide-4-convergence.svg` — signal convergence for Slide 3
- `slide-4-convergence.png` — PNG export of the above

(Filenames are historical — they reflect earlier slide numbering and weren't renamed when the deck was restructured.)

Slides 2 and 4 are type-only — build those directly in your slide tool using the existing template.

### Fallback: AI image generation

If the SVGs feel too clinical, you can regenerate with image-generation tools:

**Status tax (Slide 1) — hierarchy pyramid:**
> Minimalist editorial infographic, cream off-white background (#F1ECE0), flat vector style. A pyramidal node-link diagram showing 32 small dots at the bottom narrowing up through 4 medium dots, 2 larger dots, to 1 dark node at the top. Thin grey lines connect each level upward, showing aggregation flow. Row labels on the left side. Lots of whitespace, 16:9 aspect ratio, no distracting color — only dark greys and the cream background.

**Signal convergence (Slide 3):**
> Editorial infographic, cream off-white background (#F1ECE0), flat vector style, minimalist. Left side: four horizontal waveform lines stacked vertically, each in a distinct style — a square wave, a jagged sawtooth, a dense sine-wave noise pattern, and a smooth slow sine curve — all in muted dark grey. All four waves flow rightward, converging at a bright chartreuse-green (#C6E812) glowing node in the center-right. From the node, a single bold clean green ECG heartbeat pulse line emerges and extends to the right edge. 16:9 aspect ratio, no text, lots of whitespace.

### Figma (if you want fully editable versions)

You have Figma MCP access. To re-render either SVG in Figma: create a new file, recreate the shapes, sample exact Tilt brand hex codes, export. Most work, most editorial control.

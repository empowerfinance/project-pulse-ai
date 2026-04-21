# Presenter Note Cards

Quick-glance bullets for stage use. Full script is in `offsite-presentation.md`.

**Total runtime: 14:45 with 15s buffer. Trim order if over: publishing → pod rollup.**

---

## Card 1 — Status Tax (0:00–1:30)
**Slide:** The Status Tax (pyramid)

- Open: "A tax we're all paying — nobody has it on their job description"
- Define: time turning raw work into readable status (not the work itself)
- **Slice of Tilt:** 2 BUs · 4 pods · 32 initiatives — *say it upfront, say it again at the number, say it again at the exit*
- Walk pyramid bottom-up: 32 initiatives → 4 pods (weekly) → 2 BUs → 1 company (monthly digest)
- Land the number: **~50 hrs/month = a full work week, every month**
- Hedge: "30? 80? Non-trivial either way. Company-wide = materially more."
- Two things broken: (1) PM/EM time not returned, (2) info still in 4 tools — no consolidated async view
- Intro self: "I'm Thomas. Sprint Week. **Project Pulse AI**."
- Pivot: *"Let me set the problem up formally."*

---

## Card 2 — Define the Problem (1:30–3:45)
**Slide:** Define the Problem (4 boxes)

Walk the slide's 4 quadrants — don't go off-script, the slide is visible.

- **Problem:** fragmented across 4 tools · manual · inconsistent · delayed · often doesn't happen
- **Who:** EMs + PMs primary (paying the tax) · BU leads + execs secondary (wanting visibility)
- **Why:** not a reporting problem — a **visibility gap**. Blockers buried in Slack. PMs curate instead of deciding on product.
- **Success:** (1) every initiative current w/ evidence, (2) PM time drops, (3) company-wide scan in <1 min, on demand

Pivot: *"Let me show you what I built."*

---

## Card 3 — What I Built (3:45–4:30)
**Slide:** Noisy Signals → Single Pulse

- One sentence: synthesizes Linear + GitHub + Slack + Amplitude → status reports at every level (project → company)
- **Not replacing** anyone's process — layers on top, reads what's there
- Caveat: **day and a half solo, rough prototype** — enough to judge utility
- Pivot: *"Let me just demo it."*

---

## Card 4 — Project Demo (4:30–5:30) — LIVE
**Screen:** Claude.ai browser project

- Paste Linear URL for **"Revamp the Referral Program"**
- Prompt: **"summarize this project"**
- Kick off generation (~60–90s) → **keep talking through Card 5 while it runs**

---

## Card 5 — How It Works (5:30–6:45) — while generation runs
**Screen:** project knowledge / config files

- It's a **Claude skill** — structured natural-language program
- Unlock = **MCP** (Model Context Protocol) → Claude reads Linear/Slack/Amplitude directly. No API wrappers, no middleware.
- Same skill powers CLI *and* this browser version
- Each pod = **one JSON config** (team key, Slack channels, Amplitude charts)

---

## Card 6 — Walk Project Output (6:45–8:00)
**Screen:** generated project report

Three points to hit:

- **Evidence-linked** — every claim cites source (Linear update, cycle, Slack msg). Leadership won't trust AI without audit trail.
- **Cross-source synthesis** — Linear (what's in progress) + Slack (what's blocking) + Amplitude (is it moving metric?). Parallel queries, one coherent picture.
- **Silent-failure detection** — flags initiatives with no PRs / no issue movement / no Slack. Humans miss these at scale.

---

## Card 7 — Pod Rollup (8:00–9:15) — PRE-GENERATED
**Screen:** CAA pod rollup (ready in tab)

- *"Generated earlier so we don't watch a spinner."*
- Overview table · on-track / at-risk / off-track counts · velocity
- Initiative status table — one-line signal each
- Top risks w/ evidence links
- Key wins
- Metrics impact — Amplitude trends WoW
- Slack signals — attributed quotes from pod discussions
- Land: **"~1 hour of manual work → 90 seconds."**

---

## Card 8 — Company Pulse (9:15–10:45) — PRE-GENERATED
**Screen:** company pulse (ready in tab)

- Rolls up Cash Advance + Credit Card BUs (extensible via 1 config file)
- Exec overview · health by BU · top company risks · top 5 wins
- **Favorite section:** cross-BU dependencies — detects coupling that usually only surfaces when something breaks
- Recap hierarchy: **project → initiative → pod → BU → company**. Each level more condensed. Company = 1-min exec scan.

---

## Card 9 — Publishing (10:45–12:15)
**Screen:** Linear initiative w/ Pulse update + `#pulse-reports` Slack channel

- **Linear:** native initiative status update, one per initiative, correct health classification. Same place PMs write manually. *Show the one it generated.*
- **Slack:** `#pulse-reports` — summary card at top, full report in thread
- **Always confirms** before posting. Nothing ships without human yes.

*(First cut if running long.)*

---

## Card 10 — Broader Insight (12:15–13:15)
**Slide:** none / back to deck

- Buildable in a day = **MCP unlocked it** — tools read directly, no plumbing layer
- Interesting work moved to the **synthesis layer** — defining "on track," choosing signals, making claims auditable
- Pattern generalizes: **anywhere at Tilt someone aggregates 3+ tools into a doc, it's a Pulse-shaped problem**

---

## Card 11 — Next Steps (13:15–14:15)

- **Scheduled runs** — Monday auto-post to Slack; monthly roundup on last Friday
- **Drift detection** — WoW diffs ("was on track, now at risk")
- **Broader adoption** — pilot 2–3 more pods beyond Cash Advance / Credit Card
- **Richer integrations** — auto-discover Amplitude charts; pull Notion specs

---

## Card 12 — Close (14:15–14:45)

- Manual, inconsistent, time-consuming → **30-second conversation**
- Goal: every initiative from "unknown" → evidence-backed
- Give PMs their sprint hours back for product decisions
- Repo + browser project link in README
- **"Grab me at the break if you want to try it on your pod. Thanks."**

---

## Demo-day sanity checklist (re-read morning of)

- [ ] Pre-open tabs: Claude project → Linear "Revamp Referral Program" → Linear initiative w/ Pulse update → `#pulse-reports` Slack
- [ ] Pre-generate + save in tabs: pod rollup (CAA), company pulse
- [ ] Verify MCP connectors: Linear, Slack, Amplitude (Amplitude has flaked before — have screenshot fallback)
- [ ] Scratch doc with project URL + prompt text for copy-paste
- [ ] Rehearse once with a timer. If >15:00, cut Card 9 (publishing) first, then trim Card 7 (pod rollup).

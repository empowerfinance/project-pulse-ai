# Project Pulse AI

> Automated information synthesis: compile and summarize multiple sources into cohesive updates across organizational levels. Improve leadership visibility into project health.

---

## Problem Definition

Engineering and product leadership lack timely, consistent visibility into initiative progress, risks, and user impact because updates are fragmented across Slack, Linear, GitHub, Notion, and Amplitude, requiring manual synthesis that is inconsistent, delayed, and difficult to translate into clear, presentation-ready narratives for leadership forums.

## Evidence

| Source | Finding | Signal Type |
|---|---|---|
| Leadership / EM feedback | Weekly updates are manually compiled, inconsistent across pods, and often stale by the time they reach leadership | Qualitative |
| Process observation | Initiative updates require manual aggregation from Slack, Linear, GitHub, Notion, and Amplitude, taking significant PM/EM time each sprint | Qualitative |
| Linear data | Only ~8 of 45+ active initiatives have `health` status set — most are null | Quantitative |
| Execution gaps | Blockers and risks often surface late or informally (e.g., buried in Slack threads) | Qualitative |
| Monthly roundups | PMs manually compile updates into presentation format, leading to duplicated effort and inconsistent storytelling across pods | Qualitative |
| Org scaling | Increasing number of pods and cross-team dependencies makes manual rollups less reliable | Directional |

## Who We're Solving For

**Primary:** EMs, PMs, and Pod leads responsible for initiative updates and monthly roundup presentations
**Secondary:** BU and executive leadership who need high-level visibility into project health, risks, and business impact

## What We're NOT Solving

- Replacing Linear, GitHub, Slack, Notion, or Amplitude as systems of record
- Building a full analytics platform
- Enforcing rigid process changes on pods
- Real-time alerting (initially periodic: weekly + monthly)
- Fully automated metric attribution (initial version uses heuristic linkage)
- Automated presentation design (slides, branding)

## Success Metrics

**Primary:** Reduce time spent on manual status reporting by PMs/EMs by 50% within one sprint cycle, while maintaining or improving perceived quality of updates.

**Supporting:**
- [ ] % of initiatives with auto-generated updates (coverage)
- [ ] Accuracy of AI-generated status vs human-edited status (agreement rate)
- [ ] % of updates that include relevant user impact signals from Amplitude
- [ ] Reduction in "unknown" or missing health status at initiative level
- [ ] Engagement with generated summaries

**Guardrails:**
- [ ] Leadership trust: summaries perceived as reliable and actionable
- [ ] No incorrect or misleading metric attribution
- [ ] No increase in missed critical risks/blockers
- [ ] No additional manual burden on ICs

---

## Architecture

### Pipeline

```
Sources → Normalize → Summarize → Aggregate → Publish
```

### Data Sources (MCP Connected)

| Source | Connection | What We Pull |
|---|---|---|
| Linear | MCP (claude.ai) | Initiatives, projects, issues, comments, cycles, status changes |
| GitHub | `gh` CLI | PRs, commits, merge activity |
| Slack | MCP (claude.ai) | Channel messages, threads, structured updates |
| Notion | MCP (claude.ai) | Specs, meeting notes, docs |
| Amplitude | MCP (claude.ai) | Dashboards, charts, experiment results, user metrics |

### Canonical Update Event Schema

```json
{
  "source": "linear | github | slack | notion | amplitude",
  "initiative_id": "string",
  "project_id": "string",
  "timestamp": "ISO-8601",
  "type": "progress | blocker | decision | risk | metric",
  "content": "string",
  "author": "string",
  "confidence": 0.0-1.0,
  "evidence_links": ["url"]
}
```

### Derived Summary Schema

```json
{
  "level": "initiative | pod | BU | company",
  "entity_id": "string",
  "entity_name": "string",
  "time_window": "last_7_days | last_14_days | last_30_days",
  "status": "on_track | at_risk | off_track",
  "progress": "string",
  "blockers": ["string"],
  "risks": ["string"],
  "wins": ["string"],
  "next_steps": ["string"],
  "metrics_impact": ["string"],
  "confidence": "low | medium | high",
  "supporting_events": ["event_ids"],
  "evidence_links": ["url"]
}
```

### Output Modes

1. **Operational (weekly):** Structured updates posted to Linear initiative updates and/or Slack
2. **Presentation (monthly):** Narrative summaries for roundup meetings with highlights, metrics, and talking points

### Multi-Level Views

| Level | Audience | Content |
|---|---|---|
| Executive | C-suite | % on track, top 5 risks, key wins. Zero noise. |
| BU | BU leads | Summary paragraph, key risks, dependencies |
| Pod | Pod leads, EMs | Initiative summaries, cycle velocity, blockers |
| Initiative | PMs, EMs, ICs | Structured update with full audit trail to source evidence |

---

## Org Data Landscape (from Discovery)

### Pilot: Cash Advance - Activation (CAA)

- **Team ID:** `3bd61709-d853-4ee7-ac10-35d92a0ae196`
- **Team key:** `CAA`
- **Current cycle:** Cycle 27 (Mar 23 – Apr 6), 43/121 issues complete (35%)
- **Active FY26Q1 initiatives:** 5 (targeting Mar 31)
- **New FY26Q2 initiatives:** 4 (targeting Jun 30)
- **Key team members:** Thomas Quinto, Max Godfrey (iOS), Samer Alsayegh (Web), Frankie DeNell (Web), Steve Magelowitz (Server), Charles Luo (Server), Chloe Choe (Server), Andrew Kuo (PM), Nina Dixon (Design)

### Org-wide Scale

- **~45+ active initiatives** across CAA, Growth, Credit, MX, PH, IN, DS/ML, SubEx, Compliance
- **Naming conventions vary** by area: `[CAA - FY26Q1]`, `[PH OKR]`, `[IN]`, `[Card]`, `Growth 2026Q2 OKR`
- **Multi-team projects are common** — cross-pod dependency detection is valuable
- **Health field mostly empty** — confirms the core visibility gap

---

## Phases

### Phase 0: Discovery [COMPLETE]
- [x] Validate MCP connections (Linear, Slack, GitHub, Notion, Amplitude)
- [x] Explore Linear structure: teams, projects, initiatives, cycles
- [x] Map CAA team data: initiatives, active projects, recent issues
- [x] Map org-wide initiative landscape
- [x] Identify pilot pod (CAA)

### Phase 1: Single Initiative Summary (MVP) [COMPLETE]
> Goal: Prove AI can generate a useful, trustworthy update for one initiative.

**Scope:** For a single CAA initiative, generate a structured update from real data.

- [x] **1.1 Linear signal extraction**
  - Pull all issues for initiative (via project linkage) updated in last 14 days
  - Extract: status changes, completions, new issues, comments, cycle progress
  - Filter noise (e.g., SweeperAI auto-generated bug summaries)

- [x] **1.2 GitHub signal extraction**
  - Find PRs linked to initiative issues (via branch naming convention `caa-XXXX-*`)
  - Extract: open PRs, merged PRs, review status

- [x] **1.3 AI summarization**
  - Design prompt: structured update from Linear + GitHub signals
  - Output: progress, blockers, status classification, next steps, confidence, evidence links
  - Status classification rules:
    - `on_track`: steady progress, no unresolved blockers
    - `at_risk`: blockers exist, or no progress in 7+ days, or cycle velocity < 30%
    - `off_track`: deadlines slipped, critical blockers unresolved

- [x] **1.4 Validation**
  - Generated summary for CAA initiative "[CAA - FY26Q1] Increase mobile 14d register -> CA activation by 10%"
  - See `reports/caa/initiative-summary-mobile-activation-2026-03-30.md`

- [x] **1.5 Codify as reusable skill**
  - Created `/pulse` Claude Code skill with config-driven pod support
  - Created setup docs (README.md) for any team to self-serve
  - Pod config: `config/caa.json` + `config/example.json` template

- [ ] **1.6 Output to Linear** (deferred to Phase 3)
  - Post as initiative status update or project update in Linear

**Deliverable:** Working `/pulse` skill that generates initiative updates for any configured team.

### Phase 2: Multi-Source Enrichment [COMPLETE]
> Goal: Layer in Slack and Amplitude to improve signal quality.

- [x] **2.1 Slack signal extraction**
  - Discovered CAA Slack channels: `#pod-cash-advance` (main), `#feat-onboarding-to-ca-activation`, `#feat-cash-advance-web-onboarding`, `#alerts-cash-advance-product`
  - Added channel IDs to pod config (`config/caa.json`)
  - Updated pulse skill with Slack search strategy (keyword + risk/blocker searches)
  - Validated: found standup updates, urgency signals, team discussions

- [x] **2.2 Amplitude signal extraction**
  - Mapped CAA to key charts:
    - `fdq1dvfd` — "Cash advance activation, 14d" (the north star metric)
    - `v329bnbv` — "Cash advance activation, 1h" (real-time health)
  - Mapped to dashboards:
    - `ii1xadl` — "#pod-cash-advance critical metrics" (920 views — most-used)
    - `4b74u2tz` — "[Web] Activation in web" (Andrew Kuo's, 105 views)
  - Added chart/dashboard IDs to pod config
  - Updated pulse skill with Amplitude query strategy and interpretation rules
  - Validated: pulled 30-day trend data showing iOS 14d activation declining ~14pp, Android declining ~4pp

- [ ] **2.3 Notion signal extraction** (deferred — lower priority, most signal is in Linear/Slack/Amplitude)
  - Pull specs or meeting notes linked to active projects
  - Extract key decisions or context not captured in Linear

- [x] **2.4 Enhanced summarization**
  - Updated pulse skill output template with "Metrics Impact" and "Slack Signals" sections
  - Added metric trend interpretation to status classification rules
  - Synced skill to repo (`skill/pulse.md`)

**Deliverable:** Initiative summaries enriched with Slack discussions, Amplitude metrics, and Notion context.

### Phase 3: Pod-Level Rollup [COMPLETE]
> Goal: Aggregate initiative summaries into a pod-level view.

- [x] **3.1 Pod rollup generation**
  - Generated full CAA pod rollup from 5 active Q1 initiatives
  - Includes: overview table (on track/at risk/off track counts), per-initiative status with key signal, top risks, key wins, metrics impact, Slack signals, cross-team dependencies, leadership summary
  - Cycle velocity included (43/121, 36%)
  - Detected silent failures: two web-focused initiatives with no recent activity

- [x] **3.2 Config-driven pod definitions**
  - Pod config (`config/caa.json`) now includes Slack channels (pod + feature + alerts) and Amplitude charts/dashboards
  - `config/example.json` template available for any team
  - System is fully config-driven — no code changes needed for new pods

- [ ] **3.3 Output to Slack / Notion** (deferred to Phase 5)
  - Post pod summary to Slack channel
  - Create or update Notion page with formatted summary

- [ ] **3.4 Extend to second pod** (deferred to Phase 5)
  - Apply to one additional pod (e.g., Growth) to validate generalizability

**Deliverable:** Working pod-level rollup for CAA via `/pulse caa --pod`.

### Phase 4: Presentation Mode + Executive View [COMPLETE]
> Goal: Generate monthly roundup content and top-level dashboard.

- [x] **4.1 Monthly narrative generation**
  - Generated full March 2026 monthly roundup for CAA
  - 30-day window, presentation-ready narrative with executive summary, per-initiative highlights (summary, impact, highlights, risks, next steps), key metrics, team wins, looking ahead
  - See `reports/caa/monthly-roundup-2026-03.md`
  - Pulse skill supports `--monthly` flag for this output mode

- [x] **4.2 Executive dashboard view**
  - Pod rollup includes overview table (on track / at risk / off track counts, cycle velocity)
  - Top risks with evidence links
  - Key wins
  - Cross-team dependencies section
  - See `reports/caa/pod-rollup-2026-03-30.md`

- [x] **4.3 Drill-down audit trail**
  - Every claim links to source: Linear issues, project updates, GitHub PRs, Amplitude charts
  - Initiative summary → project detail → issue/PR level all connected

- [x] **4.4 Silent failure detection**
  - Successfully detected two web-focused initiatives with no activity in 14 days
  - Flagged as "needs PM attention" in both pod rollup and monthly roundup
  - Classification rules in skill handle: no updates, no PRs, no ticket movement → at_risk

**Deliverable:** Monthly roundup (`--monthly`), pod rollup (`--pod`), and initiative summary all working with full drill-down and silent failure detection.

### Phase 5: Automation + Scale (Future)
> Goal: Run on autopilot for all pods.

- [ ] Scheduled execution (cron / Claude Code triggers)
- [ ] All pods onboarded with config
- [ ] Drift detection (week-over-week comparison)
- [ ] Dependency graph extraction
- [ ] Feedback loop: track human edits to improve prompts

---

## Status Classification Rules

| Signal | Interpretation |
|---|---|
| Issues completing on pace with cycle | on_track |
| Blockers exist (labeled or mentioned) | at_risk |
| No progress on initiative issues in 7+ days | at_risk |
| Cycle velocity < 30% at midpoint | at_risk |
| Target date passed with incomplete work | off_track |
| Critical blocker unresolved > 7 days | off_track |
| Amplitude metrics declining post-launch | at_risk |
| No activity across all sources (silent failure) | at_risk |

## Key Design Principles

1. **Every summary must be traceable back to raw evidence.** If leadership can't drill down, they won't trust it.
2. **Flexible input, structured output.** Don't force pods to change behavior — normalize their existing signals into a standard schema.
3. **Config-driven, not code-forked.** Each pod is a config entry, not a special case.
4. **Filter noise aggressively.** SweeperAI tickets, bot messages, stale issues — exclude by default.
5. **Confidence scoring.** Low data = low confidence. Say so explicitly.
6. **Zero-friction adoption.** Any team at Tilt should be able to use this with minimal setup. No new tools, no process changes, no required input formats. The system reads from tools teams already use (Linear, Slack, GitHub). Onboarding a new pod = providing a small config (team key, channels, repos). If adoption requires training or behavior change, it will fail.

---

## Tech Stack

- **Runtime:** Claude Code with MCP integrations
- **Data sources:** Linear MCP, Slack MCP, Notion MCP, Amplitude MCP, GitHub CLI
- **AI:** Claude (structured JSON output, few-shot prompting)
- **Config:** Markdown or JSON pod definitions
- **Output:** Linear status updates, Slack messages, Notion pages
- **Storage:** Local markdown files (initially), database if needed at scale

---

## Open Questions

- [ ] What Slack channels does CAA use? (need to identify for Phase 2)
- [ ] Which Amplitude dashboards/charts map to CAA initiatives?
- [ ] What does a "good" initiative update look like today? (get examples from Andrew Kuo)
- [ ] How should we handle multi-team projects in rollups? (count toward all teams, or primary only?)
- [ ] What's the appetite for auto-posting vs human-in-the-loop review?
- [ ] Should this run as a Claude Code skill, a scheduled job, or both?

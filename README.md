# Project Pulse AI

[Open in Claude](https://claude.ai/project/019d6411-a3e6-70f2-b971-a8ac0ea9aca0)

Automated, multi-level status summaries powered by AI. Synthesizes signals from Linear, GitHub, Slack, and Amplitude into structured, evidence-backed updates — from individual projects up to company-wide executive views.

## How it works

Pulse reads from the tools your team already uses — no new processes, no new fields, no behavior changes required.

1. **Resolves** the target at the requested level (project, initiative, pod, BU, or company)
2. **Gathers signals** in parallel from:
   - **Linear:** issues, project status updates, cycle velocity, initiative health
   - **GitHub:** PRs (merged, open, stale) matched via branch naming convention (CLI only)
   - **Slack:** keyword searches in pod and feature channels for blockers, progress, decisions
   - **Amplitude:** metric trends from configured dashboards and charts
3. **Classifies health** using evidence-based rules:
   - Steady progress + no blockers → **On Track**
   - Blockers, stale issues, low velocity, declining metrics → **At Risk**
   - Missed deadlines, critical blockers, no activity → **Off Track**
4. **Detects silent failures** — initiatives with no updates, no PRs, no ticket movement
5. **Generates** a structured markdown summary where every claim links back to source data
6. **Publishes** to conversation (default), Linear (initiative status updates), Slack (`#pulse-reports`), or all at once

## Report hierarchy

Reports are available at five levels, each progressively more condensed:

```
Company          All BUs aggregated — executive scan in under a minute
  └─ BU          Multiple pods aggregated — pod summary tables + top risks/wins
      └─ Pod     All initiatives for a team — overview table + per-initiative detail
          └─ Initiative   One initiative with its projects — full signal detail
              └─ Project  Single project — issues, PRs, milestones
```

Each level aggregates the level below it. Higher levels strip out granular detail and surface only the most important signals.

## Example prompts

Pulse runs in two environments. Use CLI flags in Claude Code, or natural language in the browser:

| Level | Claude Code CLI | Browser project |
|---|---|---|
| Project | `/pulse "RAM experience v2" --project` | `Summarize project "RAM experience v2"` |
| Initiative | `/pulse [CAA - FY26Q1] Increase mobile activation by 10%` | `Summarize initiative [CAA - FY26Q1] Increase mobile activation by 10%` |
| Pod | `/pulse caa --pod` | `Generate a pod rollup for CAA` |
| BU | `/pulse --bu=cash-advance` | `BU rollup for cash advance` |
| Company | `/pulse --company` | `Company pulse` |
| Monthly | `/pulse caa --pod --monthly` | `Monthly roundup for CAA` |
| Custom period | `/pulse caa --pod --period=7d` | `Pod rollup for CAA for the last 7 days` |

## Output destinations

By default, reports are displayed in the conversation only. You can also publish:

### Linear

Posts a **status update on each initiative** with the appropriate health classification (On Track / At Risk / Off Track). Updates appear in Linear's native initiative update timeline — the same place PMs write manual updates. Each initiative gets its own focused update, not one giant blob.

**CLI:** `/pulse caa --pod --output=linear`
**Browser:** `Generate a pod rollup for CAA and post to Linear`

### Slack

Posts to **`#pulse-reports`** in two parts:
- **Top-level message:** A condensed summary card — status counts, top risks, key wins. Scannable at the channel level.
- **Thread reply:** The full detailed report. One click away, but doesn't clutter the channel.

If the report is long, it splits across multiple thread replies with part indicators (1/3, 2/3, 3/3).

**CLI:** `/pulse caa --pod --output=slack`
**Browser:** `Generate a pod rollup for CAA and post to Slack`

### All at once

Publishes to Linear + Slack + saves to file (CLI only):

**CLI:** `/pulse caa --pod --publish`
**Browser:** `Generate a pod rollup for CAA and publish to both Linear and Slack`

All publishing actions **ask for confirmation** before posting.

### File (CLI only)

Saves the report as markdown to `reports/{team_key}/` with date suffixes. These are gitignored.

**CLI:** `/pulse caa --pod --save`

## What's in a report

A pod-level report includes:

- **Overview table** — active initiatives, on track / at risk / off track counts, cycle velocity
- **Initiative status table** — each initiative with a one-line key signal
- **Top risks** — with evidence links to Linear issues, project updates, or Amplitude charts
- **Key wins** — shipped work, completed milestones, strong velocity
- **Per-initiative detail** — progress, blockers, completed/in-progress issues, merged PRs
- **Metrics impact** — Amplitude trends (e.g., "iOS 14d activation: 60% → 46%, declining ~14pp")
- **Slack signals** — relevant quotes from team discussions, attributed to speakers
- **Cross-team dependencies** — detected from multi-team projects
- **Leadership summary** — what's going well, what needs attention, recommendations

Monthly roundups use the same data with a 30-day window and narrative prose format suitable for presentations.

## Data sources

| Source | Connection | What it provides | Availability |
|---|---|---|---|
| Linear | MCP (claude.ai) | Issues, project status updates, cycle velocity, initiative health | CLI + Browser |
| GitHub | `gh` CLI | PRs, merge activity, review status | CLI only |
| Slack | MCP (claude.ai) | Team discussions, blocker signals, standup updates | CLI + Browser |
| Amplitude | MCP (claude.ai) | Metric trends, funnel conversion rates, experiment results | CLI + Browser |

## Configs

Pod configs define which team, repos, channels, and metrics to include:

| Config | Team | File |
|---|---|---|
| Cash Advance - Activation | CAA | `config/caa.json` |
| Cash Advance - Retention | CSH | `config/csh.json` |
| Card Growth | CARGO | `config/cargo.json` |
| Card Payments & Collections | CPC | `config/cpc.json` |

BU configs aggregate pods:

| BU | Pods | File |
|---|---|---|
| Cash Advance | CAA, CSH | `config/bu/cash-advance.json` |
| Credit Card | CARGO, CPC | `config/bu/credit-card.json` |

Company config (`config/company.json`) aggregates all BUs.

To add your team, use the guided setup or copy `config/example.json`.

## Setup

Pulse AI runs in two environments. Choose the one that fits your workflow — or use both:

| | Claude.ai Browser Project | Claude Code CLI |
|---|---|---|
| **Best for** | Quick reports, non-engineers, no install needed | Full-fidelity reports with GitHub PR data |
| **GitHub PR data** | Not available | Yes |
| **Save to file** | Not available | Yes |
| **Linear + Slack publishing** | Yes | Yes |
| **Setup time** | ~5 minutes | ~10 minutes |
| **Setup guide** | [Browser setup](docs/setup-browser.md) | [CLI setup](docs/setup-cli.md) |

**Quick start:** The browser project is already set up and ready to use at [claude.ai/project/019d6411-a3e6-70f2-b971-a8ac0ea9aca0](https://claude.ai/project/019d6411-a3e6-70f2-b971-a8ac0ea9aca0). Open a chat and try `Generate a pod rollup for CAA`.

## FAQ

**Do I need to change how my team uses Linear?**
No. Pulse reads whatever is already in Linear. No new fields, labels, or processes required.

**Will it post anything without my permission?**
No. Publishing to Linear or Slack always asks for confirmation first. Default output is displayed in your conversation only.

**Can I run this for a team I'm not on?**
Yes, as long as you have read access in Linear.

**What about sensitive information?**
Pulse only reads data you already have access to. It doesn't store anything externally. Reports saved with `--save` are gitignored.

**How does it match PRs to Linear issues?**
By branch naming convention. If your PR branch contains `CAA-1234` or the title contains `LB#CAA-1234`, it gets linked to that issue. (CLI only — GitHub data is not available in browser mode.)

**Where do reports go in Slack?**
To `#pulse-reports`. The top-level message is a condensed summary card. The full report goes in a thread reply so the channel stays scannable.

**Where do reports go in Linear?**
As initiative status updates — the same place PMs write manual updates. Each initiative gets its own update with the correct health classification.

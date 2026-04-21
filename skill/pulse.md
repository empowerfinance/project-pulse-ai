---
name: pulse
description: Generate AI-powered initiative or pod-level status summaries from Linear, GitHub, Slack, and Amplitude data. Use when asked about project health, initiative status, pod updates, or monthly roundups.
argument-hint: "<initiative-name | initiative-url | pod-config-name> [--pod] [--period=14d] [--monthly] [--save] [--publish] [--output=linear|slack|file]"
---

# Project Pulse AI — Initiative & Pod Summary Generator

Generate structured, evidence-backed status summaries by synthesizing signals from Linear, GitHub, Slack, Notion, and Amplitude. Every claim links to source data.

## Environment Detection

This skill works in two environments:

- **Claude Code CLI:** Invoked via `/pulse` with flags. Has filesystem access for configs and report saving. Has `gh` CLI for GitHub PR data.
- **Claude.ai browser project:** Invoked via natural language (e.g., "generate a pod rollup for CAA"). Configs are loaded from project knowledge. No filesystem or `gh` CLI — GitHub PR signals are unavailable and `--save` is not supported.

To detect the environment: if you can execute bash commands (e.g., `which gh`), you are in CLI mode. Otherwise, assume browser mode.

---

## Step 0 — Parse Arguments

**CLI mode:** Parse `$ARGUMENTS` using the flags below.

**Browser mode:** Parse the user's natural language request and map it to the same target and modifier concepts. For example, "monthly roundup for cash advance BU" maps to `--bu=cash-advance --monthly`.

**Target (what to summarize):**

| Token | Level | Meaning |
|---|---|---|
| Linear project name or URL | Project | Summarize a single project |
| Linear initiative name or URL | Initiative | Summarize a single initiative and its projects |
| Pod config name (e.g., `caa`) | Pod | Load pod config, summarize all active initiatives for that team |
| `--pod` | Pod | Explicit pod-level rollup flag (use with config name) |
| `--bu=name` (e.g., `--bu=cash-advance`) | BU | Load BU config from `config/bu/{name}.json`, aggregate all pods in the BU |
| `--company` | Company | Load `config/company.json`, aggregate all BUs |

**Modifiers:**

| Flag | Meaning |
|---|---|
| `--period=Nd` | Lookback window in days (default: 14) |
| `--monthly` | 30-day window, presentation-ready narrative format |
| `--save` | Display + save report to `reports/` as markdown file |
| `--publish` | Display + save + post to Linear + post to #pulse-reports Slack (the "do everything" flag) |
| `--output=linear` | Display + post to Linear status updates only |
| `--output=slack` | Display + post to #pulse-reports Slack only |
| `--output=file` | Display + save to file only (same as `--save`) |

**Report hierarchy:**

```
Company          /pulse --company
  └─ BU          /pulse --bu=cash-advance
      └─ Pod     /pulse caa --pod
          └─ Initiative  /pulse [initiative name]
              └─ Project  /pulse "RAM experience v2" --project
```

Each level aggregates the level below it. Higher levels are progressively more condensed.

If no arguments provided, ask the user what they want to summarize.

---

## Step 1 — Load Config

Load the appropriate config based on the target level:

| Level | Config path |
|---|---|
| Project | No config needed — resolve directly via Linear |
| Initiative | No config needed — resolve directly via Linear. Optionally load pod config if a config name is also provided (for GitHub repos, Slack, Amplitude). |
| Pod | `config/{name}.json` |
| BU | `config/bu/{name}.json` — contains `teams` array referencing pod config names. Load each pod config. |
| Company | `config/company.json` — contains `bus` array referencing BU config names. Load each BU config, which in turn loads pod configs. |

**CLI mode:** Read config files from the project-pulse-ai directory on disk (e.g., `/Users/thomasq/empower/project-pulse-ai/config/{name}.json`).

**Browser mode:** Config files are uploaded as project knowledge. Read the config content from the knowledge files with matching names (e.g., `caa.json`, `company.json`).

For pod configs, extract: `team_key`, `team_id`, `github_repos`, `issue_prefix`, `noise_filters`, `slack_channels`, `amplitude_charts`, `amplitude_dashboards`.

For BU configs, extract: `bu` (name), `teams` (array of pod config names), `slack_channel`.

For company config, extract: `company` (name), `bus` (array of BU config names), `slack_channel`.

---

## Step 2 — Resolve Targets

**Project level (`--project`):** Use `mcp__claude_ai_Linear__get_project` with `includeMilestones: true, includeMembers: true` to get the project. Then gather issues and status updates for that project only.

**Initiative level (default for initiative name/URL):** Use `mcp__claude_ai_Linear__get_initiative` with `includeProjects: true` to get the initiative and its linked projects. Then gather signals for each project.

**Pod level (`--pod` or config name):** Use `mcp__claude_ai_Linear__list_initiatives` filtered by team, status `Active`, with `includeProjects: true`. If no initiatives are found (some teams organize by project only), fall back to `mcp__claude_ai_Linear__list_projects` filtered by team. Also list issues directly for the team.

**BU level (`--bu=name`):** For each team in the BU config, run the pod-level resolution. Collect all pod results.

**Company level (`--company`):** For each BU in the company config, run the BU-level resolution. Collect all BU results.

---

## Step 3 — Gather Signals (per initiative)

For each initiative, gather data from all available sources. Run these in parallel where possible.

### 3a. Linear Signals

For each project under the initiative:

1. **Issues (last N days):** `mcp__claude_ai_Linear__list_issues` with `project` filter and `updatedAt: -P{N}D`
   - Filter out noise: skip issues matching `noise_filters.exclude_labels` or `noise_filters.exclude_title_patterns`
   - Categorize: completed, in-progress, blocked, new
   - Note assignees and status transitions

2. **Project status updates:** `mcp__claude_ai_Linear__get_status_updates` with `type: project` and `project` filter
   - Extract: health, body text, progress diffs
   - These are the most valuable signal — they're human-written context

3. **Cycle data (if team_id available):** `mcp__claude_ai_Linear__list_cycles` with `type: current`
   - Calculate velocity: completed / total scope

### 3b. GitHub Signals (CLI mode only)

> **Browser mode:** Skip this section entirely. GitHub PR data is not available in browser mode. Note in the report: "GitHub PR data not available (browser mode)." Reduce confidence by one level when GitHub signals are missing (High → Medium, Medium → Low).

**CLI mode:** For each repo in the pod config (or inferred from issue branch names):

```bash
gh pr list --repo {repo} --search "{issue_prefix}-" --state all --limit 20 --json number,title,state,createdAt,mergedAt,author,url
```

- Categorize: merged (last N days), open/in-review, stale
- Link PRs to Linear issues via branch naming convention (`{prefix}-XXXX-*` or `LB#{prefix}-XXXX`)

### 3c. Slack Signals

If `slack_channels` is configured in the pod config, gather recent discussion signals.

**Search strategy:**
1. Search the pod channel for initiative/project keywords in the time window:
   ```
   mcp__claude_ai_Slack__slack_search_public with query="{project_name} after:{start_date} in:#{pod_channel}"
   ```
2. Also search for risk/blocker keywords:
   ```
   mcp__claude_ai_Slack__slack_search_public with query="blocker OR blocked OR risk OR delayed OR slipping after:{start_date} in:#{pod_channel}"
   ```
3. If feature-specific channels exist in config, search those too for more targeted context.

**What to extract:**
- Progress updates from standups/async updates (often in threads)
- Blockers or risks mentioned in conversation
- Decisions made that aren't captured in Linear
- Sentiment signals (urgency, concern, celebration)

**Signal weighting:**
- Slack signals supplement but don't override Linear/GitHub data
- Direct statements from the initiative owner or project lead carry more weight
- Standup updates are high-value context (e.g., "frantically finishing X" → urgency signal)
- Skip bot messages and automated alerts unless they indicate a production issue

**Include in output as:**
- Relevant quotes attributed to the speaker
- Folded into the progress or blockers sections with Slack message links where possible

### 3d. Amplitude Signals

If `amplitude_charts` or `amplitude_dashboards` is configured in the pod config, pull metric trends.

**Data gathering strategy:**
1. **If specific charts are configured** (preferred): Query them directly using `mcp__claude_ai_Amplitude__query_chart` with the chart ID from config. This is the fastest and most reliable path.

2. **If no charts configured**: Search for relevant charts:
   ```
   mcp__claude_ai_Amplitude__search with queries=["{initiative keywords}"] entityTypes=["CHART","DASHBOARD"] appIds=[{amplitude_project_id}]
   ```
   Then query the most relevant results.

**How to interpret chart data:**
- For funnel charts (most common): Look at the conversion rate trend over the time window
- Compare beginning vs end of period: is the metric improving, stable, or declining?
- Compare platforms (iOS vs Android) when grouped — different trends may indicate platform-specific issues
- Flag any drops > 5 percentage points as significant

**Include in output as a "Metrics Impact" section:**
```markdown
## Metrics Impact

**14-day CA activation rate** ([chart](url)):
- iOS: {start}% → {end}% ({direction} {delta}pp over {period})
- Android: {start}% → {end}% ({direction} {delta}pp over {period})
- Assessment: {interpretation}
```

**Integration with status classification:**
- Metric trending up → supports "on track"
- Metric stable → neutral
- Metric declining > 5pp → contributes to "at risk" classification
- Metric in steep decline (> 10pp) → strong "at risk" or "off track" signal

---

## Step 4 — Classify Status

Apply these rules to determine initiative health:

| Condition | Status |
|---|---|
| Steady progress, no unresolved blockers, milestones advancing | `on_track` |
| Blockers exist (labeled or mentioned in updates/comments) | `at_risk` |
| No issue progress in 7+ days across the initiative | `at_risk` |
| Cycle velocity < 30% at midpoint | `at_risk` |
| Target date passed with significant incomplete work | `off_track` |
| Critical external blocker with no resolution timeline | `off_track` |
| Key metrics declining (if Amplitude data available) | `at_risk` |
| No activity across any source (silent failure) | `at_risk` |

**Confidence scoring:**
- **High:** Multiple sources corroborate, recent status updates exist, clear signals
- **Medium:** Some sources available, signals are mixed or sparse
- **Low:** Limited data, no recent status updates, ambiguous signals

If a project already has a human-set `health` in its status update, use that as a strong prior and only override if evidence clearly contradicts it.

---

## Step 5 — Generate Summary

### For single initiative (default):

Output a markdown document with this structure:

```markdown
# Initiative Summary: {initiative_name}

> Auto-generated by Project Pulse AI | Period: {date_range} | Confidence: **{High|Medium|Low}**

## Status: {ON TRACK | AT RISK | OFF TRACK}

**Rationale:** {1-2 sentences explaining the status classification with evidence}

## Projects

### 1. {project_name}
| Field | Value |
|---|---|
| Lead | {name} |
| Status | {status} |
| Target | {date} |
| Health | {health} |

**Progress (last {N} days):**
- {bullet points from status updates + issue completions}

**Blockers:**
- {if any, with Linear issue links}

**Completed issues:**
- [{issue_id}]({url}): {title} ({assignee}, Done {date})

**In-progress issues:**
- [{issue_id}]({url}): {title} ({assignee})

**Merged PRs:**
- [#{number}]({url}): {title}

**Open PRs:**
- [#{number}]({url}): {title}

{repeat for each project}

## Metrics Impact

{Include this section if Amplitude data was gathered. Show the key metrics trend with chart links.}

**14-day CA activation rate** ([chart]({amplitude_chart_url})):
- iOS: {start_rate}% → {end_rate}% ({trend} {delta}pp over {period})
- Android: {start_rate}% → {end_rate}% ({trend} {delta}pp over {period})
- Assessment: {e.g., "iOS activation declining ~14pp over 30 days — warrants investigation. Android relatively stable until last week."}

{Add additional metrics sections if other relevant charts were found.}

## Slack Signals

{Include this section if Slack data was gathered. Show relevant team discussion context.}

- {quote or paraphrase from team member} — {author}, #{channel} ({date})
- {additional relevant signals}

## Summary for Leadership

**What's going well:**
- {key wins, shipped work, positive momentum}

**What needs attention:**
- {blockers, risks, slowing velocity}

**Recommendation:** {actionable suggestion if applicable}

---
*Sources: {list sources used}. Generated {date}.*
```

### For pod-level rollup (`--pod`):

```markdown
# Pod Update: {pod_name}

> Auto-generated by Project Pulse AI | Period: {date_range}

## Overview
| Metric | Value |
|---|---|
| Active initiatives | {count} |
| On Track | {count} |
| At Risk | {count} |
| Off Track | {count} |
| Current cycle velocity | {completed}/{total} ({%}) |

## Initiative Status

| Initiative | Status | Key Signal |
|---|---|---|
| {name} | {status emoji + text} | {one-line summary} |

## Top Risks
1. {risk with evidence link}

## Key Wins
1. {win with evidence link}

## Cross-Team Dependencies
- {if detected from multi-team projects}

{then include per-initiative detail sections}
```

### For monthly roundup (`--monthly`):

Use 30-day window. Generate narrative prose suitable for presentation:

```markdown
# Monthly Roundup: {pod_name} — {month}

## Executive Summary
{2-3 sentence narrative of the month}

## Initiative Highlights

### {initiative_name}
**Summary:** {paragraph narrative}
**Impact:** {metrics if available}
**Highlights:**
- {bullet points}
**Risks:**
- {bullet points}
**Next:** {what's coming}

## Key Metrics
{Amplitude data if available}

## Team Wins
{notable completions, shipped features}

## Looking Ahead
{next month focus areas}
```

### For single project (`--project`):

```markdown
# Project Summary: {project_name}

> Auto-generated by Project Pulse AI | Period: {date_range} | Confidence: **{High|Medium|Low}**

## Status: {ON TRACK | AT RISK | OFF TRACK}

| Field | Value |
|---|---|
| Lead | {name} |
| Team | {team_name} |
| Target | {date} |
| Milestones | {completed}/{total} |

**Progress (last {N} days):**
- {from status updates + issue activity}

**Blockers:**
- {if any}

**Completed issues:**
- [{id}]({url}): {title} ({assignee})

**In-progress issues:**
- [{id}]({url}): {title} ({assignee})

**Merged PRs:**
- [#{number}]({url}): {title}

**Open PRs:**
- [#{number}]({url}): {title}

---
*Sources: {list}. Generated {date}.*
```

### For BU rollup (`--bu=name`):

The BU level aggregates multiple pods. It should be **more condensed than pod rollups** — leadership reading a BU view doesn't want per-issue detail.

```markdown
# BU Update: {bu_name}

> Auto-generated by Project Pulse AI | Period: {date_range}

## Overview
| Metric | Value |
|---|---|
| Pods | {count} |
| Active initiatives | {total across all pods} |
| On Track | {count} |
| At Risk | {count} |
| Off Track | {count} |

## Pod Summary

| Pod | On Track | At Risk | Off Track | Key Signal |
|---|---|---|---|---|
| {pod_name} ({key}) | {n} | {n} | {n} | {one-line summary of the pod's state} |

## Top Risks (BU-wide)
1. {risk — identify which pod it comes from}
2. {risk}
3. {risk}

## Key Wins (BU-wide)
1. {win — identify which pod}
2. {win}

## Cross-Pod Dependencies
- {dependencies between pods within this BU, or with external teams}

## Metrics Impact
{aggregate key metrics across pods if available}

## Per-Pod Detail

### {pod_name} ({key})
{condensed version of pod rollup: initiative status table + top risk + top win. Not full issue/PR detail.}

---
*Sources: {list}. Generated {date}.*
```

### For company rollup (`--company`):

The company level aggregates all BUs. This is the **most condensed view** — a single page an executive can scan in under a minute.

```markdown
# Tilt Company Pulse — {date}

> Auto-generated by Project Pulse AI | Period: {date_range}

## Executive Overview
| Metric | Value |
|---|---|
| Business units | {count} |
| Total active initiatives | {count} |
| On Track | {count} ({%}) |
| At Risk | {count} ({%}) |
| Off Track | {count} ({%}) |

## BU Health

| BU | Pods | Initiatives | ✅ | ⚠️ | 🔴 | Top Signal |
|---|---|---|---|---|---|---|
| {bu_name} | {n} | {n} | {n} | {n} | {n} | {one-line} |

## Top 5 Company Risks
1. **{risk}** — {BU / Pod} — {one-line description with evidence link}
2. ...

## Top 5 Company Wins
1. **{win}** — {BU / Pod} — {one-line description}
2. ...

## Cross-BU Dependencies
- {if any detected}

## Per-BU Detail

### {bu_name}
{condensed: pod summary table + top risk + top win. No initiative-level detail.}

---
*Sources: {list}. Generated {date}.*
```

### Condensation principle

Each level up removes one layer of detail:

| Level | Shows detail for | Summarizes |
|---|---|---|
| Project | Issues, PRs, milestones | — |
| Initiative | Projects (with issues/PRs) | — |
| Pod | Initiatives (with project summaries) | Issues/PRs into counts |
| BU | Pods (with initiative tables) | Projects into one-line signals |
| Company | BUs (with pod tables) | Initiatives into counts + top signals |

---

## Step 6 — Output

Always display the full summary in the conversation first. Then handle additional outputs based on flags.

### Output flag summary

| Flag | Display | Save file | Linear | Slack |
|---|---|---|---|---|
| *(none)* | Yes | No | No | No |
| `--save` or `--output=file` | Yes | Yes | No | No |
| `--output=linear` | Yes | No | Yes | No |
| `--output=slack` | Yes | No | No | Yes |
| `--publish` | Yes | Yes | Yes | Yes |

### 6a. Save to file (`--save`, `--output=file`, or `--publish`)

> **Browser mode:** File saving is not available. If the user requests `--save`, let them know: "File saving is not available in browser mode. The report is displayed above. You can copy it manually, or use `--output=slack` / `--output=linear` to publish to those platforms." For `--publish`, proceed with the Linear and Slack steps but skip the file save.

**CLI mode:** Save the report to `reports/` using the directory and naming conventions below.

**Directory structure:**

| Level | Directory |
|---|---|
| Project | `reports/{team_key}/` |
| Initiative | `reports/{team_key}/` |
| Pod | `reports/{team_key}/` |
| BU | `reports/bu/{bu-slug}/` |
| Company | `reports/` |

**Naming conventions:**

| Mode | Filename |
|---|---|
| Single project (`--project`) | `project-summary-{slugified-name}-{YYYY-MM-DD}.md` |
| Single initiative | `initiative-summary-{slugified-name}-{YYYY-MM-DD}.md` |
| Pod rollup (`--pod`) | `pod-rollup-{YYYY-MM-DD}.md` |
| Pod monthly (`--monthly`) | `monthly-roundup-{YYYY-MM}.md` |
| BU rollup (`--bu`) | `bu-rollup-{YYYY-MM-DD}.md` |
| BU monthly (`--bu --monthly`) | `bu-monthly-{YYYY-MM}.md` |
| Company (`--company`) | `company-pulse-{YYYY-MM-DD}.md` |
| Company monthly (`--company --monthly`) | `company-monthly-{YYYY-MM}.md` |

Create directories as needed. Overwrite if same filename exists (re-running same day = update, not duplicate).

After saving, note: `Saved to: reports/{path}/{filename}`

### 6b. Post to Linear (`--output=linear` or `--publish`)

**Before posting, always confirm with the user:** "Ready to post status updates to Linear for {N} initiatives? (y/n)"

**For pod rollups (`--pod`) and monthly roundups (`--monthly`):**

Post a separate status update on each initiative using `mcp__claude_ai_Linear__save_status_update`:

```
For each initiative:
  mcp__claude_ai_Linear__save_status_update(
    type: "initiative",
    initiative: "{initiative_id}",
    health: "{onTrack|atRisk|offTrack}",
    body: "{that initiative's summary section from the report}"
  )
```

The `body` for each initiative should include:
- The progress summary for that initiative
- Key blockers (if any)
- Evidence links
- A note: `*Auto-generated by Pulse AI — {date}*`

Do NOT post the entire pod rollup as one giant update. Each initiative gets its own focused update with the appropriate health status.

**For single initiative summaries:**

Post one status update on the initiative:

```
mcp__claude_ai_Linear__save_status_update(
  type: "initiative",
  initiative: "{initiative_id}",
  health: "{onTrack|atRisk|offTrack}",
  body: "{the Summary for Leadership section + key blockers + evidence links}"
)
```

If the initiative has no ID (team uses projects instead of initiatives), post at the project level instead:

```
mcp__claude_ai_Linear__save_status_update(
  type: "project",
  project: "{project_id}",
  health: "{onTrack|atRisk|offTrack}",
  body: "{project summary}"
)
```

After posting, report: `Posted {N} status updates to Linear.`

### 6c. Post to Slack (`--output=slack` or `--publish`)

**Before posting, always confirm with the user:** "Ready to post to #pulse-reports? (y/n)"

**Step 1: Find or confirm the channel.**

Search for the `#pulse-reports` channel:
```
mcp__claude_ai_Slack__slack_search_channels(query: "pulse-reports")
```

If found, use its channel ID. If not found, tell the user: "The #pulse-reports channel doesn't exist yet. Please create it and re-run, or specify a channel: `--slack-channel=#your-channel`"

**Step 2: Post the top-level summary message.**

Compose a condensed card (not the full report). Format:

For pod rollups:
```
📊 Pod Rollup: {pod_name} ({team_key}) — {date}

On Track: {n}  |  At Risk: {n}  |  Off Track: {n}
Cycle {n}: {completed}/{total} ({%})

Top risks:
• {risk 1}
• {risk 2}

Key wins:
• {win 1}
• {win 2}

Full report in thread ↓
```

For single initiative:
```
📋 Initiative: {initiative_name} — {status_emoji} {STATUS}

{1-2 sentence rationale}

Full report in thread ↓
```

For monthly roundups:
```
📅 Monthly Roundup: {pod_name} — {month}

{executive_summary — 2-3 sentences}

On Track: {n}  |  At Risk: {n}  |  Off Track: {n}

Full report in thread ↓
```

Status emojis: On Track = ✅, At Risk = ⚠️, Off Track = 🔴

Post via:
```
mcp__claude_ai_Slack__slack_send_message(
  channel_id: "{pulse_reports_channel_id}",
  message: "{condensed card}"
)
```

**Step 3: Post the full report as a thread reply.**

Take the `ts` (timestamp) from the top-level message response and post the full markdown report as a thread reply:

```
mcp__claude_ai_Slack__slack_send_message(
  channel_id: "{pulse_reports_channel_id}",
  thread_ts: "{top_level_message_ts}",
  message: "{full markdown report}"
)
```

Note: Slack has a 5000 character limit per message. If the full report exceeds this, split it into multiple thread replies at natural section boundaries (e.g., one reply per project section). Add a part indicator: `(1/3)`, `(2/3)`, `(3/3)`.

**Step 4: Post Linear links as a final thread reply (if Linear was also posted to).**

If `--publish` was used (both Linear and Slack):
```
mcp__claude_ai_Slack__slack_send_message(
  channel_id: "{pulse_reports_channel_id}",
  thread_ts: "{top_level_message_ts}",
  message: "Linear updates posted:\n• {initiative_name} → {status} ({linear_update_url})\n..."
)
```

After posting, report: `Posted to #pulse-reports (top-level + {N} thread replies).`

---

## Important Guidelines

1. **Never fabricate data.** If a source isn't available or returns no results, say so. Reduce confidence accordingly.
2. **Always include evidence links.** Every factual claim should link to a Linear issue, PR, project update, or Slack message.
3. **Filter noise.** Exclude auto-generated issues (SweeperAI), bot PRs, and stale items unless they're the only signal.
4. **Respect human judgment.** If a PM has written a status update, quote or incorporate it rather than overriding it.
5. **Be concise for leadership, detailed for ICs.** The "Summary for Leadership" section should be scannable in 30 seconds. The full detail is for people who want to drill down.
6. **Status classification should be conservative.** When in doubt between on_track and at_risk, choose at_risk. Missing a risk is worse than over-flagging.
7. **Note missing sources.** In browser mode, always include in the Sources line: "GitHub PR data not available (browser mode)." Omit the Merged PRs and Open PRs subsections from project/initiative detail when GitHub data is unavailable.

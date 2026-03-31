# Project Pulse AI

Automated, multi-level status summaries powered by AI. Synthesizes signals from Linear, GitHub, Slack, and Amplitude into structured, evidence-backed updates — from individual projects up to company-wide executive views.

## What it does

- Generates **status reports at every level**: project, initiative, pod, BU, and company
- Classifies health as On Track / At Risk / Off Track with evidence
- Links every claim to a source: Linear issue, GitHub PR, project update, Amplitude chart
- Detects **silent failures** — initiatives with no updates, no PRs, no movement
- Publishes to **Linear** (initiative status updates) and **Slack** (`#pulse-reports` with thread detail)
- Supports **monthly roundup** narratives for leadership presentations

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed
- The following Claude.ai MCP connectors authenticated (see [Setup](#setup) below):
  - **Linear** (required)
  - **GitHub** via `gh` CLI (required)
  - **Slack** (optional — enhances summaries with team discussion signals, required for `--output=slack` and `--publish`)
  - **Amplitude** (optional — adds product metric trends to reports)

## Setup

### 1. Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

### 2. Clone this repo

```bash
git clone <repo-url> project-pulse-ai
cd project-pulse-ai
```

### 3. Copy the skills into your Claude commands directory

```bash
mkdir -p ~/.claude/commands
cp skill/pulse.md ~/.claude/commands/pulse.md
cp skill/pulse-setup.md ~/.claude/commands/pulse-setup.md
```

Or, if you want it scoped to a specific project directory:

```bash
mkdir -p /path/to/your/project/.claude/commands
cp skill/pulse.md /path/to/your/project/.claude/commands/pulse.md
cp skill/pulse-setup.md /path/to/your/project/.claude/commands/pulse-setup.md
```

### 4. Run guided setup

The easiest way to get going is with the setup skill. After copying the skills, restart Claude Code and run:

```
/pulse-setup
```

This will:
- Check all MCP connections (Linear, GitHub, Slack, Notion, Amplitude)
- Help you create a pod config for your team (or choose an existing one)
- Verify everything is ready

You can also set things up manually by following steps 5-6 below.

### 5. Authenticate MCP connectors

MCP connectors are managed through [claude.ai/settings/integrations](https://claude.ai/settings/integrations). Each connector needs to be authenticated once — after that it works across all Claude Code sessions.

**Required:**

| Connector | How to authenticate |
|---|---|
| **Linear** | Go to claude.ai Settings → Integrations → Linear → Connect. Sign in with your Tilt Linear account. |
| **GitHub** | Install the `gh` CLI and run `gh auth login`. Follow the prompts to authenticate with your GitHub account. |

**Optional (but recommended):**

| Connector | How to authenticate |
|---|---|
| **Slack** | claude.ai Settings → Integrations → Slack → Connect. Sign in with your Tilt Slack workspace. |
| **Amplitude** | claude.ai Settings → Integrations → Amplitude → Connect. Sign in with your Tilt Amplitude account. |

**Verify your connections:**

```bash
claude mcp list
```

You should see `✓ Connected` for Linear and any optional connectors you set up.

### 6. Create your pod config (manual alternative to /pulse-setup)

Copy the example config and fill in your team's details:

```bash
cp config/example.json config/your-team.json
```

Edit `config/your-team.json`:

```json
{
  "pod": "Your Pod Name",
  "team_key": "YOUR_KEY",
  "team_id": "your-linear-team-uuid",
  "github_repos": [
    "empowerfinance/your-repo-1",
    "empowerfinance/your-repo-2"
  ],
  "amplitude_project_id": "152808",
  "issue_prefix": "YOUR_KEY",
  "noise_filters": {
    "exclude_labels": ["SweeperAI"],
    "exclude_title_patterns": ["SweeperAI Summary"]
  }
}
```

**How to find your values:**

| Field | Where to find it |
|---|---|
| `team_key` | Your Linear team's short prefix (e.g., `CAA`, `GRW`). Visible in issue IDs like `CAA-1234`. |
| `team_id` | Open Claude Code and run: `/pulse` then ask it to look up your team. Or find it in the Linear URL when viewing your team. |
| `github_repos` | The GitHub repos your team contributes to. Format: `org/repo-name`. |
| `amplitude_project_id` | `152808` for Empower production. Use `154226` for dev. |
| `issue_prefix` | Usually same as `team_key`. Used to match GitHub PRs to Linear issues. |
| `noise_filters` | Labels or title patterns to exclude from summaries (e.g., automated bot issues). |

## Usage

Start Claude Code in any directory that has the pulse skill available:

```bash
claude
```

### Report hierarchy

```
Company          /pulse --company
  └─ BU          /pulse --bu=cash-advance
      └─ Pod     /pulse caa --pod
          └─ Initiative  /pulse [initiative name]
              └─ Project  /pulse "RAM experience v2" --project
```

Each level aggregates the level below. Higher levels are progressively more condensed.

### Examples by level

```bash
# Project — single project detail
/pulse "RAM experience v2" --project

# Initiative — one initiative with its projects
/pulse [CAA - FY26Q1] Increase mobile 14d register -> CA activation by 10%

# Pod — all initiatives for a team
/pulse caa --pod

# BU — aggregate multiple pods
/pulse --bu=cash-advance

# Company — all BUs
/pulse --company

# Monthly variants (30-day window, narrative format)
/pulse caa --pod --monthly
/pulse --bu=cash-advance --monthly
/pulse --company --monthly

# Custom time period
/pulse caa --pod --period=7d
```

### Output options

By default, `/pulse` displays the report in the conversation only. Use flags to save or publish:

```
/pulse caa --pod --save              # Display + save to reports/caa/
/pulse caa --pod --output=linear     # Display + post status updates to Linear
/pulse caa --pod --output=slack      # Display + post to #pulse-reports Slack
/pulse caa --pod --publish           # Display + save + Linear + Slack (all at once)
```

| Flag | Display | File | Linear | Slack |
|---|---|---|---|---|
| *(none)* | Yes | | | |
| `--save` | Yes | Yes | | |
| `--output=linear` | Yes | | Yes | |
| `--output=slack` | Yes | | | Yes |
| `--publish` | Yes | Yes | Yes | Yes |

**Linear output:** Posts a status update on each initiative with health classification (On Track / At Risk / Off Track). Shows up in Linear's native initiative update timeline.

**Slack output:** Posts a condensed summary card to `#pulse-reports` with the full report in a thread reply. Scannable at the channel level, detailed in the thread.

All publishing actions ask for confirmation before posting.

## Configs

### Pod configs (`config/`)

| Config | Team | File |
|---|---|---|
| Cash Advance - Activation | CAA | `config/caa.json` |
| Cash Advance - Retention | CSH | `config/csh.json` |
| Card Growth | CARGO | `config/cargo.json` |
| Card Payments & Collections | CPC | `config/cpc.json` |

To add your team, run `/pulse-setup` or copy `config/example.json`.

### BU configs (`config/bu/`)

BU configs aggregate multiple pod configs:

| BU | Pods | File |
|---|---|---|
| Cash Advance | CAA, CSH | `config/bu/cash-advance.json` |
| Credit Card | CARGO, CPC | `config/bu/credit-card.json` |

```json
{
  "bu": "Cash Advance",
  "teams": ["caa", "csh"],
  "slack_channel": "#pulse-reports"
}
```

To add a BU, copy `config/bu/example.json` and list the pod config names in `teams`.

### Company config (`config/company.json`)

Aggregates all BUs:

```json
{
  "company": "Tilt",
  "bus": ["cash-advance", "credit-card"],
  "slack_channel": "#pulse-reports"
}
```

Add new BUs to the `bus` array as they're configured.

## How it works

1. **Resolves** the target at the requested level (project → initiative → pod → BU → company)
2. **Gathers signals** in parallel from:
   - Linear: issues, project status updates, cycle velocity
   - GitHub: PRs (merged, open, stale) matched via branch naming convention
   - Slack: keyword searches in relevant channels (if configured)
   - Amplitude: metric trends from dashboards/charts (if configured)
3. **Classifies health** using rules:
   - Steady progress + no blockers → On Track
   - Blockers, stale issues, low velocity, declining metrics → At Risk
   - Missed deadlines, critical blockers, no activity → Off Track
4. **Generates** a structured markdown summary with evidence links, progressively condensed at higher levels
5. **Outputs** to conversation (default), file (`--save`), Linear (`--output=linear`), Slack (`--output=slack`), or all (`--publish`)

## Example Output

Reports are published to **Linear** (initiative status updates) and **Slack** (`#pulse-reports`) using the `--publish` flag. For local reference, `--save` writes reports to `reports/{team_key}/` with date suffixes — these are gitignored and not committed to the repo.

## FAQ

**Q: Do I need to change how my team uses Linear?**
No. Pulse reads whatever is already in Linear. No new fields, labels, or processes required.

**Q: What if my team doesn't use GitHub?**
GitHub is optional. The summary will note that no PR data was available and reduce confidence accordingly.

**Q: How does it match PRs to Linear issues?**
By branch naming convention. If your PR branch contains `CAA-1234` or the title contains `LB#CAA-1234`, it gets linked to that issue.

**Q: Can I run this for a team I'm not on?**
Yes, as long as you have read access in Linear. The tool uses your authenticated Linear account.

**Q: Will it post anything without my permission?**
No. The `--output=linear`, `--output=slack`, and `--publish` flags all ask for confirmation before posting. Default output is markdown displayed in your terminal.

**Q: Where do reports go in Slack?**
To `#pulse-reports`. The top-level message is a condensed summary card (status counts, top risks, wins). The full report goes in a thread reply so the channel stays scannable.

**Q: Where do reports go in Linear?**
As initiative status updates — the same place PMs write manual updates. Each initiative gets its own update with the correct health classification (On Track / At Risk / Off Track).

**Q: What about sensitive information?**
Pulse only reads data you already have access to. It doesn't store anything externally. Summaries are generated in your local Claude Code session. Reports saved with `--save` are gitignored.

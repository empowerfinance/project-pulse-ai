# CLI Setup (Claude Code)

Run Pulse AI from the terminal with full-fidelity reports including GitHub PR data.

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed
- **Linear** MCP integration authenticated (required)
- **GitHub** via `gh` CLI authenticated (required)
- **Slack** MCP integration authenticated (optional — enhances summaries with team discussion signals, required for `--output=slack`)
- **Amplitude** MCP integration authenticated (optional — adds product metric trends)

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

### 4. Authenticate MCP connectors

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

### 5. Run guided setup

The easiest way to configure your team is with the setup skill. Restart Claude Code and run:

```
/pulse-setup
```

This will:
- Check all MCP connections (Linear, GitHub, Slack, Amplitude)
- Help you create a pod config for your team (or choose an existing one)
- Verify everything is ready

### 6. Create your pod config (manual alternative)

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
| `team_id` | Run `/pulse-setup` and it will look it up, or find it in the Linear URL when viewing your team. |
| `github_repos` | The GitHub repos your team contributes to. Format: `org/repo-name`. |
| `amplitude_project_id` | `152808` for Empower production. Use `154226` for dev. |
| `issue_prefix` | Usually same as `team_key`. Used to match GitHub PRs to Linear issues. |
| `noise_filters` | Labels or title patterns to exclude from summaries (e.g., automated bot issues). |

## Usage

Start Claude Code in any directory that has the pulse skill available:

```bash
claude
```

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

### Output flags

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

All publishing actions ask for confirmation before posting.

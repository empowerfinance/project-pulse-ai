---
name: pulse-setup
description: Guided setup for Project Pulse AI — checks MCP connections, helps configure a pod, and verifies the skill is ready to use.
---

## Pulse AI Setup

You are guiding the user through setting up Project Pulse AI. Walk through each step interactively, checking what's already done and skipping completed steps. Be concise — report pass/fail for each check and only stop to ask questions when input is needed.

### Step 0 — Detect Environment

Determine which environment you're running in:

- **Claude Code CLI:** You can execute bash commands. Proceed with the full setup (Steps 1–4).
- **Claude.ai browser project:** You cannot execute bash commands. Use the browser setup flow (Steps 1B, 3B, 4B) and skip Steps 2 and the GitHub check.

### Step 1 — Check MCP Connections (CLI mode)

These are the data sources Pulse AI reads from. Check each one and report status.

**Required:**

1. **Linear** — Try calling `mcp__claude_ai_Linear__get_authenticated_user`. If it returns a user → PASS. If it fails → FAIL.
2. **GitHub CLI** — Run `gh auth status 2>&1 | head -3`. If authenticated → PASS. If not → FAIL.

**Optional (enhance summary quality):**

3. **Slack** — Try calling `mcp__claude_ai_Slack__slack_search_channels` with query `general` limit 1. If it returns results → PASS. If it fails → FAIL.
4. **Notion** — Try calling `mcp__claude_ai_Notion__notion-search` with query `test` and empty filters. If it returns anything (even no results) without error → PASS. If it errors → FAIL.
5. **Amplitude** — Try calling `mcp__claude_ai_Amplitude__get_context`. If it returns org/projects → PASS. If it fails → FAIL.

Report results as a table:

```
MCP Connection Status:
  Linear:      ✓ Connected (as {user_name})
  GitHub CLI:  ✓ Authenticated (as {github_user})
  Slack:       ✓ Connected / ✗ Not connected
  Notion:      ✓ Connected / ✗ Not connected
  Amplitude:   ✓ Connected / ✗ Not connected
```

If Linear or GitHub fail, stop and help the user fix them:
- **Linear:** "Go to claude.ai → Settings → Integrations → Linear → Connect"
- **GitHub:** "Run `gh auth login` in your terminal"

If optional connections fail, note them but continue: "Slack/Notion/Amplitude are optional. Summaries will work with just Linear + GitHub, but adding these enriches the output."

### Step 1B — Check MCP Connections (Browser mode)

Same checks as Step 1, but skip the GitHub CLI check. Replace it with:

2. **GitHub:** ℹ️ Not available in browser mode. Reports will use Linear, Slack, and Amplitude signals only. GitHub PR data is a CLI-only feature.

Report results as:

```
MCP Connection Status:
  Linear:      ✓ Connected (as {user_name})
  GitHub:      ℹ️ Not available (browser mode — CLI only)
  Slack:       ✓ Connected / ✗ Not connected
  Notion:      ✓ Connected / ✗ Not connected
  Amplitude:   ✓ Connected / ✗ Not connected
```

If Linear fails, stop and help: "Go to claude.ai → Settings → Integrations → Linear → Connect"

Then skip to Step 3B.

### Step 2 — Check Pulse Skill Installation (CLI mode only)

> **Browser mode:** Skip this step. In a browser project, the instructions are already loaded as project instructions.

Verify the `/pulse` skill is available by checking if the file exists:

```bash
ls -la ~/.claude/commands/pulse.md 2>/dev/null || ls -la .claude/commands/pulse.md 2>/dev/null
```

If found → PASS.

If not found, check if the repo is available:
```bash
ls -la /Users/thomasq/empower/project-pulse-ai/skill/pulse.md 2>/dev/null
```

If the skill file exists in the repo but not in commands:
- Ask: "The pulse skill isn't installed yet. Want me to copy it to your commands directory?"
- If yes: `cp /Users/thomasq/empower/project-pulse-ai/skill/pulse.md ~/.claude/commands/pulse.md`
- Note: "You'll need to restart Claude Code for the new skill to be available."

If the skill file doesn't exist anywhere:
- "The pulse skill file wasn't found. Make sure the project-pulse-ai repo is cloned."

### Step 3 — Pod Configuration

List existing pod configs:

```bash
ls /Users/thomasq/empower/project-pulse-ai/config/*.json 2>/dev/null | grep -v example
```

Show the user what's available:

```
Available pod configs:
  caa.json    — Cash Advance - Activation
  subex.json  — Subscription Experience
  cargo.json  — Card Growth
  cpc.json    — Card Payments & Collections

Would you like to:
  1. Use an existing config (you're all set!)
  2. Create a new config for your team
```

**If the user wants to create a new config:**

Ask for the following information, one at a time:

1. **Team name:** "What's your Linear team name? (e.g., 'Card Growth')"
   - Then look it up: call `mcp__claude_ai_Linear__get_team` with the name
   - Extract the `team_id` and `key` automatically
   - Confirm with user: "Found team '{name}' with key '{key}'. Is that right?"

2. **GitHub repos:** "Which GitHub repos does your team contribute to? Common ones at Tilt:"
   - `empowerfinance/empower-android`
   - `empowerfinance/empower-ios`
   - `empowerfinance/empower-app`
   - `empowerfinance/tilt-us-monorepo`
   - "Enter repo names separated by commas, or press enter for the defaults above."

3. **Slack channels (optional):** "Does your team have a pod Slack channel? (e.g., #pod-cash-advance)"
   - If yes, search for it: `mcp__claude_ai_Slack__slack_search_channels` with the channel name
   - Extract channel ID automatically
   - Ask about feature channels: "Any feature-specific channels? (comma-separated, or skip)"

4. **Amplitude charts (optional):** "Do you have key Amplitude dashboards or charts to include? (paste URLs or names, or skip)"
   - If URLs provided, extract chart/dashboard IDs from the URL pattern
   - If names provided, search via `mcp__claude_ai_Amplitude__search`

Write the config file:

```bash
# Save to /Users/thomasq/empower/project-pulse-ai/config/{team_key_lowercase}.json
```

Use this template:
```json
{
  "pod": "{team_name}",
  "team_key": "{team_key}",
  "team_id": "{team_id}",
  "github_repos": ["{repos}"],
  "amplitude_project_id": "152808",
  "issue_prefix": "{team_key}",
  "noise_filters": {
    "exclude_labels": ["SweeperAI"],
    "exclude_title_patterns": ["SweeperAI Summary"]
  }
}
```

Add `slack_channels` and `amplitude_charts`/`amplitude_dashboards` sections if the user provided them.

### Step 3B — Pod Configuration (Browser mode)

Same interactive flow as Step 3 for gathering team information (team name, Slack channels, Amplitude charts). However, instead of writing the config to disk:

1. Generate the JSON config in the conversation.
2. Tell the user: "Copy the JSON below and save it as `{team_key_lowercase}.json`. Then upload it to this project's Knowledge files so I can access it in future conversations."
3. If the user already has config files uploaded as project knowledge, list them and offer to use one.

### Step 4 — Verify Setup (CLI mode)

Run a quick validation:

1. Load the pod config file and confirm it parses as valid JSON
2. Call `mcp__claude_ai_Linear__get_team` with the configured team key to confirm it resolves
3. Check that at least one GitHub repo is accessible: `gh repo view {first_repo} --json name 2>/dev/null`

Print a summary:

**CLI mode:**
```
Pulse AI Setup Complete!

  MCP Connections:
    Linear:     ✓ {user_name}
    GitHub:     ✓ {github_user}
    Slack:      ✓ / ✗
    Notion:     ✓ / ✗
    Amplitude:  ✓ / ✗

  Pod Config:   {config_file_path}
  Team:         {team_name} ({team_key})
  Repos:        {repo_list}
  Slack:        {channels or "not configured"}
  Amplitude:    {charts or "not configured"}

To generate summaries:
  /pulse {team_key_lowercase} --pod          Pod-level rollup
  /pulse {initiative_name}                   Single initiative summary
  /pulse {team_key_lowercase} --pod --monthly   Monthly roundup

For more details, see the README in project-pulse-ai/.
```

### Step 4B — Verify Setup (Browser mode)

1. Load the pod config from project knowledge and confirm it parses as valid JSON
2. Call `mcp__claude_ai_Linear__get_team` with the configured team key to confirm it resolves
3. Skip GitHub repo check (not available)

Print a summary:

```
Pulse AI Setup Complete! (Browser mode)

  MCP Connections:
    Linear:     ✓ {user_name}
    GitHub:     ℹ️ Not available (browser mode)
    Slack:      ✓ / ✗
    Notion:     ✓ / ✗
    Amplitude:  ✓ / ✗

  Pod Config:   {config_filename} (project knowledge)
  Team:         {team_name} ({team_key})
  Slack:        {channels or "not configured"}
  Amplitude:    {charts or "not configured"}

To generate summaries, just ask:
  "Generate a pod rollup for {team_key_lowercase}"
  "Summarize initiative {initiative_name}"
  "Monthly roundup for {team_key_lowercase}"

Note: GitHub PR data is not available in browser mode. Reports use Linear, Slack, and Amplitude signals.
```

### Important Notes

- Do NOT attempt to create or modify MCP server configurations. MCP connections are managed through claude.ai Settings → Integrations.
- If a connection check fails due to authentication, direct the user to claude.ai settings — do not try to authenticate programmatically.
- The Amplitude project ID `152808` is the Empower production project. Use `154226` for dev.
- Keep the setup conversational and fast. Don't over-explain — most users just want to get to running `/pulse`.

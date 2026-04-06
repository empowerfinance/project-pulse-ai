# Browser Setup (Claude.ai Project)

Run Pulse AI entirely in a Claude.ai browser project — no CLI, no terminal, no installation required.

## Prerequisites

- A [claude.ai](https://claude.ai) account with access to Projects
- **Linear** MCP integration authenticated (required)
- **Slack** MCP integration authenticated (optional — needed for Slack signals and posting to `#pulse-reports`)
- **Amplitude** MCP integration authenticated (optional — needed for metric trends)

Authenticate integrations at [claude.ai/settings/integrations](https://claude.ai/settings/integrations).

## Setup

### 1. Create a new project

Go to [claude.ai/projects](https://claude.ai/projects) and click **+ New project**.

- **Name:** Project Pulse AI
- **Description:** Automated, multi-level status summaries powered by AI. Synthesizes signals from Linear, Slack, and Amplitude into structured, evidence-backed updates.
- **Visibility:** Your choice (Private or Tilt org-wide)

### 2. Set project instructions

Open `browser/PROJECT_INSTRUCTIONS.md` from this repo, copy the entire contents, and paste it into the project's **Instructions** field (click the "+" or edit icon next to "Instructions" in the right panel).

### 3. Upload config files as Project Knowledge

Click the "+" next to **Files** in the right panel, then choose **Add text content** for each config file.

Upload the configs relevant to your teams:

**Pod configs (required — at least one):**
- `config/caa.json` — Cash Advance - Activation
- `config/csh.json` — Cash Advance - Retention
- `config/cargo.json` — Card Growth
- `config/cpc.json` — Card Payments & Collections

**BU configs (optional — for BU-level reports):**
- `config/bu/cash-advance.json`
- `config/bu/credit-card.json`

**Company config (optional — for company-level reports):**
- `config/company.json`

**Templates (optional — for creating new team configs):**
- `config/example.json`
- `config/bu/example.json`

For each file: set the **Title** to the filename (e.g., `caa.json`) and paste the file contents into **Content**.

### 4. Start using it

Open a conversation in your project and use natural language:

```
Generate a pod rollup for CAA
```
```
Summarize initiative [CAA - FY26Q1] Increase mobile 14d register -> CA activation by 10%
```
```
Monthly roundup for cash advance BU
```
```
Company pulse
```

To publish:
```
Generate a pod rollup for CAA and post to Linear
Generate a pod rollup for CAA and post to Slack
Generate a pod rollup for CAA and publish to both Linear and Slack
```

## Browser mode differences

| Feature | Browser | CLI |
|---|---|---|
| GitHub PR data | Not available | Yes (`gh` CLI) |
| Save to file (`--save`) | Not available | Yes |
| Post to Linear | Yes | Yes |
| Post to Slack | Yes | Yes |
| Invocation style | Natural language | `/pulse` with flags |
| Config source | Project knowledge | Files on disk |
| Confidence | Capped at Medium without GitHub data | Full (High possible) |

## Adding a new team

1. Open a conversation in the project and ask: "Help me set up a new pod config for my team"
2. Claude will walk you through gathering your Linear team ID, Slack channels, and Amplitude charts
3. Copy the generated JSON and add it as a new knowledge file with **Add text content**

# Project Pulse AI — Loom Demo Outline

Target length: ~10-15 minutes

---

## 1. The Problem (~2-3 min)

**Opening hook:** "Of our 45+ active initiatives across the org, only 8 have a health status set in Linear. Multiple active projects have zero status updates in 30+ days — despite active issue and PR work happening. Leadership is flying blind."

**Key points to hit:**

- Status updates are fragmented across Linear, GitHub, Slack, and Amplitude — no single view exists
- PMs and EMs spend significant time each sprint manually compiling updates from these sources
- The output is inconsistent across pods, often stale by the time it reaches leadership, and frequently missing entirely
- Monthly roundups require PMs to manually translate raw data into presentation-ready narratives — duplicated effort across every pod
- Blockers and risks surface late or informally (e.g., buried in Slack threads) — reducing ability to respond quickly
- The evidence table from the Notion page reinforces this:
  - Only 8/45 initiatives have health status set (quantitative)
  - Multiple projects with 0 status updates in 30+ days (quantitative)
  - Manual aggregation takes significant PM/EM time each sprint (qualitative)

**Who we're solving for:**
- Primary: PMs who own initiative updates and monthly roundups (heaviest time savings)
- Secondary: BU and executive leadership who need visibility without reading every initiative

---

## 2. What We Built (~5-7 min)

**High-level:** "Project Pulse AI synthesizes signals from Linear, Slack, and Amplitude into structured, evidence-backed status reports — from a single project all the way up to a company-wide executive view."

### The build journey

**Started with Claude Code CLI:**
- Built a Claude Code "skill" — a reusable prompt template that knows how to gather data, classify health, and generate reports
- The skill (`/pulse`) reads from MCP-connected data sources: Linear, Slack, Amplitude, and GitHub via `gh` CLI
- Config-driven: each pod is a JSON config with team key, Slack channels, Amplitude charts — no code changes needed to add a team

**MCP connections are the key enabler:**
- Model Context Protocol lets Claude natively read from Linear, Slack, Amplitude — no API wrappers, no middleware
- Show the data sources: Linear (issues, project updates, cycle velocity), Slack (team discussions, blocker signals), Amplitude (metric trends)
- GitHub via `gh` CLI for PR data (CLI only)

**Then brought it to the browser:**
- Created a Claude.ai Project with the same instructions and config files as project knowledge
- Anyone can use it — no CLI, no installation, just open the project and ask in natural language

### Live demo (browser project)

**Show the Claude project:** Open [the project](https://claude.ai/project/019d6411-a3e6-70f2-b971-a8ac0ea9aca0) and walk through:
- The instructions (briefly — "this is the skill that tells Claude how to gather data and generate reports")
- The config files uploaded as knowledge ("each pod has a config — team key, Slack channels, Amplitude charts")

**Run a pod rollup:** Type `Generate a pod rollup for CAA` and let it run. While it's working, narrate:
- "It's now querying Linear for all active initiatives on the CAA team..."
- "Pulling recent issues, project status updates, cycle velocity..."
- "Searching Slack for blocker signals and team discussions..."
- "Querying Amplitude for the 14-day activation metric trend..."

**Walk through the output:**
- Overview table: on track / at risk / off track counts
- Initiative status table with one-line key signals
- Top risks with evidence links (click one to show it goes to the actual Linear issue)
- Key wins
- Metrics impact section showing Amplitude trends
- Slack signals with attributed quotes
- Silent failure detection — "these two initiatives had no activity in 14 days, flagged automatically"

**Show the report hierarchy:** Briefly mention:
- "This same system works at every level — project, initiative, pod, BU, company"
- "Each level up gets more condensed — a company pulse is a one-page exec scan"

**Show publishing (optional — if time permits):**
- "I can also say 'post to Linear' and it will create initiative status updates with the correct health classification"
- "Or 'post to Slack' and it posts a summary card to #pulse-reports with the full report in a thread"

---

## 3. Next Steps (~2-3 min)

### Enhance the product
- **Scheduled automation:** Run Pulse on a weekly cron (Claude Code supports scheduled triggers) — auto-generate pod rollups every Monday morning, post to Slack
- **Drift detection:** Week-over-week comparison — "this initiative was On Track last week but is now At Risk, here's what changed"
- **Dependency graph:** Cross-pod dependency detection — surface when one pod's blocker affects another pod's initiative
- **Richer Amplitude integration:** Auto-discover relevant charts rather than requiring manual config mapping
- **Notion integration:** Pull specs and meeting notes for additional context

### Drive adoption
- **Zero-friction onboarding:** Any team can start with just a Linear team key — run `/pulse-setup` or ask in the browser project
- **Share the browser project org-wide:** Change visibility to Tilt so anyone can open it and try
- **Pilot with 2-3 more pods** to validate generalizability beyond Cash Advance / Credit Card
- **Get PM feedback:** "Would you share this as-is, or does it need heavy editing?" — iterate on the output quality

### Automate
- **Weekly auto-publish:** Scheduled pod rollups posted to #pulse-reports every Monday
- **Monthly roundup generation:** Trigger on the last Friday of each month for presentation prep
- **Linear status updates on autopilot:** Auto-set initiative health classifications weekly, with human review opt-in

---

## 4. Lessons Learned (~1-2 min)

**AI's strength is cross-source synthesis:**
- The real value isn't in any single data source — it's in connecting Linear issues to Slack discussions to Amplitude metrics into one coherent picture
- A human doing this manually has to context-switch between 4+ tools. AI can query them all in parallel and synthesize in seconds
- The "silent failure detection" — finding initiatives with no activity across *any* source — is something that's genuinely hard for humans to catch at scale

**MCP is a game changer for enterprise AI:**
- No custom API integrations needed — Claude reads directly from Linear, Slack, Amplitude through MCP
- Adding a new data source is connecting an integration, not writing code
- The same MCP connections work in both CLI and browser — no duplication

**Prompt engineering at scale:**
- The `/pulse` skill is ~700 lines of structured instructions — it's essentially a program written in natural language
- Config-driven design means the skill never changes when you add a new team — just add a JSON config
- The same instructions power both the CLI skill and the browser project

**Trust requires evidence:**
- Every claim in a Pulse report links to its source — a Linear issue, a Slack message, an Amplitude chart
- Without this, leadership wouldn't trust AI-generated status updates
- Confidence scoring (High/Medium/Low) explicitly communicates when data is sparse

---

## Closing

"Project Pulse AI takes the manual, inconsistent, time-consuming process of status reporting and turns it into a 30-second conversation. The goal is to get every initiative at Tilt from 'unknown' to a real, evidence-backed health classification — and free up PM time for actual product decisions."

Link to repo: github.com/empowerfinance/project-pulse-ai
Link to browser project: claude.ai/project/019d6411-a3e6-70f2-b971-a8ac0ea9aca0

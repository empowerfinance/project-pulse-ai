# AI Trailblazers — Offsite Presentation

**Session:** Information Synthesis — Automating progress/status updates
**Presenter:** Thomas Quinto (with Alaina)
**Date:** Wed Apr 22, 2026 — Scottsdale
**Length:** 15 minutes, no dedicated Q&A

---

## Slide: Define the Problem (fill-in)

**PROBLEM STATEMENT**
Engineering and product leadership lack timely, consistent visibility into initiative health. Status updates are fragmented across Linear, GitHub, Slack, and Amplitude — requiring manual synthesis that is inconsistent, delayed, and often missing entirely.

**WHO'S AFFECTED**
- Primary: PMs and EMs who manually compile updates every sprint and author monthly roundups
- Secondary: BU leads and executives trying to track ~32 active EPD initiatives
- Workflows: weekly standups, initiative status updates, monthly roundups, exec reporting

**WHY IT MATTERS**
- Only **6 of ~32** active EPD initiatives have a health status set in Linear
- Multiple active projects have **0 status updates in 30+ days** — despite active issue and PR work
- Hours of PM/EM time per sprint spent on manual aggregation instead of product decisions
- Blockers and risks surface late — buried in Slack threads, missed by leadership
- Monthly roundups duplicate the same manual work across every pod

**SUCCESS CRITERIA**
- Every active initiative has a current, evidence-backed health status
- PM/EM time spent on status reporting drops meaningfully each sprint
- Leadership can get a company-level exec scan in under a minute, on demand
- Cross-source synthesis (Linear + Slack + Amplitude + GitHub) happens automatically

---

## 15-Minute Script

### 0:00 – 0:45 | Hook + intro (45s)

*[Slide: Initiative Question & Answer]*

> "Quick question for the room — across EPD, we have about 32 active initiatives in Linear. How many do you think have a health status set? … The answer is six. Multiple active projects have zero status updates in the last 30 days — while work is clearly happening, PRs are shipping, Slack is buzzing. Leadership is flying blind not because the work isn't getting done, but because **the signal is scattered across four different tools**.
>
> I'm Thomas, and during AI Sprint Week I built **Project Pulse AI** — a tool that collapses that scattered signal into evidence-backed status reports, at any level from a single project up to the whole company. I'll show you the problem, then demo it live."

### 0:45 – 3:30 | Define the problem (2:45)

*[Slide: Define the Problem]*

> "Here's the problem in four parts.
>
> **Problem statement.** Status updates at Tilt are fragmented across Linear, GitHub, Slack, and Amplitude. No single view exists. The synthesis is manual, inconsistent, delayed — and often just doesn't happen.
>
> **Who's affected.** Primarily PMs and EMs — they're the ones compiling updates every sprint, writing initiative notes, and turning raw data into the monthly roundup decks. Secondarily, everyone upstream of them: BU leads, execs, all of you — anyone trying to understand where EPD's 32 initiatives stand.
>
> **Why it matters.** The numbers I opened with — 6 of 32, and multiple projects dark for 30+ days — that's not a reporting problem, that's a **visibility gap**. Blockers surface late because they're buried in Slack. Monthly roundup time gets duplicated across every pod. And every hour a PM spends copy-pasting from Linear into Notion is an hour not spent on a product decision.
>
> **Success criteria.** Three things. One: every active initiative has a current, evidence-backed health status. Two: PM time on status reporting drops meaningfully. Three: leadership can get a company-wide exec scan in under a minute — on demand, not once a month."

### 3:30 – 4:15 | What I built (45s)

*[Slide: Noisy Signals -> Single Pulse]*

> "So what I built is **Project Pulse AI**. In one sentence: it synthesizes signals from Linear, GitHub, Slack, and Amplitude into structured, evidence-backed status reports — from a single project, up through initiative, pod, business unit, all the way to a company-level exec view.
>
> It's **not** replacing anyone's process. It doesn't ask teams to track anything new in Linear. It layers on top of the tools you already use and reads what's there.
>
> Fair warning — I worked on this solo for about a day and a half before jumping onto another team, so this is a rough prototype. But it's far enough along to show whether there's utility here. Let me just demo it."

### 4:15 – 12:30 | Live demo (8:15)

**[4:15 – 5:15 | Project-level demo — kick off the slow one first]**

*Open the Claude.ai project. Have a Linear project URL ready in clipboard.*

> "This is a Claude.ai project I set up — the browser version. No CLI, no install. Anyone with Claude access can open it. I'll prompt it with a Linear project URL — this is our 'Revamp the Referral Program' project — and just say: **summarize this project**."

*Paste prompt, kick off generation. It takes ~60-90 sec — keep talking while it runs.*

**[5:15 – 6:30 | While it runs — show how it works]**

> "While that runs, let me show you what's happening under the hood. The whole thing is a **Claude skill** — basically a structured natural-language program that tells Claude how to gather data, classify health, and shape a report.
>
> The key enabler is **MCP** — Model Context Protocol. Claude reads directly from Linear, Slack, and Amplitude through their MCP connectors. No API wrappers, no middleware, no glue code. Adding a new data source is adding a connector, not writing an integration. And the same skill drives both the CLI version and this browser version."

*Briefly show the project knowledge — point at the pod config files.*

> "Each pod is one JSON config — team key, Slack channels, Amplitude charts to watch. Adding your team is adding one file."

**[6:30 – 7:45 | Walk through the project-level output]**

*Scroll through the generated project report.*

> "Okay — here's the output. This project is classified **On Track**. Notice a few things.
>
> One — every claim is **linked to evidence**. The health classification cites the recent status update it read. The velocity number cites the Linear cycle. The blocker mention links to the actual Slack message. That's deliberate: leadership won't trust AI-generated updates unless they can audit them.
>
> Two — it **synthesizes across sources**. Linear issues tell you what's in progress. Slack tells you what's actually blocking. Amplitude tells you whether the work is moving the metric. A human doing this manually has to context-switch across four tools. Claude queries them in parallel and writes one coherent picture.
>
> Three — it flags **silent failures**. If an initiative has no PRs, no issue movement, no Slack activity — that's a signal, and it calls it out. That's the kind of thing humans miss at scale."

**[7:45 – 9:15 | Scale it up — pod rollup]**

*Switch to a pre-generated pod rollup for CAA (don't wait for live generation).*

> "That was one project. Same skill, higher level — here's a **pod rollup for CAA**. I generated this earlier so we don't watch a spinner.
>
> Overview table up top: active initiatives, on-track counts, cycle velocity. Initiative status table — one-line signal per initiative. Top risks with evidence links. Key wins. Metrics impact — this is where Amplitude shows up, activation trends week over week. And Slack signals — actual attributed quotes from pod discussions.
>
> This is what an EM or PM would spend an hour assembling manually. It's generated in about 90 seconds."

**[9:15 – 10:45 | All the way up — company pulse]**

*Switch to pre-generated company pulse.*

> "Same system, top of the stack — **company pulse**. Rolls up all configured BUs — right now that's Cash Advance and Credit Card, but we can add any pod with one config file.
>
> Executive overview. Health by business unit. Top company-wide risks. Top five wins. And one of my favorite sections — **cross-BU dependencies**. It's actually identifying initiatives that depend on work happening in another BU — the kind of thing that usually only surfaces when something breaks.
>
> The whole hierarchy: project, initiative, pod, BU, company. Each level up gets more condensed. A company pulse is a one-minute exec scan."

**[10:45 – 12:30 | Publishing — Linear + Slack]**

*Show a Linear initiative with a Pulse-generated status update on it; show the #pulse-reports Slack channel.*

> "Last piece — output destinations. By default, reports render in the conversation. But you can also publish.
>
> **To Linear** — it posts a native initiative status update, one per initiative, with the health classification set correctly. Same place PMs write updates manually. Here's one it generated.
>
> **To Slack** — posts to `#pulse-reports`. Summary card at the top level so the channel stays scannable, full report in a thread reply. Here's the channel.
>
> Everything confirms before it posts — nothing goes out without a human saying yes."

### 12:30 – 13:30 | The broader insight (60s)

> "Stepping back for a second — the thing I want you to take away is less about this specific tool and more about **what made it buildable in a day**.
>
> The unlock is that Claude can now read directly from our operational tools through MCP. The interesting work isn't plumbing anymore — it's in the **synthesis layer**. Writing good instructions for what 'on track' means. Deciding which signals actually matter. Making every claim auditable.
>
> That pattern — *'take scattered signal from the tools we already use, synthesize it for a specific audience'* — generalizes way beyond status reports. Anywhere at Tilt where someone is manually aggregating from 3+ tools into a doc, it's probably a Pulse-shaped problem."

### 13:30 – 14:30 | Next steps (60s)

> "Near-term, the things I'd want to push on:
>
> - **Scheduled runs** — pod rollups auto-posted to Slack every Monday morning; monthly roundups triggered last Friday of the month.
> - **Drift detection** — week-over-week diffs. 'This initiative was on track last week, now it's at risk, here's what changed.'
> - **Broader adoption** — the browser project is ready to go. Any pod can start with just a Linear team key and a config file. I'd love to pilot with two or three more pods to pressure-test it beyond Cash Advance and Credit Card.
> - **Richer Amplitude + Notion integration** — auto-discovering relevant charts, pulling in specs and meeting notes for extra context."

### 14:30 – 15:00 | Close (30s)

> "Bottom line — the manual, inconsistent, time-consuming process of status reporting becomes a 30-second conversation. The goal is to get every initiative at Tilt from 'unknown' to evidence-backed, and give PMs their sprint hours back for actual product decisions.
>
> Repo and browser project link are in the README. Grab me at the break if you want to try it on your pod. Thanks."

---

## Demo-day prep checklist

- [ ] **Pre-generate and save**: pod rollup (CAA), company pulse, a Linear initiative with a Pulse-posted status update, a `#pulse-reports` Slack thread. Don't rely on live generation for these — latency will eat your budget.
- [ ] **Live-generate only the project-level demo** — it's fast enough (~60-90s) and makes the tool feel real.
- [ ] **Pre-open tabs** in order: Claude.ai project, Linear project URL, Linear initiative with Pulse update, `#pulse-reports` Slack channel.
- [ ] **Verify MCP connectors** the morning of (Linear, Slack, Amplitude). The Loom showed Amplitude flaking — have a pre-generated fallback ready.
- [ ] **Copy-paste buffer**: keep the project URL and prompt text ready in a scratch doc so you're not retyping on stage.
- [ ] **Time check**: rehearse once end-to-end with a timer. If you're at 16+ min, cut the publishing segment (10:45–12:30) first — it's the most expendable.

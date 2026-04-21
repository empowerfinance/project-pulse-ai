# AI Trailblazers — Offsite Presentation

**Session:** Information Synthesis — Automating progress/status updates
**Presenter:** Thomas Quinto (with Alaina)
**Date:** Wed Apr 22, 2026 — Scottsdale
**Length:** 15 minutes, no dedicated Q&A

---

## Slide: Define the Problem (fill-in)

*(This section is a transcript of the slide as built — the script below walks through it.)*

**PROBLEM STATEMENT**
Status updates at Tilt are fragmented across Linear, GitHub, Slack and Amplitude. No single view exists. Synthesis is **manual**, **inconsistent**, **delayed** — and often just doesn't happen.

**WHO'S AFFECTED**
Primarily **EMs** and **PMs** — the ones compiling updates every sprint and turning raw data into monthly roundup decks. Secondarily, everyone upstream: **BU leads**, **execs** — anyone who wants visibility into where initiatives stand.

**WHY IT MATTERS**
Initiatives with no status updates — that's not a reporting problem, that's a **visibility gap**. Blockers surface late because they're buried in Slack. PMs collect and curate progress updates instead of spending time on product decisions.

**SUCCESS CRITERIA**
1. Every active initiative is current with evidence-based status
2. PM time on status reporting drops meaningfully
3. Leadership can get a **company-wide exec scan** in under a minute (on demand, not once a month)

---

## 15-Minute Script

### 0:00 – 1:30 | Open on the Status Tax (1:30)

*[Slide: The Status Tax]*

> "Before I tell you what I built, I want to talk about a tax we're all paying — but nobody has on their job description.
>
> I call it the **Status Tax**. At every level of our org, time gets spent turning raw work into readable status. Not the work itself — just the reporting on top of it.
>
> Now, to make the math concrete, I want to show you a **slice of Tilt** — the piece I built this tool against. Two business units. Four pods. Thirty-two active initiatives. That's not the whole company — it's a sample. The real numbers are multiples larger. But even in this slice, here's the shape.
>
> At the bottom: 32 active initiatives. On each one, an IC is shipping code, moving tickets, debugging. That's the work. One level up — four pods. Every week, a PM or EM reads across the initiatives below and writes a summary — for standups, for leadership, for Slack. One more level — two business units — rolling up two pods each. And at the top: one monthly digest that summarizes everything below.
>
> At every level, a human is reading the level below and writing for the level above. Back of the envelope, just in this slice, **about 50 hours a month** — a full work week, every month, on writing updates.
>
> Now, you can pressure-test that number. Maybe it's 30, maybe it's 80 — the assumptions are rough. But whatever the exact figure, it's a **non-trivial amount** of PM and EM time, every month, going to synthesis. And across the whole company, it scales materially higher.
>
> Two things are broken here. One: that's a lot of PM and EM time we don't get back. Two: even with all that effort, the information still lives in four tools — Linear, GitHub, Slack, and Amplitude. Leadership relies on manually-written updates, with no consolidated place to check on things async in between.
>
> I'm Thomas, and during AI Sprint Week I built **Project Pulse AI** to take on that tax. Let me set the problem up formally, then I'll demo it."

### 1:30 – 3:45 | Define the problem (2:15)

*[Slide: Define the Problem]*

> "Let's put this in the four-part structure.
>
> **Problem statement.** Status updates at Tilt are fragmented across Linear, GitHub, Slack, and Amplitude. No single view exists. Synthesis is manual, inconsistent, delayed — and often just doesn't happen at all.
>
> **Who's affected.** Primarily EMs and PMs — they're the ones compiling updates every sprint, turning raw data into monthly roundup decks. That's the Status Tax I just described, landing on them. Secondarily, everyone upstream who wants visibility into where initiatives stand — BU leads, execs, all of you.
>
> **Why it matters.** Initiatives with no status updates aren't a reporting problem — they're a **visibility gap**. Blockers surface late because they're buried in Slack. And PMs are collecting and curating updates instead of spending that time on product decisions.
>
> **Success criteria.** Three things. One: every active initiative is current with evidence-based status. Two: PM time on status reporting drops meaningfully. Three: leadership can get a company-wide exec scan in under a minute — on demand, not once a month."

### 3:45 – 4:30 | What I built (45s)

*[Slide: Noisy Signals -> Single Pulse]*

> "So what I built is **Project Pulse AI**. In one sentence: it synthesizes signals from Linear, GitHub, Slack, and Amplitude into structured, evidence-backed status reports — from a single project, up through initiative, pod, business unit, all the way to a company-level exec view.
>
> It's **not** replacing anyone's process. It doesn't ask teams to track anything new in Linear. It layers on top of the tools you already use and reads what's there.
>
> Fair warning — I worked on this solo for about a day and a half before jumping onto another team, so this is a rough prototype. But it's far enough along to show whether there's utility here. Let me just demo it."

### 4:30 – 12:15 | Live demo (7:45)

**[4:30 – 5:30 | Project-level demo — kick off the slow one first]**

*Open the Claude.ai project. Have a Linear project URL ready in clipboard.*

> "This is a Claude.ai project I set up — the browser version. No CLI, no install. Anyone with Claude access can open it. I'll prompt it with a Linear project URL — this is our 'Revamp the Referral Program' project — and just say: **summarize this project**."

*Paste prompt, kick off generation. It takes ~60-90 sec — keep talking while it runs.*

**[5:30 – 6:45 | While it runs — show how it works]**

> "While that runs, let me show you what's happening under the hood. The whole thing is a **Claude skill** — basically a structured natural-language program that tells Claude how to gather data, classify health, and shape a report.
>
> The key enabler is **MCP** — Model Context Protocol. Claude reads directly from Linear, Slack, and Amplitude through their MCP connectors. No API wrappers, no middleware, no glue code. Adding a new data source is adding a connector, not writing an integration. And the same skill drives both the CLI version and this browser version."

*Briefly show the project knowledge — point at the pod config files.*

> "Each pod is one JSON config — team key, Slack channels, Amplitude charts to watch. Adding your team is adding one file."

**[6:45 – 8:00 | Walk through the project-level output]**

*Scroll through the generated project report.*

> "Okay — here's the output. This project is classified **On Track**. Notice a few things.
>
> One — every claim is **linked to evidence**. The health classification cites the recent status update it read. The velocity number cites the Linear cycle. The blocker mention links to the actual Slack message. That's deliberate: leadership won't trust AI-generated updates unless they can audit them.
>
> Two — it **synthesizes across sources**. Linear issues tell you what's in progress. Slack tells you what's actually blocking. Amplitude tells you whether the work is moving the metric. A human doing this manually has to context-switch across four tools. Claude queries them in parallel and writes one coherent picture.
>
> Three — it flags **silent failures**. If an initiative has no PRs, no issue movement, no Slack activity — that's a signal, and it calls it out. That's the kind of thing humans miss at scale."

**[8:00 – 9:15 | Scale it up — pod rollup]**

*Switch to a pre-generated pod rollup for CAA (don't wait for live generation).*

> "That was one project. Same skill, higher level — here's a **pod rollup for CAA**. I generated this earlier so we don't watch a spinner.
>
> Overview table up top: active initiatives, on-track counts, cycle velocity. Initiative status table — one-line signal per initiative. Top risks with evidence links. Key wins. Metrics impact — this is where Amplitude shows up, activation trends week over week. And Slack signals — actual attributed quotes from pod discussions.
>
> This is what an EM or PM would spend an hour assembling manually. It's generated in about 90 seconds."

**[9:15 – 10:45 | All the way up — company pulse] (1:30)**

*Switch to pre-generated company pulse.*

> "Same system, top of the stack — **company pulse**. Rolls up all configured BUs — right now that's Cash Advance and Credit Card, but we can add any pod with one config file.
>
> Executive overview. Health by business unit. Top company-wide risks. Top five wins. And one of my favorite sections — **cross-BU dependencies**. It's actually identifying initiatives that depend on work happening in another BU — the kind of thing that usually only surfaces when something breaks.
>
> The whole hierarchy: project, initiative, pod, BU, company. Each level up gets more condensed. A company pulse is a one-minute exec scan."

**[10:45 – 12:15 | Publishing — Linear + Slack] (1:30)**

*Show a Linear initiative with a Pulse-generated status update on it; show the #pulse-reports Slack channel.*

> "Last piece — output destinations. By default, reports render in the conversation. But you can also publish.
>
> **To Linear** — it posts a native initiative status update, one per initiative, with the health classification set correctly. Same place PMs write updates manually. Here's one it generated.
>
> **To Slack** — posts to `#pulse-reports`. Summary card at the top level so the channel stays scannable, full report in a thread reply. Here's the channel.
>
> Everything confirms before it posts — nothing goes out without a human saying yes."

### 12:15 – 13:15 | The broader insight (60s)

> "Stepping back for a second — the thing I want you to take away is less about this specific tool and more about **what made it buildable in a day**.
>
> The unlock is that Claude can now read directly from our operational tools through MCP. The interesting work isn't plumbing anymore — it's in the **synthesis layer**. Writing good instructions for what 'on track' means. Deciding which signals actually matter. Making every claim auditable.
>
> That pattern — *'take scattered signal from the tools we already use, synthesize it for a specific audience'* — generalizes way beyond status reports. Anywhere at Tilt where someone is manually aggregating from 3+ tools into a doc, it's probably a Pulse-shaped problem."

### 13:15 – 14:15 | Next steps (60s)

> "Near-term, the things I'd want to push on:
>
> - **Scheduled runs** — pod rollups auto-posted to Slack every Monday morning; monthly roundups triggered last Friday of the month.
> - **Drift detection** — week-over-week diffs. 'This initiative was on track last week, now it's at risk, here's what changed.'
> - **Broader adoption** — the browser project is ready to go. Any pod can start with just a Linear team key and a config file. I'd love to pilot with two or three more pods to pressure-test it beyond Cash Advance and Credit Card.
> - **Richer Amplitude + Notion integration** — auto-discovering relevant charts, pulling in specs and meeting notes for extra context."

### 14:15 – 14:45 | Close (30s)

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
- [ ] **Time check**: total script budget is **14:45 with a 15-second buffer**. Rehearse once end-to-end with a timer. If you're at 15+ min, trim the publishing segment (10:45–12:15) first — it's the most expendable. Second cut: shorten the pod rollup walkthrough (8:00–9:15).

# Your AI agent is live. Now what does it cost? One attendee found out the hard way.

## 85 developers. 50 trial accounts. 2 apps shipped. 1 person who burned every credit before the guardrails went in.

---

In V1 we asked: can CoCo build a working AI agent in 60 minutes with 100 developers and a blank account?

In V2 we asked: can you wire that agent to Claude Code, Cursor, or any AI tool in the world?

V3 was always going to be the hard question. The one every enterprise team hits about three weeks after the demo:

**What does this actually cost to run?**

Last Wednesday at the TechEquity AI Infrastructure Forum, 85 developers — virtual and in-person — sat down with a live agent (or the CHECKPOINTS.sql to restore one) and we added the governance layer. Step by step. In real time.

One person did not wait for the governance layer.

---

## The thing that happened two days before

On August 18, 2026 — two days before this workshop — Snowflake announced Dynamic Model Routing inside Cortex AI Gateway.

The short version: your agent now automatically routes simpler tasks to lighter, cheaper models. Reserving frontier models for complex reasoning. In one internal evaluation, agents using mixed open + proprietary models showed up to 3x greater token efficiency versus routing everything to the frontier.

Sridhar Ramaswamy put it simply: *"Usage is an input. The question that matters is what a company gets in return."*

We opened the session with that quote. Then we asked: who here knows what their agent cost to run last week?

Nobody raised their hand.

That's the gap Steps 4 through 6 close.

---

## 50 trial accounts. 2 apps. 1 memorable lesson.

Before we built anything, a few numbers from the session:

- **85 participants** — virtual and in-person, the largest v3 audience yet
- **50 trial accounts activated** — people who had the agent running within the session window
- **2 community apps deployed** — attendees who shipped something beyond the workshop and posted it

And then there was the one account that lit up in ACCOUNT_USAGE before we got to Step 5.

Someone got curious. They ran the GitTrend agent — the one we built in V1 and V2 — heavily. Asked it a lot of questions. Hit the AI_COMPLETE call repeatedly against the 107M event dataset. By the time we got to "Step 4: Cost Visibility," their CORTEX_AI_FUNCTIONS_USAGE_HISTORY was... lively.

They had burned through their trial credit allocation before we could teach them to stop it.

This was the best possible advertisement for why we built Step 5.

---

## What changed from V2 to V3

V2 ended with GITTREND_MCP — a Snowflake-managed MCP Server that made your agent queryable from any MCP client in the world. One DDL statement. Your agent shows up in Claude Desktop, Cursor, VS Code.

V3 picks up exactly there. The agent is alive. The question is: what does it cost, and how do you keep it from surprising you?

Three things we added, in order:

**Step 4 — Cost Visibility**

Four ACCOUNT_USAGE views. Different latency, different granularity.

```
CORTEX_AI_FUNCTIONS_USAGE_HISTORY  — ≤5 min lag. Per user, per model, per function.
CORTEX_AGENT_USAGE_HISTORY         — up to 1 hr lag. Per agent, per user.
SNOWFLAKE_COWORK_USAGE_HISTORY     — up to 1 hr lag. CoWork sessions.
METERING_HISTORY                   — up to 3 hr lag. High-level summary.
```

One CoCo prompt runs all four. On a brand-new trial account, CORTEX_AI_FUNCTIONS_USAGE_HISTORY is the one with data immediately — your AI_COMPLETE calls from Steps 0-3 are already there within minutes, showing your username, the model (claude-sonnet-4-6 or snowflake-arctic-embed), the call count, and the exact credit cost.

That's when the room got quiet. Real numbers. Real usage. Real credit costs. Attached to real names.

**Step 5 — Cost Controls**

Two billing systems in one Snowflake account. Platform Credits (compute) and AI Credits ($2.00/credit flat since April 2026, edition-independent). Resource Monitors cover the first. They do not cover the second.

Three guardrails, one CoCo session:

1. **Resource Monitor** on WORKSHOP_WH — compute ceiling, suspend at 100%
2. **Account Budget** (`account_root_budget`) — monthly limit on everything: warehouse, AI, Cortex Search. Two calls. No tag setup.
3. **Per User AI Quota** — `CREATE SNOWFLAKE.CORE.QUOTA`. Covers AI Functions, Cortex Agents, CoCo, CoWork. Daily and monthly limits. Block enforcement on. When you hit it, new AI requests are denied within minutes.

The person who burned their credits? Their trial account was the live demo for what happens without Step 5.

**Step 6 — Ask CoCo About the Cost**

This one surprised people. You built the agent. You measured it. You put guardrails on it. Then you ask the same tool that built it to optimize it.

The Cost Intelligence skill in CoCo queries CORTEX_AI_FUNCTIONS_USAGE_HISTORY, renders a daily cost chart inline, and gives you specific recommendations: reduce `max_results` on GITHUB_REPO_SEARCH, add a concise instruction to the system prompt, switch from `auto` to a specific model for simple queries.

All real levers. All implementable in 10 minutes without changing what the agent does.

Dynamic model routing is Snowflake doing this at the infrastructure layer automatically. Cost Intelligence is you doing it at the application layer deliberately. Both matter.

---

## What it actually costs

Here's the number nobody expected.

The entire v3 build — restore from V2, Step 4 (four-view cost breakdown), Step 5 (three guardrails), Step 6 (optimization loop), a CoCo session — runs under 0.01 AI credits for the workshop itself.

The S3 data load (107M events) hits WORKSHOP_WH for about 1.2 compute credits. The Cortex Search index refresh costs a small fraction of an AI credit. The AI_COMPLETE calls in the agent are measurable in thousandths of a credit per query.

The infrastructure to govern all of that? Free to create.

The question isn't whether you can afford to run a governed AI agent. It's whether you can afford to run one you can't see.

---

## When it stopped feeling like a tutorial

Step 4, first result. CORTEX_AI_FUNCTIONS_USAGE_HISTORY came back with rows.

Not sample data. Your username. Your function calls. Your model. Your credits. Attached to the thing you built in the last 30 minutes.

For some people that was a small number. For at least one person it was not small at all. Both reactions proved the same point: visibility is the prerequisite for everything else. You can't optimize what you can't measure. You can't govern what you can't see.

FinOps for AI is just FinOps. Same discipline, new services to track.

---

## Build it yourself

Everything is open source:

**Workshop repo:** https://github.com/sfc-gh-rbachala/building-ai-agents-with-coco-workshop

`WORKSHOP-GUIDE-V3.md` walks through Steps 4-6. `CHECKPOINTS.sql` has fallback SQL for every step — CP7 through CP9 are fully doc-verified. The `media/` folder has demo videos.

**Free trial:** https://signup.snowflake.com/

You need the V2 state before you start — either you ran it, or SETUP + CP1–CP6 in CHECKPOINTS.sql restores everything in about 7 minutes.

The same governance pattern works on any Snowflake-based agent: your support ticket classifier, your sales pipeline analyzer, your internal docs assistant. The GitHub dataset is the example. The pattern transfers.

---

## What's next: V4

The question the room kept asking after Step 6: *can the agent govern itself?*

Not just CoCo making recommendations — but the agent embedding cost awareness into its own behavior. System prompts with budget constraints. Dynamic max_results based on query complexity. Agents that know they're operating inside a quota and route their own tool calls accordingly.

That's V4. Watch the repo.

If this was useful — ⭐ star the repo. Takes two seconds and helps others in the community find it.

https://github.com/sfc-gh-rbachala/building-ai-agents-with-coco-workshop

---

*[Richie Bachala](https://x.com/richiebachala) — Solutions Architecture @ Snowflake*

---

**Tags:** Snowflake · AI Agent · FinOps · Cost Management · CoCo · Cortex · MCP · Model Context Protocol

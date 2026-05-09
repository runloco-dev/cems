# CEMS Handoff — Getting Started with Claude Code

This doc gets you from zero to contributing in one session.

## What this is

CEMS replaces the existing conference and event management software at the facility. It's built on Salesforce Government Cloud Plus (FedRAMP High) using Service Cloud, Sales Cloud, Experience Cloud, and Agentforce.

You know this facility better than anyone on the team. The design was built from a high-level conversation and will have wrong assumptions. **Your domain expertise is the most important thing right now — more than writing code.**

## Step 1: Clone the repo

```bash
git clone git@github.com:runloco-dev/cems.git
cd cems
```

## Step 2: Install Claude Code

```bash
npm install -g @anthropic/claude-code
```

Requires an Anthropic API key. Set it:

```bash
export ANTHROPIC_API_KEY=your-key-here
```

Then launch from the repo root:

```bash
cd /path/to/cems
claude
```

## Step 3: Read these first (in order)

1. [Architecture and Design Spec](https://github.com/runloco-dev/cems/blob/main/docs/superpowers/specs/2026-05-08-cems-design.md) — the full design we've agreed on. Read this with a skeptical eye.
2. [Issue #1 — Design Review](https://github.com/runloco-dev/cems/issues/1) — targeted questions where we most need your corrections.
3. [Phase 1 Implementation Plan](https://github.com/runloco-dev/cems/blob/main/docs/superpowers/plans/2026-05-08-phase1-enrollment.md) — the build plan for enrollment, waitlist, grading, portal, and Agentforce. Not deployed yet.

## Step 4: File corrections before we build

**Do this before running any implementation tasks.** Go to [Issue #1](https://github.com/runloco-dev/cems/issues/1) and comment on anything that doesn't match how the facility actually works. Open new issues for anything missing.

The individual open questions are in Issues #2–6 — answer those too if you have context.

Once your corrections are in, we'll revise the design spec and plan accordingly. No rework cost to getting it right now.

## Step 5: Run the implementation plan with Claude Code

Once the design is validated, tell Claude Code to start building:

> "Use the superpowers:subagent-driven-development skill to execute the Phase 1 plan at https://github.com/runloco-dev/cems/blob/main/docs/superpowers/plans/2026-05-08-phase1-enrollment.md"

Claude Code will work through the plan task by task, deploying metadata, writing Apex with tests, and checking in for review between tasks.

If you want to run a specific task directly:

> "Execute Task 8 from the Phase 1 plan — EnrollmentService Apex TDD"

## Useful Claude Code prompts to start with

**Review the design against your domain knowledge:**
> "Read the design spec at https://github.com/runloco-dev/cems/blob/main/docs/superpowers/specs/2026-05-08-cems-design.md and ask me questions about whether the data model and workflows match how this facility actually operates. Ask one question at a time."

**Add missing workflows:**
> "Based on the existing design spec, help me spec out [workflow you know is missing] and file it as a GitHub issue."

**Revise a specific section:**
> "The nomination workflow in the design spec doesn't account for [thing you know]. Update the spec and the Phase 1 plan to reflect this."

**Check what's open:**
> "List all open GitHub issues in this repo and summarize what needs my input."

## Key decisions already made

- **Platform:** Salesforce GovCloud+ (FedRAMP High) — not commercial Salesforce
- **Data classification:** CUI only in Salesforce. Classified content and adjudication records stay off-platform.
- **Badging:** CEMS manages the data (badge tiers, requests, workflow). Physical ACS hardware integration is a later milestone via Platform Events.
- **Clearance references:** Clearance tier stored as a reference field on Contact, not the authoritative record. Mocked for now; will connect to DISS/NBIS API later.
- **Agentforce:** 3 agents planned (Enrollment Concierge, Scheduling Assistant, Badging Coordinator). All agent actions are also invocable Apex so they work as Flow steps if a GovCloud+ AI feature isn't yet FedRAMP High authorized.
- **4 phases:** Enrollment → Scheduling → Lodging → Badging. Nothing is deployed yet.

## Contacts

- **Brian (bdins)** — project lead, reach via GitHub or Slack

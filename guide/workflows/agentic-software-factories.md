---
title: "Agentic Software Factories: Orientation Map"
description: "Where to start when scaling from a single Claude Code session to a multi-agent software factory, and when a closed commercial platform actually beats the native path"
tags: [workflow, agents, orchestration, software-factory, decision-guide]
---

# Agentic Software Factories: Orientation Map

This page answers one question: you want to run Claude Code as something closer to a software factory than a single session, so where do you start and what do you pick. It does not repeat the detailed material that already exists across six other files in this guide. It points to it, in order, and adds the two things that were missing: an honest decision tree for closed commercial platforms, and a governance checklist that includes a question no vendor page asks about itself.

**Reading time**: ~15 min
**Prerequisites**: Basic familiarity with sub-agents and the Task tool
**Related**: [Agent Teams](./agent-teams.md), [Dynamic Workflows](./dynamic-workflows.md), [Plan-Validate-Execute Pipeline](./plan-pipeline.md), [Spec-First Development](./spec-first.md), [Agent Tools: Beyond Claude Code](../ecosystem/agentic-tools.md)

---

## Table of Contents

1. [The spectrum: six levels, six costs](#1-the-spectrum-six-levels-six-costs)
2. [The missing decision tree: when does a closed software factory actually win](#2-the-missing-decision-tree-when-does-a-closed-software-factory-actually-win)
3. [The map of six files](#3-the-map-of-six-files)
4. [Five governance questions before you adopt anything](#4-five-governance-questions-before-you-adopt-anything)
5. [The unbounded velocity trap](#5-the-unbounded-velocity-trap)

---

## 1. The spectrum: six levels, six costs

Every option below solves a real problem the previous level does not. None of them is free, and the cost is rarely just money.

**Level 0: a single Claude Code session.** Solves nothing beyond what one developer, one context window, and one conversation can hold. Costs whatever your subscription already costs. Move up when a task naturally splits into independent chunks that would otherwise force you to context-switch manually inside one session.

**Level 1: `/batch`.** A native command, added in v2.1.63, that distributes a large repetitive change (a migration, a bulk type-annotation pass, a dependency swap) across 5 to 30 parallel agents, each in its own git worktree, each opening its own pull request. Solves the "same edit, many files, no shared judgment needed" case. Costs roughly N times the token cost of a single agent doing one unit of the work, paid up front, with no coordination overhead because there is nothing to coordinate: the units are independent by construction. Full description at [guide/ultimate-guide.md:9642](../ultimate-guide.md#the-batch-command). Move up when the units are not actually independent, when they need to see each other's output, or when the task is not repetitive but exploratory.

**Level 2: Agent Teams (native, experimental).** Multiple Claude Code instances coordinating through git and a shared mailbox, with a lead agent synthesizing results from teammates. Solves read-heavy coordination: multi-angle code review, parallel hypothesis testing during debugging, architecture analysis from several perspectives at once. Costs 3x or more in tokens compared to a single agent, and real cognitive load in learning when it helps versus when it just burns budget for the same output. Six documented production cases exist (Fountain, CRED, an Anthropic Research internal C compiler project, Paul Rayner's job-search app, parallel hypothesis testing, and one large refactor), all detailed with sources in [guide/workflows/agent-teams.md](./agent-teams.md) section 4. The decision tree at [guide/workflows/agent-teams.md](./agent-teams.md) section 7 already separates Agent Teams from plain multi-instance and dual-instance work; read that before reading anything else on this page. Move up when the work stops being "several agents look at the same thing" and becomes "several agents build different pieces of a defined pipeline."

**Level 3: Dynamic Workflows (`ultracode`).** A JavaScript-orchestrated pipeline, added in v2.1.154, built from a small set of primitives (`agent`, `parallel`, `pipeline`, `phase`) that gives you deterministic control flow, schema-validated outputs between stages, and automatic resume after an interruption. The orchestrator script itself costs nothing to run; every dollar goes into the `agent()` calls it makes. This is the closest thing in the native toolchain to a scriptable factory: a real dev-flow example (issue to merged PR) is documented end to end in [guide/workflows/dynamic-workflows.md](./dynamic-workflows.md) section 7. Move up when you need durable state across many runs, an ADR-driven learning loop, or a formal plan-validate-execute separation with independent reviewers, none of which the raw primitives give you by themselves.

**Level 4: Plan-Validate-Execute Pipeline.** Three commands (`/plan-start`, `/plan-validate`, `/plan-execute`) built on top of the same primitives, adding a dynamic research agent pool, a two-layer independent validation pool, and an ADR learning loop that reduces how often a human needs to interrupt the next run. Documented in full in [guide/workflows/plan-pipeline.md](./plan-pipeline.md), including a real cost profile (typically $2 to $10 per Tier 2 feature, falling as ADR coverage grows). This is the practical ceiling of what the native Claude Code toolchain offers today for something resembling a software factory. Move up only if you need something outside the Anthropic ecosystem entirely: cross-machine agent federation, a different model provider mixed into the pipeline, or a vendor-managed governance layer you do not want to build yourself.

**Level 5: third-party orchestration frameworks.** Tools that sit above or beside Claude Code and manage multiple instances or agents at a level the native toolchain does not reach: Ruflo (hierarchical swarms, cross-machine federation), Gas Town and multiclaude (multi-Claude workspace managers), Entire CLI (governance-first, sequential handoffs, compliance audit trails). Full comparison in [guide/ecosystem/third-party-tools.md](../ecosystem/third-party-tools.md) under Multi-Agent Orchestration and External Orchestration Frameworks, and in [guide/ecosystem/agentic-tools.md](../ecosystem/agentic-tools.md) for tools that replace Claude Code rather than orchestrate it. Costs go up in three ways at once: token spend, infrastructure to run and maintain the orchestrator, and the time to learn a second system's failure modes on top of Claude Code's own. Move up (or rather, sideways) to level 6 only if you also want someone else to own the entire spec-to-deploy lifecycle, not just the coordination layer.

**Level 6: closed full-cycle software factories.** Commercial platforms (Maleus, Factory.ai, Blitzy, and Devin when used this way) that run the whole cycle behind a managed interface: specification intake, isolated parallel build, validation, deployment. Documented as a market category in [guide/workflows/spec-first.md:959-999](./spec-first.md#full-cycle-ai-software-factories). This is where section 2 below matters, because the honest cost of this level is not published by any of the vendors.

---

## 2. The missing decision tree: when does a closed software factory actually win

Two decision trees already exist in this guide for adjacent choices: multi-instance versus single-instance work in [guide/ultimate-guide.md:20320](../ultimate-guide.md#decision-matrix) (branches on developer count and budget), and Agent Teams versus multi-instance versus dual-instance in [guide/workflows/agent-teams.md](./agent-teams.md) section 7 (branches on task shape). No equivalent existed for the choice between a closed software factory and the native stack (Agent Teams, `/batch`, dynamic-workflows) that already covers most of the same ground for the cost of a Claude subscription. Here is one, built from what could actually be verified rather than from vendor claims.

```
Do you need the whole spec-to-deploy cycle owned by one vendor?
├─ No, you just need faster parallel execution on defined tasks
│  └─ Use Level 1-4 (native stack). Stop here.
│
├─ Yes, but your team can write specs and review PRs
│  ├─ Team size < 10, no dedicated platform engineer
│  │  └─ Native stack (Level 2-4) almost always wins on cost and control.
│  │     A closed factory adds a vendor relationship, a second billing
│  │     model, and a lock-in risk for a problem you can already solve.
│  │
│  ├─ Team size 10-50, budget clearly allocated for tooling
│  │  ├─ Regulatory or audit requirement (SOC2, HIPAA, traceability)?
│  │  │  └─ Consider Entire CLI (governance-first, sequential, native
│  │  │     Claude Code compatible) before a full closed factory.
│  │  │     See guide/ops/ai-traceability.md section 5.1.
│  │  └─ No compliance driver, just want to move faster
│  │     └─ Pilot one closed factory on a single bounded project.
│  │        Do not migrate the primary codebase on a first pilot.
│  │
│  └─ Team size 50+, or non-developer stakeholders need to ship product
│     └─ A closed factory (Maleus for natural-language intent, Factory.ai
│        for the Missions architecture, Blitzy for very large codebases)
│        becomes defensible. Run the governance checklist in section 4
│        below before signing anything, because the vendor will not
│        run it for you.
│
└─ Yes, and you want to self-host and keep the code fully in-house
   └─ OpenHands is the closest open-source equivalent to Devin's
      dependency-graph parallelism, at the cost of running the
      infrastructure yourself. See guide/ecosystem/agentic-tools.md
      section 2.4.
```

Say the quiet part directly: for a small team, the honest answer is almost never. Treat that as this guide's editorial judgment rather than a measured threshold, because no one has published the comparative cost study that would settle it. The reasoning behind it is checkable, though: Agent Teams has six production use cases documented with sources ([`agent-teams.md`](/Users/florianbruniaux/Sites/perso/claude-code-ultimate-guide/guide/workflows/agent-teams.md) section 4), while the closed platforms have funding rounds and demos and no independently verified deployment with metrics. Agent Teams plus `/batch` plus dynamic-workflows already cover the large majority of what a closed software factory sells, for the price of a Claude subscription rather than a second vendor contract. The gap that remains, mostly, is not capability. It is governance packaging: audit trails, approval gates, and a managed interface for people who are not going to read a git diff. If you need that packaging specifically, buy it. If you are buying a closed factory because parallel agent execution sounds like it needs a dedicated platform, it does not, not yet, not at that team size.

The evidence gap cuts the other way too, and it should make anyone cautious regardless of team size. As of this writing, none of the closed platforms in this category (Maleus, Factory.ai outside its Missions CLI, Blitzy) has a single independently verified production case study with measured outcomes documented anywhere in this guide's research, in contrast to the six sourced production cases for Agent Teams. Funding announcements are not proof of production reliability. Treat every claim from a closed factory's own marketing material the way section 5 below treats a README: verify it is wired before you believe it.

---

## 3. The map of six files

No single file in this guide covers the whole subject, and that is deliberate: each file earns its place by covering one layer well rather than everyone trying to cover everything shallowly. Read in this order depending on what you actually need.

**Starting from zero**, read this page first, then [guide/workflows/agent-teams-quick-start.md](./agent-teams-quick-start.md) for a 5-minute native setup with four copy-paste patterns, before touching anything else.

**Building a native pipeline**, read [guide/workflows/dynamic-workflows.md](./dynamic-workflows.md) for the primitives, then [guide/workflows/plan-pipeline.md](./plan-pipeline.md) for the three-command production pipeline built on top of them.

**Writing specs before code**, read [guide/workflows/spec-first.md](./spec-first.md), which also covers the closed software factory market category, the four governance questions this page extends in section 4, and the decorative-CI trap.

**Comparing methodologies at the strategic level** (BMAD, GSD, and thirteen others), read [guide/core/methodologies.md](../core/methodologies.md), Tier 1: Strategic Orchestration.

**Evaluating a third-party tool that runs alongside or instead of Claude Code**, read [guide/ecosystem/agentic-tools.md](../ecosystem/agentic-tools.md) for autonomous coders and multi-agent frameworks (Devin, OpenHands, CrewAI, LangGraph), and [guide/ecosystem/third-party-tools.md](../ecosystem/third-party-tools.md) for tools that specifically orchestrate multiple Claude Code instances (Gas Town, multiclaude, Ruflo, Entire CLI).

**Scaling the decision itself** (how many developers, what budget, what red flags mean rollback), read [guide/ultimate-guide.md](../ultimate-guide.md), the progressive scaling section starting around line 19986.

---

## 4. Five governance questions before you adopt anything

[guide/workflows/spec-first.md:970-975](./spec-first.md#full-cycle-ai-software-factories) already lists four questions worth asking before adopting any full-cycle platform, closed or open. Repeated here because they apply just as much to a native dynamic-workflow pipeline you build yourself as to a vendor platform, plus a fifth added from direct evidence.

1. **Deterministic gate, or LLM self-grading?** Does a separate, non-LLM process (lint, type check, test suite, contract validation) block the merge, or does the agent that wrote the code also decide whether it is correct?
2. **Is there a stop-the-line mechanism?** After N failed remediation attempts, does the pipeline halt and escalate to a human, or does it keep retrying and burning tokens?
3. **Is every decision traceable?** Can you reconstruct, after the fact, which agent and which model version made a given change, and whether a human approved it?
4. **Does the spec stay authoritative after the first generation?** Or does the code drift away from the spec as the app evolves past its first version, with no reliable sync mechanism?
5. **Is the verification feature the README advertises actually wired into the code path that runs?** This is not a hypothetical concern. A source-level audit of one self-described "software factory" (see section 5 below) found a 986-line adversarial verification module, a genuinely strong design that would answer question 1 above better than most alternatives in this guide, imported with `import type` only. Type-only imports erase at compile time. The module was never instantiated at the single production entry point; the code path that should have called it silently treated the check as failed instead. The feature existed, was tested, and did not run. Checking that a file exists, or even that it is imported, proves nothing about whether it executes. Ask for the specific runtime entry point that calls the verification code, not the module's existence in the repository.

---

## 5. The unbounded velocity trap

A source-level evaluation of Fusion, a self-described "software factory run by a multi-agent orchestrator," is documented in full at [docs/resource-evaluations/fusion-multi-agent-orchestrator.md](../../docs/resource-evaluations/fusion-multi-agent-orchestrator.md). It is short and worth reading directly. The summary here is deliberately narrow: what it demonstrates about velocity without architecture, not a review of the tool itself.

The repository is under four months old (first commit 2026-03-25, evaluated 2026-07-15) and contains 727,279 lines of code excluding tests, written almost entirely by one person working with agents: 94% of the 11,333 commits on main come from a single contributor identity plus its bot accounts, with the second human contributor at 3.3%. One file, `executor.ts`, runs to 19,328 lines. Two storage backends (SQLite and Postgres) are maintained in full parallel, each store implemented twice, with the roadmap still calling the migration "planned" while the cutover review sits dated a day before the evaluation. The behavioral verification module described in question 5 above, a design pattern strong enough to solve the decorative-CI problem this guide already flags at [guide/workflows/spec-first.md:979-988](./spec-first.md#the-decorative-ci-trap-question-1-in-practice), was built, tested, and never wired into production.

None of this is a claim that agentic velocity is bad. It is a measurement of what happens when velocity outruns architecture: 727,000 lines shipped in sixteen weeks by one person is a real capability, not a myth. What that capability produces without a second reviewer, a bus factor above one, or a deliberate stop to consolidate the two storage backends into one, is documentation describing behavior that does not happen and a codebase that will cost more to untangle than it took to write. Scale up your own use of Level 3 through 6 above with that trade in view. The question is never whether agents can produce this much code. It is whether anyone, including the person who wrote the prompts, can still explain what all of it does a year later.

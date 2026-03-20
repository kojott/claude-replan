---
name: replan
description: Review and validate an existing implementation plan by launching parallel subagents that check it from multiple angles — codebase alignment, best practices, code standards, feasibility, and fresh perspectives. Use whenever you have a plan ready and want to validate it before execution, or when the user invokes /replan. This applies to ANY kind of plan — implementation plans, research plans, design plans, migration plans, etc.
allowed-tools: [Agent, Read, Glob, Grep, Bash, Edit, TodoWrite]
---

# Plan Review (/replan)

You have a plan in the current conversation (or at a file path the user provides). Your job is to validate it thoroughly by dispatching parallel review subagents, then update the plan based on their findings.

**Announce:** "Reviewing the plan with parallel subagents..."

## Step 1: Understand the Plan

Read the plan carefully. Identify:
- **Type of plan**: implementation, research, design, migration, refactor, etc.
- **Key domains it touches**: backend, frontend, database, API, design, infrastructure, external services, etc.
- **Specific technologies and files** mentioned
- **Assumptions** the plan makes (explicit and implicit)

## Step 2: Design Review Agents

Based on what the plan actually covers, decide which review perspectives are needed. The goal is full coverage of the plan's scope — not a fixed set of agents.

**Pick from these perspectives (and invent new ones if the plan demands it):**

| Perspective | When to use | What it checks |
|---|---|---|
| **Codebase alignment** | Always for code-touching plans | Do referenced files/functions/APIs actually exist? Does the plan match current code structure and patterns? Are there existing utilities it should reuse? |
| **Best practices & architecture** | Always for code-touching plans | SOLID, DRY, YAGNI, separation of concerns, error handling, security (OWASP top 10), performance implications |
| **Project standards** | When CLAUDE.md / linting / conventions exist | Naming conventions, file structure patterns, test conventions, commit style, tech stack alignment |
| **Feasibility & risks** | Always | Missing steps, implicit dependencies between tasks, ordering issues, underestimated complexity, things that could block execution |
| **Fresh perspective** | Always — this is the wildcard | An agent that reads the plan cold and asks "what's missing that nobody thought of?" — edge cases, user experience gaps, operational concerns, things the plan author's tunnel vision missed |
| **Research & fact-checking** | Plans involving external APIs, libs, protocols | Are the APIs/libraries/versions referenced real and current? Do they work the way the plan assumes? |
| **Design & UX** | Plans with UI/visual components | Visual consistency, accessibility, responsive behavior, interaction patterns, loading/error states |
| **Data & schema** | Plans touching databases or data models | Migration safety, backwards compatibility, indexing, data integrity, query performance |
| **Security** | Plans touching auth, user input, APIs | Authentication/authorization flows, input validation, secrets handling, attack surface |
| **Operations** | Plans touching infra, deployment, monitoring | Rollback strategy, monitoring gaps, configuration management, failure modes |

**Rules for agent count:**
- Minimum 3 agents (even for simple plans — codebase alignment + feasibility + fresh perspective)
- Always include the **fresh perspective** agent — it consistently catches things others miss
- Scale with plan complexity: a 3-task plan might need 3-4 agents, a 15-task plan might need 6-8
- Each agent should have a distinct, non-overlapping focus area
- If you're unsure whether a perspective is needed, include it — over-reviewing is better than missing something

## Step 3: Dispatch All Agents in Parallel

Launch all review agents simultaneously using the Agent tool. Each agent gets:

1. **The complete plan text** (copy it in full — agents don't have your conversation context)
2. **Relevant codebase context** — for codebase alignment agents, include file paths to check. For standards agents, include CLAUDE.md content.
3. **A specific review mandate** — what exactly to look for
4. **Output format** — structured findings

### Agent Prompt Template

Each agent should receive a prompt structured like this:

```
You are reviewing an implementation plan from the perspective of [PERSPECTIVE].

## The Plan
[FULL PLAN TEXT]

## Your Review Mandate
[SPECIFIC INSTRUCTIONS FOR THIS PERSPECTIVE]

## Project Context
[RELEVANT CONTEXT: CLAUDE.md contents, file paths, tech stack, etc.]

## Output Format
Return your review as:

### [PERSPECTIVE] Review

**Verdict: PASS | ISSUES FOUND | CONCERNS**

**Critical Issues** (must fix before execution):
- [issue]: [why it matters] → [suggested fix]

**Recommendations** (should fix, but not blocking):
- [recommendation]: [why it matters] → [suggested approach]

**Observations** (informational, no action needed):
- [observation]

If everything looks good, say PASS and briefly explain why the plan is solid from your perspective.
```

For the **fresh perspective** agent specifically, use this framing:

```
You are reviewing this plan with completely fresh eyes. You haven't been part of any discussion about it. Read it cold and ask yourself:

- What would go wrong that nobody anticipated?
- What's missing that seems obvious once you think about it?
- Are there simpler alternatives to any of the proposed approaches?
- What will the person executing this plan wish they'd known upfront?
- Are there edge cases, error states, or user scenarios the plan ignores?

Be constructively critical. The goal is to find the blind spots.
```

## Step 4: Synthesize and Update the Plan

Once all agents return:

1. **Collect all findings** — read every agent's review
2. **Categorize by severity**:
   - **Critical**: Must be fixed — wrong assumptions, missing steps, security holes, references to non-existent code
   - **Important**: Should be fixed — better approaches, missing error handling, overlooked edge cases
   - **Minor**: Nice to have — style suggestions, optional improvements
3. **Present a summary** to the user: which agents found what, critical issues first
4. **Update the plan** directly — incorporate all critical and important findings. For minor items, use your judgment.
5. **Note what changed** — after updating, briefly list the key changes made so the user knows what was improved

The plan should be noticeably better after this process. If agents found nothing critical, that's a good sign — say so.

## Adapting to Plan Type

The beauty of this approach is that it adapts to whatever the plan contains:

- **Research plan?** → Dispatch fact-checking agents that verify sources, check if referenced tools/APIs exist, validate methodology
- **Design plan?** → Dispatch agents that check visual consistency, load reference images/designs if paths are provided, verify component library usage
- **Migration plan?** → Dispatch agents focused on rollback safety, data integrity, backwards compatibility, downtime estimation
- **Refactor plan?** → Dispatch agents that verify test coverage exists for affected code, check for hidden consumers, validate that the refactor actually simplifies things

Read the plan. Think about what could go wrong. Design agents to catch those things.

# claude-replan

A Claude Code plugin that validates your implementation plans before execution by launching parallel review subagents — each checking the plan from a different angle.

## The problem

You write a plan. It looks solid. You start executing. Three tasks in, you realize the plan assumed a function that doesn't exist, missed a critical edge case, or ignored your project's conventions entirely. Now you're rewriting the plan mid-flight.

## The solution

`/replan` throws your plan at a squad of parallel review agents before you write a single line of code. Each agent has a different job — one checks if the code references are real, another stress-tests feasibility, another reads the plan cold and asks "what did everyone miss?"

The result: a plan that's been pressure-tested from every angle, with all findings incorporated automatically.

## How it works

```
You: /replan

Claude: Reviewing the plan with parallel subagents...

  Agent 1 (Codebase alignment)   → Checks referenced files/APIs actually exist
  Agent 2 (Best practices)       → SOLID, DRY, security, error handling
  Agent 3 (Feasibility & risks)  → Missing steps, ordering issues, blockers
  Agent 4 (Fresh perspective)    → Reads the plan cold, finds blind spots
  Agent 5 (Project standards)    → CLAUDE.md conventions, naming, test patterns

  ✓ All agents returned.

  Critical: Plan references `getUserById()` which was renamed to `findUser()` in v3.2
  Critical: Task 4 depends on Task 6 output — wrong ordering
  Important: No error handling for API timeout in Task 2
  Important: Missing rollback strategy for database migration

  → Plan updated with all findings.
```

## Key features

**Adaptive agent selection** — agents are chosen based on what the plan actually covers, not a fixed checklist. A database migration plan gets different reviewers than a UI redesign plan.

**Always includes a "fresh perspective" agent** — an agent that reads the plan with zero context and asks "what would go wrong that nobody anticipated?" This consistently catches things the other agents miss.

**Scales with complexity** — simple 3-task plans get 3-4 agents. Complex 15-task plans get 6-8. Each with a distinct, non-overlapping focus area.

**Works with any plan type:**

| Plan type | Review focus |
|---|---|
| Implementation | Codebase alignment, architecture, test coverage |
| Research | Fact-checking, methodology, source verification |
| Design/UI | Visual consistency, accessibility, component library usage |
| Migration | Rollback safety, data integrity, backwards compatibility |
| Refactor | Hidden consumers, test coverage, actual simplification |
| Infrastructure | Failure modes, monitoring gaps, configuration management |

**Updates the plan automatically** — findings are categorized by severity (critical → important → minor) and incorporated directly. You read the plan only after it's been validated.

## Review perspectives

| Perspective | What it checks |
|---|---|
| Codebase alignment | Do referenced files/functions/APIs actually exist? Current patterns? Reusable utilities? |
| Best practices | SOLID, DRY, YAGNI, error handling, security (OWASP top 10), performance |
| Project standards | Naming conventions, file structure, test patterns, commit style, tech stack |
| Feasibility & risks | Missing steps, dependency ordering, underestimated complexity, blockers |
| Fresh perspective | Edge cases, blind spots, simpler alternatives, operational concerns |
| Research & facts | API/library accuracy, version compatibility, protocol correctness |
| Design & UX | Visual consistency, accessibility, responsive behavior, loading/error states |
| Data & schema | Migration safety, indexing, data integrity, query performance |
| Security | Auth flows, input validation, secrets handling, attack surface |
| Operations | Rollback strategy, monitoring, configuration management, failure modes |

Only relevant perspectives are selected — a CLI tool plan won't get a UX review agent.

## Installation

```bash
claude plugin marketplace add https://github.com/kojott/claude-replan
claude plugin install replan@claude-replan
```

### Updating

```bash
claude plugin marketplace update claude-replan
claude plugin update replan@claude-replan
```

## Usage

```
/replan              # Review the plan in current conversation
/replan path/to/plan.md   # Review a plan file
```

That's it. One command, multiple parallel reviewers, updated plan.

## Requirements

- Claude Code with subagent support (Agent tool)
- Works best with Opus model for review quality

## License

MIT + Commons Clause — free to use, modify, and share. Commercial selling requires author's approval. See [LICENSE](LICENSE) for details.

# claude-replan

A Claude Code plugin for plan validation and post-implementation review. `/replan` validates plans before execution, `/recheck` verifies the implementation matches the plan afterward. Both use parallel review subagents checking from multiple angles.

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

## /recheck — post-implementation review

You've finished implementing a plan. `/recheck` verifies the code actually matches what was planned, plus runs parallel quality/security/test reviews.

```
You: /recheck

Claude: Reviewing the implementation against the plan with parallel subagents...

  Agent 1 (Plan compliance)     → Did the code match the plan? Missing steps? Scope creep?
  Agent 2 (Code quality)        → DRY, dead code, complexity, naming
  Agent 3 (Security)            → OWASP top 10, injection, auth, secrets
  Agent 4 (Test coverage)       → New paths tested? Edge cases? Meaningful assertions?
  Agent 5 (Fresh perspective)   → Reads code cold: "What breaks first in production?"

  ✓ All agents returned.

  Plan Compliance: 8/9 items implemented, 1 partial
  Critical: Missing input validation on user-facing endpoint (plan item #4)
  Important: No test for error path in retry logic
  Minor: Naming inconsistency in helper function

  → Want me to fix the critical/important issues?
```

### Key differences from /replan

| | /replan | /recheck |
|---|---|---|
| **When** | Before implementation | After implementation |
| **Reviews** | The plan document | The code + diff |
| **Primary focus** | Feasibility, correctness, missing steps | Plan compliance, code quality, security |
| **Auto-fixes** | Updates plan directly | Presents findings, user decides |

### What /recheck checks

| Perspective | What it looks for |
|---|---|
| Plan compliance | Each plan item implemented? Deviations? Scope creep? |
| Code quality | DRY, dead code, complexity, naming, error handling |
| Security | OWASP top 10, injection, auth, secrets, access control |
| Test coverage | New paths tested? Edge cases? Meaningful assertions? |
| Performance | N+1 queries, O(n²), missing caching, unbounded ops |
| Fresh perspective | "What breaks first in production?" |
| Project standards | CLAUDE.md conventions, naming, structure |

## When to use /recheck vs /code-review

| Tool | Best for |
|---|---|
| `/recheck` | Validating implementation matches a plan. Multi-perspective parallel review (security, tests, performance, quality). Plan compliance is the primary focus. |
| `/code-review` | General code review for bugs, logic errors, and CLAUDE.md compliance. Single-pass review without plan context. |
| `superpowers:requesting-code-review` | Quick single-agent review before merging. |

**Use /recheck when:** You had a plan, you executed it, and you want to verify nothing was missed before calling it done.

**Use /code-review when:** You want a general code review without a specific plan to check against.

**Use both when:** You want plan compliance verification (/recheck) AND a separate detailed bug review (/code-review).

## Usage

```
/replan              # Review the plan in current conversation
/replan path/to/plan.md   # Review a plan file
/recheck             # Review implementation against the plan
```

One command, multiple parallel reviewers.

## Requirements

- Claude Code with subagent support (Agent tool)
- Works best with Opus model for review quality

## License

MIT + Commons Clause — free to use, modify, and share. Commercial selling requires author's approval. See [LICENSE](LICENSE) for details.

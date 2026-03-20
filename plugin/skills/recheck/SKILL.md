---
name: recheck
description: Post-implementation review — validates that code matches the original plan and launches parallel subagents for quality, security, and test coverage checks. Use after completing implementation or when user invokes /recheck. Differentiates from /code-review by focusing on plan compliance and multi-perspective parallel validation.
allowed-tools: [Agent, Read, Glob, Grep, Bash]
---

# Post-Implementation Review (/recheck)

You've finished implementing a plan. Your job is to validate that the code matches the plan and passes multi-perspective quality checks by dispatching parallel review subagents.

**Announce:** "Reviewing the implementation against the plan with parallel subagents..."

## Step 1: Determine Scope

Figure out what changed and what the plan was.

### Find the plan

- Check the current conversation for the plan that was executed
- If the user provides a file path, read the plan from that file
- If no plan is visible, ask: "Which plan should I check this implementation against? Paste it or give me a file path."

### Find the changes

Try these in order:

1. **User-specified files/range** → use exactly that
2. **Git repo exists** → run `git log --oneline -10` to understand recent history, then:
   - For uncommitted changes: `git diff` and `git diff --cached`
   - For committed work: `git diff origin/main...HEAD` (or appropriate base branch)
   - Combine both if there are staged + unstaged changes
3. **No git repo** → ask user for the list of files to review

### Edge cases — handle before proceeding

- **Empty diff** → "No changes detected — nothing to review." Stop here.
- **>30 files changed** → "This touches [N] files. To give a thorough review, can you narrow the scope? For example: specific directories, a commit range, or the most critical files." Wait for user response.
- **<5 lines changed, single file** → Skip parallel dispatch. Run a single comprehensive review agent that covers plan compliance + code quality + security in one pass. Present findings and stop.

### Gather context

- Read changed files in full (not just diffs — agents need surrounding context)
- Read CLAUDE.md / project conventions if they exist
- Collect the diff output for line-level review

## Step 2: Design Review Agents

Based on what changed, select which review perspectives are needed. The **plan compliance** agent is always primary — this is what makes /recheck different from /code-review.

**Pick from these perspectives:**

| Perspective | When to include | What it checks |
|---|---|---|
| **Plan compliance** | Always (primary agent) | Did implementation match the plan? Missing steps? Scope creep? Deviations? Were all plan items addressed? |
| **Code quality** | Always | DRY, dead code, complexity, reuse opportunities, unnecessary abstractions, naming clarity |
| **Security** | Code touches user input, auth, file I/O, exec, eval, network | OWASP top 10: injection, broken auth, sensitive data exposure, XXE, broken access control, misconfig, XSS, insecure deserialization, known vulnerabilities, insufficient logging |
| **Test coverage** | Always for code changes | Are new code paths tested? Edge cases covered? Meaningful assertions? Missing test scenarios? |
| **Performance** | Loops, DB queries, API calls, data processing | N+1 queries, unnecessary allocations, missing caching, O(n²) algorithms, unbounded operations |
| **Fresh perspective** | Always | Reads the code cold: "Does this actually work? What breaks first in production? What did the implementer assume that isn't guaranteed?" |
| **Project standards** | CLAUDE.md / linting config / conventions exist | Naming, file structure, patterns, error handling conventions, import ordering, test structure |

**Rules for agent count:**
- Minimum 3 agents: plan compliance + code quality + fresh perspective
- Always include **plan compliance** — this is the primary value of /recheck
- Always include **fresh perspective** — catches what others miss
- Scale up to 7 with complexity and scope
- Each agent has a distinct, non-overlapping focus

## Step 3: Dispatch All Agents in Parallel

Launch all review agents simultaneously using the Agent tool. Each agent gets:

1. **The list of changed files and diff content**
2. **Full file contents** for key changed files (agents need surrounding context, not just diffs)
3. **The original plan text** (critical for plan compliance agent)
4. **Project context** — CLAUDE.md contents, conventions, tech stack
5. **A specific review mandate** with inline criteria
6. **The output format** below

### Plan Compliance Agent Prompt

```
You are reviewing a code implementation to verify it matches the original plan. This is the PRIMARY review — plan compliance is the unique value of this check.

## The Original Plan
[FULL PLAN TEXT]

## Changed Files
[LIST OF FILES WITH DIFFS]

## Full File Contents
[KEY CHANGED FILES IN FULL]

## Your Review Mandate
Go through the plan point by point:
1. For each plan item/task/step, verify it was implemented
2. Flag any plan items that were NOT implemented or were only partially done
3. Flag any code that was added but was NOT in the plan (scope creep)
4. Check that the implementation approach matches what the plan specified
5. Verify any specific requirements the plan called out (error handling, edge cases, etc.)

Be precise — reference specific plan items and specific code locations (file:line).

## Output Format
### Plan Compliance Review

**Verdict: PASS | ISSUES FOUND | CONCERNS**

**Plan Items Status:**
- [ ] or [x] [Plan item] — [status: implemented / missing / partial / deviated] — [file:line if implemented]

**Critical Issues** (plan violations that must be addressed):
- [issue]: [which plan item] → [what's wrong] → [what should be done]

**Scope Creep** (code added beyond the plan):
- [file:line]: [what was added] — [is this justified?]

**Recommendations** (improvements aligned with plan intent):
- [recommendation]: [why] → [suggested approach]

**Observations** (informational):
- [observation]
```

### Code Quality Agent Prompt

```
You are reviewing code changes for quality issues. Focus on the CHANGED code, not pre-existing patterns.

## Changed Files
[LIST OF FILES WITH DIFFS]

## Full File Contents
[KEY CHANGED FILES IN FULL]

## Your Review Mandate
Check the changed code for:
1. DRY violations — is code duplicated that should be extracted?
2. Dead code — anything added but never called?
3. Unnecessary complexity — over-engineering, premature abstractions?
4. Naming — do names accurately describe what things do?
5. Error handling — are errors caught, propagated, and handled appropriately?
6. Code organization — are things in the right files/modules?

Only flag issues in the CHANGED code. Do not review unchanged surrounding code.

## Output Format
### Code Quality Review

**Verdict: PASS | ISSUES FOUND | CONCERNS**

**Critical Issues** (bugs, logic errors, broken functionality):
- [file:line]: [issue] → [why it matters] → [suggested fix]

**Recommendations** (should fix):
- [file:line]: [issue] → [why it matters] → [suggested approach]

**Observations** (informational):
- [observation]
```

### Security Agent Prompt

```
You are reviewing code changes for security vulnerabilities. Apply OWASP top 10 and general security best practices.

## Changed Files
[LIST OF FILES WITH DIFFS]

## Full File Contents
[KEY CHANGED FILES IN FULL]

## Your Review Mandate
Check for:
1. Injection (SQL, command, LDAP, XPath) — any user input reaching queries/commands without sanitization?
2. Broken authentication — weak session handling, credential exposure?
3. Sensitive data exposure — secrets in code, unencrypted sensitive data, excessive logging?
4. XML External Entities — unsafe XML parsing?
5. Broken access control — missing authorization checks, IDOR?
6. Security misconfiguration — debug enabled, default credentials, unnecessary features?
7. Cross-Site Scripting — unescaped user content in output?
8. Insecure deserialization — untrusted data deserialized?
9. Known vulnerable components — outdated dependencies with CVEs?
10. Insufficient logging — security events not logged?

Also check: path traversal, race conditions, SSRF, open redirects.

Only flag issues you're confident about. False positives waste time.

## Output Format
### Security Review

**Verdict: PASS | ISSUES FOUND | CONCERNS**

**Critical Issues** (exploitable vulnerabilities):
- [file:line]: [vulnerability type] — [how it could be exploited] → [fix]

**Recommendations** (hardening):
- [file:line]: [concern] → [suggested mitigation]

**Observations** (informational):
- [observation]
```

### Test Coverage Agent Prompt

```
You are reviewing whether the code changes have adequate test coverage.

## Changed Files
[LIST OF FILES WITH DIFFS]

## Full File Contents
[KEY CHANGED FILES IN FULL]

## Your Review Mandate
1. Identify all new code paths, branches, and error cases in the changed code
2. Check if corresponding tests exist for these paths
3. Evaluate test quality — are assertions meaningful or just smoke tests?
4. Identify missing edge case tests
5. Check that error paths are tested, not just happy paths
6. Verify test naming describes the scenario being tested

## Output Format
### Test Coverage Review

**Verdict: PASS | ISSUES FOUND | CONCERNS**

**Untested Code Paths:**
- [file:line]: [code path] — [suggested test scenario]

**Weak Tests:**
- [test file:line]: [issue with test] → [how to improve]

**Missing Edge Cases:**
- [scenario]: [why it matters] → [suggested test]

**Observations** (informational):
- [observation]
```

### Performance Agent Prompt

```
You are reviewing code changes for performance issues.

## Changed Files
[LIST OF FILES WITH DIFFS]

## Full File Contents
[KEY CHANGED FILES IN FULL]

## Your Review Mandate
Check for:
1. N+1 query patterns — loops that trigger individual DB/API calls
2. Unnecessary memory allocation — creating large objects/arrays unnecessarily
3. Missing caching — repeated expensive computations or fetches
4. O(n²) or worse algorithms — nested loops over potentially large collections
5. Unbounded operations — no limits on result sets, file reads, or iterations
6. Blocking operations — sync I/O in async contexts, missing concurrency

Only flag issues where performance impact is material, not micro-optimizations.

## Output Format
### Performance Review

**Verdict: PASS | ISSUES FOUND | CONCERNS**

**Critical Issues** (will cause visible performance problems):
- [file:line]: [issue] — [expected impact] → [fix]

**Recommendations** (should improve):
- [file:line]: [concern] → [suggested optimization]

**Observations** (informational):
- [observation]
```

### Fresh Perspective Agent Prompt

```
You are reviewing this code with completely fresh eyes. You haven't been part of any discussion about it. Look at the code cold and ask yourself:

- Does this code actually do what the variable names and function names suggest?
- What breaks first when this hits production?
- Are there assumptions baked in that aren't guaranteed?
- What happens with unexpected input, network failures, or concurrent access?
- Is there a simpler way to achieve the same thing?
- What will the person maintaining this code in 6 months curse about?

## Changed Files
[LIST OF FILES WITH DIFFS]

## Full File Contents
[KEY CHANGED FILES IN FULL]

## Your Review Mandate
Be constructively critical. You're the last line of defense. The goal is to find what everyone else missed — the "oh no" moments that happen in production, not in code review.

Focus on:
1. Logic errors that would pass a surface read
2. Hidden assumptions (hardcoded values, expected state, timing dependencies)
3. Error scenarios that aren't handled
4. Concurrency or race conditions
5. Things that work in dev but break at scale or in prod

## Output Format
### Fresh Perspective Review

**Verdict: PASS | ISSUES FOUND | CONCERNS**

**Critical Issues** (will break in production):
- [file:line]: [what will happen] → [why] → [fix]

**Concerns** (might break, depends on context):
- [file:line]: [concern] → [what to verify]

**Observations** (not bugs, but worth noting):
- [observation]

If everything looks solid, say PASS and explain what gives you confidence.
```

### Project Standards Agent Prompt

```
You are reviewing code changes for compliance with project conventions and standards.

## Changed Files
[LIST OF FILES WITH DIFFS]

## Full File Contents
[KEY CHANGED FILES IN FULL]

## Project Conventions
[CLAUDE.md CONTENTS AND/OR DETECTED CONVENTIONS]

## Your Review Mandate
Check that changed code follows project conventions:
1. Naming conventions (variables, functions, files, classes)
2. File/directory structure patterns
3. Error handling patterns used elsewhere in the codebase
4. Import ordering and module structure
5. Test file naming and organization
6. Documentation/comment style
7. Any explicit rules from CLAUDE.md or project configuration

Only flag deviations from established patterns. Don't invent new standards.

## Output Format
### Project Standards Review

**Verdict: PASS | ISSUES FOUND | CONCERNS**

**Issues** (convention violations):
- [file:line]: [violation] — [convention] → [fix]

**Observations** (informational):
- [observation]
```

## Step 4: Synthesize Findings

Once all agents return:

1. **Collect all findings** — read every agent's review
2. **Deduplicate** — if multiple agents flagged the same issue, merge into one finding with the most specific file:line reference
3. **Categorize by severity:**
   - **Critical**: Bugs, security vulnerabilities, broken functionality, unimplemented plan items
   - **Important**: Missing error handling, untested paths, plan deviations, performance issues
   - **Minor**: Style, naming, minor convention deviations
4. **Present structured summary:**

```
## Implementation Review Results

### Agents Dispatched
| Agent | Verdict |
|---|---|
| Plan Compliance | PASS / ISSUES FOUND / CONCERNS |
| Code Quality | ... |
| ... | ... |

### Critical Issues
1. [file:line]: [issue] — found by [agent]

### Important Issues
1. [file:line]: [issue] — found by [agent]

### Minor Issues
1. [file:line]: [issue] — found by [agent]

### What's Solid
- [positive finding from agents — what was done well]

### Plan Compliance Summary
- [X/Y plan items fully implemented]
- [any deviations or scope creep noted]
```

5. **Ask:** "Want me to fix the critical/important issues?"

## Step 5: Fix If Requested

If the user wants fixes:

- Dispatch a **separate fix agent** for each critical/important issue (or group related issues)
- Each fix agent gets: the specific issue, the file contents, and clear instructions
- The fix agent uses Edit/Write tools — the user reviews and approves each change
- Do NOT auto-fix without user confirmation — code changes are harder to reverse than plan updates

## What /recheck Is NOT

- It's not `/code-review` — that checks code for bugs and CLAUDE.md compliance
- It's not a linter — use your project's actual linter for style enforcement
- It's not a replacement for tests — agents can identify missing tests, but can't replace running them

**/recheck's unique value is plan compliance** — verifying that what was built matches what was planned, while also catching quality/security issues through parallel multi-perspective review.

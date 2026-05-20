---
name: code-review
description: Review pull requests and code changes with concise, issues-only feedback. Use when user asks for a code review, PR review, "review this PR", "review my changes", or wants feedback on a diff or branch.
---

# Code Review

## Critical rules

**1. NEVER run tests.** No `npm test`, `pytest`, `cargo test`, `go test`, or any test runner. Your job is reviewing for missing coverage — running tests belongs to CI/CD.

**2. Approve when no BLOCKER issues.** If the PR has only IMPORTANT or NIT issues (or none), you MUST approve via `--approve` with all issues in the body. Only use `--request-changes` when at least one BLOCKER exists. On re-review: if all issues are fixed or acceptably declined, approve — even if you previously requested changes.

**3. Follow the format exactly.** Every issue uses:
```
**[Severity] [Category] `file:line`** — one-line summary
> what's wrong, why it matters, concrete fix
```
No greetings, closings, summaries, or praise. Issues only.

Example:
```
**[BLOCKER] [Correctness] `src/auth.ts:42`** — missing token expiry check
> `verifyToken()` never checks `payload.exp`. An expired token silently passes auth. Add expiry check after the verify call.
```

## Workflow

### 1. Gather the diff
```bash
gh pr diff <number>
gh pr list --head <branch> --json number -q '.[0].number' | xargs gh pr diff
git diff $(git merge-base origin/HEAD HEAD)..HEAD
```
If >10 files, prioritize largest hunks, core logic, and security-sensitive paths.

### 2. Read changed files
Read relevant sections around each change — not the entire file. Read imports only when a change touches their interface. Read callers only when a signature changes. Read files in parallel when independent.

### 3. Review checklist
Skip categories with no genuine findings.

**Correctness** — logic errors, off-by-one, inverted conditions, missing null/error handling, race conditions, security (injection, auth bypass, exposed secrets, input validation)

**Design** — unnecessary complexity, tangled responsibilities, duplicated logic, violations of codebase patterns

**Missing tests** — new logic without tests, uncovered edge cases (empty, null, boundary, error), untested integration points

**Missing context** — PR description has no issue/design doc link AND no `Self-contained` marker. IMPORTANT if non-trivial and motivation isn't obvious from the diff; NIT if self-explanatory. Do NOT flag if `Self-contained` is present.

**Better ideas** — simpler algorithm or data structure, existing utility the author missed, more idiomatic pattern

**Nits** — misleading names, dead code, leftover comments, stray logs, loose/tight types, unhelpful error messages

### 4. Format and post

Write issues to `review.md`. Every issue:
```
**[Severity] [Category] `file:line`** — one-line summary
> what's wrong, why it matters, concrete fix
```

Severities: `BLOCKER` (must fix), `IMPORTANT` (should fix), `NIT` (nice to fix).
Categories: `[Correctness]`, `[Design]`, `[Missing test]`, `[Missing context]`, `[Better idea]`, `[Nit]`.
Cross-cutting: `**[Severity] [Category] Multiple files** — summary` with file list.

Verify format, then post:
```bash
gh pr review <number> --request-changes --body "$(cat review.md)"   # if BLOCKER
gh pr review <number> --approve --body "$(cat review.md)"            # if no BLOCKER
gh pr review <number> --approve --body "LGTM. No issues found."      # if empty
```
If `gh` isn't authenticated, ask the user to authenticate. Fall back to printing.

### 5. Re-review after updates

Author may fix or decline issues using `[Declined] [Reason]` replies. Evaluate each:

| Reason | How to evaluate |
|---|---|
| `[Already handled]` | Verify the claim. If true, accept. If not, explain why it's still an issue. |
| `[Out of scope]` | Accept if truly out of scope, suggest follow-up. If not, explain why it belongs here. |
| `[Convention]` | Accept if convention exists. If author is mistaken, cite the relevant standard. |
| `[Follow-up filed]` | Accept if issue linked. Ask for number if missing. |
| `[Disagree]` | Re-read code. Push back once with new evidence. If still a matter of opinion, defer to author. |

**NITs never block.** Author's call is final. Comment if you want, but approve.

**One pushback max on IMPORTANTs.** If author declines and you push back with evidence, that's your round. If they still disagree, defer and approve. Never re-request changes for the same IMPORTANT twice.

**After evaluating:** if all issues are fixed, acceptably declined, or at final-disagreement, approve. Only re-request changes for an unfixed BLOCKER.

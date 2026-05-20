---
name: code-review
description: Review pull requests and code changes with concise, issues-only feedback. Use when user asks for a code review, PR review, "review this PR", "review my changes", or wants feedback on a diff or branch.
---

# Code Review

## Critical rules

**1. NEVER run tests.** Do not execute any test runner. Your job is to review for missing test coverage — that's it. Running tests belongs to CI/CD.

**2. Approve when there are no BLOCKER issues.** If the PR has only IMPORTANT or NIT issues (or none), approve with `--approve` and all issues in the body. Only `--request-changes` when at least one BLOCKER exists.

On re-review: if the author fixed issues or gave reasonable `[Declined]` explanations (see code-author skill), approve. Only keep `--request-changes` for an unfixed BLOCKER. See [REFERENCE.md](REFERENCE.md) for how to evaluate each decline reason.

**3. Follow the format exactly.** Every issue uses:

```
**[Severity] [Category] `file:line`** — one-line summary
> what's wrong, why it matters, concrete fix
```

Example:

```
**[BLOCKER] [Correctness] `src/auth.ts:42`** — missing token expiry check
> `verifyToken()` never checks `payload.exp`. An expired token silently passes. Add `if (payload.exp < Date.now()/1000) throw new TokenExpiredError()`.

**[IMPORTANT] [Missing test] `src/auth.ts:42-58`** — no test for expired token rejection
> The new expiry check has no coverage. Add a case in `auth.test.ts` with an expired token asserting 401.
```

## Principles

- **Issues only** — no praise, no summaries, no greetings. Only problems and suggestions.
- **Approve when clean** — no BLOCKER = approve. IMPORTANT and NIT go in the approval body.
- **Detailed issues** — every finding has file:line, severity, category, and a concrete fix.
- **Post to PR** — use `gh pr review`. Fall back to printing only if posting isn't possible.

## Workflow

### 1. Gather the diff

```bash
gh pr diff <number>
gh pr list --head <branch> --json number -q '.[0].number' | xargs gh pr diff
git diff $(git merge-base origin/HEAD HEAD)..HEAD
```

If the diff is large (>10 files), prioritize files with the largest hunks and security-sensitive paths.

### 2. Read changed files

Read relevant sections around each change — not entire files. Read enough context to understand the logic. Read in parallel when files are independent.

### 3. Review checklist

Scan for these categories. Skip any with no genuine findings.

**Correctness** — logic errors, off-by-one, missing null/error handling, race conditions, security (injection, auth bypass, exposed secrets)

**Design** — unnecessary complexity, missing separation of concerns, duplicated logic, violations of codebase patterns

**Missing tests** — new logic without tests, edge cases uncovered (empty, boundary, error), untested integration points

**Better ideas** — simpler algorithm/library, existing utility the author missed, more idiomatic pattern

**Nits** — misleading names, dead code, leftover comments, stray logs, loose type annotations, unhelpful error messages

### 4. Format the review

```
**[Severity] [Category] `file:line`** — one-line summary
> what's wrong, why it matters, concrete fix
```

Severities: `BLOCKER` (must fix), `IMPORTANT` (should fix), `NIT` (nice to fix).
Categories: `[Correctness]`, `[Design]`, `[Missing test]`, `[Better idea]`, `[Nit]`.

Cross-cutting issues: `**[Severity] [Category] Multiple files** — summary` with affected files listed.

Write the output to `review.md`.

### 5. Verify format before posting

Check every line in `review.md`: starts with `**[`, has severity, has `[Category]`, has `` `file:line` `` or `Multiple files`. No greetings, closings, or praise.

### 6. Post to PR

```bash
# BLOCKER present → request changes:
gh pr review <number> --request-changes --body "$(cat review.md)"

# Only IMPORTANT and/or NIT → approve:
gh pr review <number> --approve --body "$(cat review.md)"

# No issues → approve:
gh pr review <number> --approve --body "LGTM. No issues found."
```

If `gh` is not authenticated, use `gh auth status` to check.

### 7. Re-review after author updates

When the author pushes fixes and replies, re-evaluate each issue. Verify fixes, evaluate declined explanations per [REFERENCE.md](REFERENCE.md).

**NITs can never block approval.** The author's call is final on nits.

**One round of pushback max on IMPORTANTs.** Push back once with evidence, then defer — the author owns the code.

After evaluating: if every issue is fixed or acceptably declined, approve. Only re-request changes for an unfixed BLOCKER.

---
name: code-review
description: Review pull requests and code changes with concise, issues-only feedback. Each issue is detailed with file references, root cause, and suggested fix. Use when user asks for a code review, PR review, "review this PR", "review my changes", or wants feedback on a diff or branch.
---

# Code Review

## Principles

- **Issues only** — never list what's good. The review is exclusively problems and suggestions.
- **Detailed issues** — each finding includes file/line references, root cause, risk, and a concrete fix or alternative.
- **Concise overall** — no preamble, no praise, no summaries. Jump straight to issues.
- **Do not run tests** — assume the author or CI/CD runs them. Review for missing test coverage, not test execution.
- **Suggest better ideas** — when a simpler, safer, or more idiomatic approach exists, say what and why.
- **Always post to the PR** — use `gh pr review` or `gh pr comment` so feedback lands on the PR.

## Workflow

### 1. Gather the diff

If the user names a PR number or branch, fetch the diff:

```bash
gh pr diff <number>           # GitHub PR
gh pr diff <branch>           # or by branch
git diff origin/main...HEAD   # local branch vs base
```

If no PR is specified, diff the working tree against the merge base.

### 2. Read the changed files

For each changed file, read the full file (not just the diff) to understand context. Look at surrounding code, imported modules, and callers.

### 3. Review with this checklist

Scan for these categories. List only issues found — skip categories that are clean.

**Correctness**
- Logic errors, off-by-one, inverted conditions, missing null/error handling
- Race conditions, async ordering, stale closures
- Security: injection, auth bypass, exposed secrets, input validation gaps

**Design**
- Unnecessary complexity or abstraction that could be simpler
- Missing separation of concerns, tangled responsibilities
- Duplicated logic that should be shared
- Violations of existing codebase patterns or conventions

**Missing tests**
- New logic without corresponding test cases
- Edge cases the tests don't cover (empty input, boundary values, error paths)
- Integration points that need tests (API calls, DB queries, external services)

**Better ideas**
- A simpler algorithm, data structure, or library function that replaces the current approach
- An existing utility or helper in the codebase that the author may have missed
- A more idiomatic pattern for the language/framework

**Nits**
- Misleading names, dead code, leftover comments, stray logs
- Type annotations that are too loose or too tight
- Error messages that don't help the reader diagnose the problem

### 4. Format the review

Each issue follows this compact template:

```
**`file:line`** — <one-line summary>
> <1–3 sentences: what's wrong, why it matters, concrete fix>
```

Group related issues under a category header only when there are ≥3 findings in that category. Otherwise, list them flat.

No greeting, no closing, no "overall this looks good" — issues only.

### 5. Post to PR

```bash
gh pr review <number> --comment --body "$(cat review.md)"
# or for inline comments:
gh pr review <number> --request-changes --body "$(cat review.md)"
```

If `gh` is not authenticated, use `gh auth status` to check and ask the user to authenticate. Fall back to printing the review if posting isn't possible.


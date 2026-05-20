---
name: code-review
description: Review pull requests and code changes with concise, issues-only feedback. Use when user asks for a code review, PR review, "review this PR", "review my changes", or wants feedback on a diff or branch.
---

# Code Review

## Critical rules — read before anything else

These rules override everything below. Violating any of them breaks the review.

**1. NEVER run tests.** Do not execute `npm test`, `pytest`, `cargo test`, `go test`, or any test runner. Not even to "check if they pass." Not even a single file. Your job is to review for missing test coverage — that's it. Running tests wastes time, may not be configured for your environment, and belongs to CI/CD.

**2. Approve whenever there are no BLOCKER issues.** This is the single most important rule. If the PR has only IMPORTANT or NIT issues (or no issues at all), you MUST approve it. Use `--approve` with the review body containing all issues. This clears any previous "changes requested" status that would otherwise block merge. IMPORTANT and NIT issues are communicated in the approval body — the author decides whether to address them. Only use `--request-changes` when there is at least one BLOCKER.

This applies to re-reviews too. When an author pushes fixes and replies to your previous issues with `[Declined]` explanations, evaluate each one. If the explanation is reasonable, resolve the thread and move on — a declined IMPORTANT with a good reason is no longer an issue. If all remaining issues are either fixed or acceptably declined, approve. Only keep `--request-changes` if the author left a BLOCKER unfixed or gave an unacceptable explanation for one.

**3. Follow the format exactly.** Every issue MUST use the template below. Do not invent your own format. Do not add greetings, closings, summaries, or praise. The format is:

```
**[Severity] [Category] `file:line`** — one-line summary
> what's wrong, why it matters, concrete fix
```

Example of a correctly formatted review with one BLOCKER and one IMPORTANT:

```
**[BLOCKER] [Correctness] `src/auth.ts:42`** — missing token expiry check
> `verifyToken()` returns `payload` but never checks `payload.exp`. An expired token silently passes authentication. Add `if (payload.exp < Date.now()/1000) throw new TokenExpiredError()` after the verify call.

**[IMPORTANT] [Missing test] `src/auth.ts:42-58`** — no test for expired token rejection
> The new expiry check has no test coverage. Add a case in `auth.test.ts` that passes an already-expired token and asserts a 401 response.
```

## Principles

- **Issues only** — never list praise or summaries. The review is exclusively problems and suggestions.
- **Approve when clean** — if there are no BLOCKER issues, approve. Do not hold the PR for IMPORTANT or NIT issues.
- **Skip empty categories** — only include a category (Correctness, Design, Better ideas, etc.) when you have real findings in it.
- **Detailed issues** — each finding includes file/line references, severity, category tag, and a concrete fix.
- **Prefer posting to the PR** — use `gh pr review` so feedback lands on the PR. Fall back to printing only if posting isn't possible.
- **Focus on material issues** — correctness, maintainability, and performance. Skip trivial formatting nits unless they violate project conventions.

## Workflow

### 1. Gather the diff

```bash
gh pr diff <number>                            # GitHub PR by number or URL
gh pr list --head <branch> --json number -q '.[0].number' | xargs gh pr diff  # by branch
git diff $(git merge-base origin/HEAD HEAD)..HEAD   # local branch vs merge base (works regardless of default branch name)
```

If the diff is large (>10 files), prioritize files with the most significant changes (largest hunks, core logic, security-sensitive paths) rather than reading every file.

### 2. Read changed files

Read the relevant sections around each change in the diff — not the entire file. For each hunk, read enough surrounding context to understand the logic. Read imported modules only when a change touches their interface. Read callers only when a signature changes.

Read files in parallel when they are independent.

### 3. Review with this checklist

Scan for these categories. Skip any category that has no genuine findings — an empty category is noise.

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

Each issue MUST follow this exact template:

```
**[Severity] [Category] `file:line`** — one-line summary
> what's wrong, why it matters, concrete fix
```

Severities: `BLOCKER` (must fix before merge), `IMPORTANT` (should fix), `NIT` (nice to fix).

Categories: `[Correctness]`, `[Design]`, `[Missing test]`, `[Better idea]`, `[Nit]`.

For cross-cutting issues spanning multiple files, use `**[Severity] [Category] Multiple files** — summary` with a list of affected files.

No greeting, no closing, no "overall this looks good" — issues only. No markdown headings for categories — just the issue lines.

Write the output to `review.md`.

### 5. Verify format before posting

Before posting, check every issue line in `review.md` against the template:

- Does every issue start with `**[` followed by a severity and `]**`?
- Does every issue have a `[Category]` tag?
- Does every issue have a `` `file:line` `` reference (or `Multiple files`)?
- Are there no greetings, closings, or praise lines?

### 6. Post to PR

The command depends on the highest severity found:

```bash
# BLOCKER present → request changes (blocks merge):
gh pr review <number> --request-changes --body "$(cat review.md)"

# Only IMPORTANT and/or NIT → approve (clears previous blocks, author decides on remaining issues):
gh pr review <number> --approve --body "$(cat review.md)"

# No issues at all → approve with a brief note:
gh pr review <number> --approve --body "LGTM. No issues found."
```

Key rule: **approve unless there is a BLOCKER.** A prior reviewer's "request changes" must not block the PR permanently. Your approval clears it while still communicating all remaining issues.

If `gh` is not authenticated, use `gh auth status` to check and ask the user to authenticate. Fall back to printing the review if posting isn't possible.

### 7. Re-review after author updates

When the author pushes new commits and replies to your review, re-review the PR. The author may have:

- **Fixed issues** — verify the fix actually resolves the problem. Mark the thread resolved.
- **Declined issues** — the author used a `[Declined]` reply format. Evaluate the explanation on its merits, not on whether you would have made the same call.

**How to handle `[Declined]` replies:**

The author's reason tag tells you what to weigh:

| Reason | How to evaluate |
|---|---|
| `[Already handled]` | Verify the claim. If the caller/upstream does handle it, accept. If not, explain why it's still an issue. |
| `[Out of scope]` | Is this really out of scope for the PR? If yes, accept and suggest a follow-up issue. If no, explain why it belongs here. |
| `[Convention]` | Check whether the convention exists and applies. If it's a real convention, accept. If the author is mistaken, cite the relevant standard. |
| `[Follow-up filed]` | Check that a follow-up issue exists and is linked. If yes, accept. If the issue number is missing, ask for it. |
| `[Disagree]` | This is the only reason that may warrant pushing back. The author disagrees with your assessment. Re-read the code — did you misread it? If you still believe the issue stands, explain why with new evidence. If it's genuinely a matter of opinion, defer to the author. |

**NITs can never block approval.** Even if you disagree with how the author handled a NIT, approve anyway. The author's call is final on nits — comment if you want, but don't hold the PR.

**One round of pushback max on IMPORTANTs.** If the author declines an IMPORTANT and you push back with concrete evidence, that's your one round. If the author still disagrees after your pushback, defer to the author and approve. Do not re-request changes for the same IMPORTANT twice. A second round of back-and-forth is unproductive — the author owns the code.

**After evaluating all replies:** if every issue is either fixed, acceptably declined, or declined-with-final-disagreement (one round exhausted), approve. Even if you previously requested changes, your new approval clears that block. Only re-request changes if a BLOCKER remains unfixed.

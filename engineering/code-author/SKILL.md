---
name: code-author
description: Address code review feedback, follow design plans, add tests, and respond to PR comments. Use when the user says "address the review", "fix the PR feedback", "implement the plan", "respond to review comments", or needs to act on a code review.
---

# Code Author

## Critical rules — read before anything else

**1. Follow the plan.** If a design doc, spec, or execution plan exists, read it first. Implement exactly what it specifies — no scope creep, no unrelated refactors. If the plan is ambiguous, ask before deviating.

**2. Fix all BLOCKER and IMPORTANT issues.** These are non-negotiable. Read the review (`review.md`, PR comments, or inline feedback) and address every BLOCKER and IMPORTANT finding before touching anything else.

**3. Fix NITs when they improve the code.** Nits are suggestions, not demands. Fix them if they make the code clearer, safer, or more idiomatic. Skip them if they're purely stylistic disagreements or would cause disproportionate churn. When in doubt, fix it — a nit costs little to address.

**4. Always add tests for new or changed code.** Every new function, branch, and code path gets a test. Use the TDD skill if available (`/tdd`) — write the test first, watch it fail, then implement. Minimum coverage: happy path + edge cases (empty input, boundary values, error paths). If a BLOCKER or IMPORTANT issue flagged missing tests, those tests are mandatory.

**5. Respond to unfixed issues.** For any issue you choose not to fix (some NITs, or an IMPORTANT you disagree with), post a reply on the PR explaining why. Be specific — "this would require changing the public API and is out of scope for this PR" is better than "won't fix." Silence reads as oversight.

## Principles

- **Plan-driven** — the design doc or spec is the source of truth. Implementation follows the plan, not improvisation.
- **Review-first** — address every piece of feedback before adding new work. Unresolved review threads block progress.
- **Test discipline** — no new code ships without tests. Prefer TDD when the skill is available.
- **Communicate decisions** — every unfixed issue gets a response. The reviewer should never wonder "did they miss this or disagree?"
- **Minimal scope** — fix what the review asks for. Don't bundle unrelated cleanups, refactors, or "while I'm here" changes.

## Workflow

### 1. Read the plan

If the PR references a design doc, spec, or issue, read it. Understand what the code is supposed to do before changing what it does.

```bash
gh pr view <number>        # PR description, linked issues, design doc references
gh issue view <number>     # linked issue details
```

### 2. Read the review

Collect all feedback sources:

```bash
gh pr diff <number>                              # the changes under review
gh pr view <number> --comments                   # inline review comments
gh api repos/owner/repo/pulls/<number>/reviews   # full reviews (request-changes, approve)
```

If a `review.md` file exists locally, read it — it may contain the latest round of feedback.

### 3. Triage and fix

Process issues in severity order:

- **BLOCKER** — fix immediately. These are correctness, security, or data-loss issues that must not merge.
- **IMPORTANT** — fix before marking work as done. These are design flaws, missing tests, or maintainability problems that degrade the PR materially.
- **NIT** — fix if it clearly improves the code. If it's debatable or expensive, skip it and explain why in a PR reply.

For each fix:
- Make the minimal change that addresses the issue.
- If multiple issues overlap in the same code, fix them together to avoid churn.
- Verify the fix compiles and is syntactically correct.

### 4. Add tests

Every new or changed code path gets a test. If the TDD skill is available, invoke it:

```
/tdd
```

Otherwise, follow TDD principles manually:
- Write the test first. Define the expected behavior.
- Run it to confirm it fails (red).
- Write the implementation that makes it pass (green).
- Refactor if needed while keeping tests green.

Coverage checklist:
- Happy path — the primary use case works
- Edge cases — empty input, null values, boundary conditions, error states
- Integration points — API calls, DB queries, external services (mock as needed)
- Review-flagged gaps — any missing test the review explicitly called out

### 5. Respond to unfixed issues

For each issue you chose not to fix, post a PR reply. Every reply MUST follow this template:

```
**[Declined] [Reason] `file:line`** — one-line summary of the declined issue
> why this isn't being fixed, what the trade-off is, and any follow-up
```

Reasons: `[Out of scope]`, `[Already handled]`, `[Convention]`, `[Follow-up filed]`, `[Disagree]`.

Examples of correctly formatted replies:

```
**[Declined] [Already handled] `src/auth.ts:42`** — redundant null check
> The caller at `service.ts:120` already validates the token before this function is reached. Adding a second check here is defensive but doesn't improve safety.
```

```
**[Declined] [Follow-up filed] `src/utils.ts:88`** — extract shared helper
> This would require refactoring three other callers across the codebase. Filed #1234 to track this as a separate cleanup PR.
```

```
**[Declined] [Convention] `src/api.ts:15`** — non-standard error format
> This matches the error shape the team agreed on in RFC-42. Changing it here would break consistency with the other 12 endpoints.
```

Never reply with just "won't fix" or "disagree" — always include the reason in the `>` block.

Post replies via:

```bash
# Reply to an inline comment:
gh api repos/owner/repo/pulls/<number>/comments/<comment_id>/replies \
  -f body="<formatted reply>"

# Reply to a review thread:
gh api repos/owner/repo/pulls/<number>/reviews/<review_id>/comments \
  -f body="<formatted reply>"
```

### 6. Push changes

Commit each logical fix group separately so the reviewer can follow your work:

```bash
git add <files>
git commit -m "fix: address review — <brief description of what was fixed>"
git push
```

After pushing, the PR updates automatically. The reviewer sees the new commits and your inline replies.

### 7. Merge when ready

The PR is ready to merge when ALL of these are true:

- **All reviewers have approved.** Every requested reviewer has submitted an approving review (not just "commented" — a green checkmark).
- **No unresolved BLOCKER issues.** Every BLOCKER has been fixed and the fix accepted by the reviewer who raised it.
- **IMPORTANT issues are resolved.** Each IMPORTANT is either fixed, or declined with an explanation. If the reviewer pushes back once and you still disagree, state your final position — the issue is now resolved in your favor and you may proceed. One round of pushback is healthy; two is a deadlock.
- **NITs don't block.** Nits that were fixed are fine. Nits that were declined with an explanation don't block merge — the reviewer can accept or ignore the explanation; silence on a declined nit after a reasonable wait means implicit acceptance.

If any reviewer still has "requested changes" status, do not merge — push another round of fixes or replies and wait for them to re-review.

Merge via:

```bash
gh pr merge <number> --squash     # or --merge, --rebase per project convention
```

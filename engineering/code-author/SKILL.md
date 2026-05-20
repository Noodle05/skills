---
name: code-author
description: Address code review feedback, follow design plans, add tests, and respond to PR comments. Use when the user says "address the review", "fix the PR feedback", "implement the plan", "respond to review comments", or needs to act on a code review.
---

# Code Author

## Critical rules

**1. Follow the plan.** If a design doc, spec, or issue exists, read it first. Implement exactly what it specifies — no scope creep. If ambiguous, ask.

**2. Fix all BLOCKER and IMPORTANT issues first.** These are non-negotiable. Read the review and address every BLOCKER and IMPORTANT before anything else.

**3. NITs are optional.** Fix them if they improve the code. Skip if purely stylistic or disproportionate. When in doubt, fix it.

**4. Always add tests.** Every new function, branch, and code path gets a test. Use `/tdd` if available. Minimum: happy path + edge cases + error paths. Review-flagged missing tests are mandatory.

**5. Respond to every unfixed issue.** Post a PR reply explaining why. Silence reads as oversight.

**6. Link the work.** Every PR description must include an issue/design doc link, or a `Self-contained` marker so the reviewer knows the absence is intentional.

## Workflow

### 1. Read the plan
```bash
gh pr view <number>        # PR description, linked issues, design docs
gh issue view <number>     # linked issue details
```

### 2. Read the review
```bash
gh pr diff <number>
gh pr view <number> --comments
gh api repos/owner/repo/pulls/<number>/reviews
```
Also read `review.md` if it exists locally.

### 3. Triage and fix

Process in severity order:
- **BLOCKER** — correctness, security, data-loss. Fix immediately.
- **IMPORTANT** — design flaws, missing tests, maintainability. Fix before done.
- **NIT** — fix if it clearly improves the code. Otherwise skip and explain.

Make minimal changes. Fix overlapping issues together to avoid churn.

### 4. Add tests

Use `/tdd` if available. Otherwise: write test → see it fail → implement → refactor.
Cover: happy path, edge cases (empty, null, boundary), integration points, review-flagged gaps.

### 5. Respond to unfixed issues

Template:
```
**[Declined] [Reason] `file:line`** — one-line summary
> why this isn't being fixed, what the trade-off is, and any follow-up
```
Reasons: `[Out of scope]`, `[Already handled]`, `[Convention]`, `[Follow-up filed]`, `[Disagree]`.

Example:
```
**[Declined] [Already handled] `src/auth.ts:42`** — redundant null check
> The caller at `service.ts:120` already validates the token. Adding a second check is defensive but doesn't improve safety.
```

Post via:
```bash
gh api repos/owner/repo/pulls/<number>/comments/<comment_id>/replies -f body="<reply>"
gh api repos/owner/repo/pulls/<number>/reviews/<review_id>/comments -f body="<reply>"
```

### 6. Push
```bash
git add <files>
git commit -m "fix: address review — <brief description>"
git push
```

### 7. Merge when ready

- All reviewers approved (green checkmark, not just "commented")
- Every BLOCKER fixed and accepted by the reviewer who raised it
- Every IMPORTANT fixed or declined with explanation (one round of pushback max, then author's call)
- NITs don't block — declined with explanation is fine
- If any reviewer still has "requested changes", do NOT merge

```bash
gh pr merge <number> --squash     # or --merge, --rebase per project convention
```

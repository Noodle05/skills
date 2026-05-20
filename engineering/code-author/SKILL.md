---
name: code-author
description: Address code review feedback, follow design plans, add tests, and respond to PR comments. Use when the user says "address the review", "fix the PR feedback", "implement the plan", "respond to review comments", or needs to act on a code review.
---

# Code Author

## Critical rules

**1. Follow the plan.** Read the design doc, spec, or execution plan first. Implement exactly what it specifies — no scope creep. If ambiguous, ask.

**2. Fix all BLOCKER and IMPORTANT issues.** Address every BLOCKER and IMPORTANT from the review (`review.md`, PR comments, or inline feedback) before anything else.

**3. Fix NITs when they improve the code.** Skip if purely stylistic or disproportionately expensive — but explain why in a PR reply.

**4. Always add tests.** Every new function, branch, and code path gets a test. Use `/tdd` if available. Minimum: happy path + edge cases (empty, null, boundary, error).

**5. Respond to unfixed issues.** Post a PR reply for every issue you don't fix. Silence reads as oversight.

**6. Always create a PR.** No work is done until it's in a PR. If one doesn't exist for this branch, create it before finishing.

## Workflow

### 1. Read the plan

If an existing PR, pull its context. If implementing from a spec without a PR yet, read the spec file directly.

```bash
gh pr view <number>        # existing PR: description, linked issues
gh issue view <number>     # linked issue details
```

### 2. Read the review

Check `review.md` first — this is the output from the code-review skill. Then pull PR feedback:

```bash
gh pr diff <number>
gh pr view <number> --comments
gh api repos/owner/repo/pulls/<number>/reviews
```

### 3. Triage and fix

Process in severity order: BLOCKER (correctness, security) → IMPORTANT (design, missing tests) → NIT (clarity, style). Make minimal changes. Fix overlapping issues together to avoid churn.

### 4. Add tests

Every changed code path gets a test. Invoke `/tdd` if available. Coverage: happy path, edge cases, integration points, review-flagged gaps.

### 5. Respond to unfixed issues

For each issue not fixed, post a PR reply using this template:

```
**[Declined] [Reason] `file:line`** — one-line summary
> why this isn't being fixed, the trade-off, and any follow-up
```

Reasons: `[Out of scope]`, `[Already handled]`, `[Convention]`, `[Follow-up filed]`, `[Disagree]`.

Example:
```
**[Declined] [Follow-up filed] `src/utils.ts:88`** — extract shared helper
> Would require refactoring three callers. Filed #1234 as follow-up.
```

Post replies via:
```bash
gh api repos/owner/repo/pulls/<number>/comments/<comment_id>/replies -f body="<reply>"
```

### 6. Push changes

```bash
git add <files>
git commit -m "fix: address review — <brief description>"
git push
```

### 7. Create PR (if needed)

```bash
# Check if PR exists
gh pr list --head $(git branch --show-current) --json number --jq '.[0].number'

# If not, create one:
gh pr create \
  --title "<concise title>" \
  --body "$(cat <<'EOF'
## Summary
- <what was done and why>
## Related
- Plan/issue: <link>
🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### 8. Merge when ready

Only merge when: all reviewers approved, no unresolved BLOCKERs, every IMPORTANT is fixed or declined. NITs don't block. If any reviewer still has "requested changes," do not merge.

```bash
gh pr merge <number> --squash
```

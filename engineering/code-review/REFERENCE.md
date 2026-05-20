# Code Review — Reference

## Handling `[Declined]` replies on re-review

When the author (see code-author skill) pushes fixes and replies with `[Declined]` explanations, evaluate each reason type:

| Reason | How to evaluate |
|---|---|
| `[Already handled]` | Verify the claim. If the caller/upstream does handle it, accept. If not, explain why it's still an issue. |
| `[Out of scope]` | Is this really out of scope for the PR? If yes, accept and suggest a follow-up issue. If no, explain why it belongs here. |
| `[Convention]` | Check whether the convention exists and applies. If it's a real convention, accept. If the author is mistaken, cite the relevant standard. |
| `[Follow-up filed]` | Check that a follow-up issue exists and is linked. If yes, accept. If the issue number is missing, ask for it. |
| `[Disagree]` | Re-read the code — did you misread it? If you still believe the issue stands, explain why with new evidence. If it's genuinely a matter of opinion, defer to the author. |

**NITs can never block approval.** The author's call is final on nits — comment if you want, but don't hold the PR.

**One round of pushback max on IMPORTANTs.** If the author declines and you push back with concrete evidence, that's your round. If they still disagree, defer — the author owns the code. Do not re-request changes for the same IMPORTANT twice.

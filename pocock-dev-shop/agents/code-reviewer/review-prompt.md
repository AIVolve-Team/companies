# TASK

Review the sub-issue's branch and fix what you find, in place, on that branch.

Read the sub-issue with `GET /api/issues/{issueId}/heartbeat-context` and read the thread — the Staff
Engineer's hand-off comment names the seams tested and the decisions taken. Those are review material,
not background.

The `## Branches` section of the description names the branch to review and the batch branch it will
merge into.

# THE FIXED POINT

The `code-review` skill reviews a diff against a fixed point you supply. Here the fixed point is the
merge-base between the sub-issue branch and the **batch branch** — the changes this sub-issue is about
to contribute, and nothing the batch already carries.

```
git diff pocock/batch-<identifier>...pocock/issue-<identifier>
git log pocock/batch-<identifier>..pocock/issue-<identifier> --oneline
```

Confirm both refs resolve and the diff is non-empty before going further. An empty diff means the
branch has nothing on it: say so on the issue and stop — do not review the batch by accident.

# THE SPEC AXIS

The skill looks for the originating spec through the repo's issue tracker. Here the spec is the
Paperclip sub-issue you are already holding, plus its parent. You have both — pass them to the review
as the spec rather than sending the skill hunting for `docs/agents/issue-tracker.md`. If that file is
missing from the repo, that is expected in this company and is not a reason to stop.

# THE REVIEW

Run the `code-review` skill on that diff. Both of its axes apply:

- **Standards** — does the code follow what this repo documents, plus the skill's smell baseline
- **Spec** — does it actually do what the sub-issue asked, with the edge cases handled and the new
  behaviour covered by tests

Beyond the skill's own checks, look for what makes the next reader slower: unnecessary nesting,
redundant abstractions, names that hide what they hold, nested ternaries where a switch or an if-chain
would read plainly, comments restating the code. And look for what makes it unsafe: unchecked casts,
`any`, swallowed errors, credentials in the diff, unvalidated input reaching a query or a shell.

Keep the balance. Do not flatten a helpful abstraction, do not merge concerns to save a file, and do
not trade clarity for brevity. A change that makes the code shorter and harder to follow is not an
improvement.

# FIX IT

You fix findings; you do not report them upward.

1. Make the changes on the sub-issue branch
2. Preserve behaviour exactly — every output and every effect stays the same. If a finding cannot be
   fixed without changing behaviour, it is not yours to fix: see below
3. Run typecheck and tests again after your changes
4. Commit your changes separately from the Staff Engineer's, so the batch history shows which hunks
   came from review

If the branch is already clean, change nothing. An empty review is a legitimate outcome.

# WHAT IS NOT A REVIEW FINDING

Two things do not belong in a fix commit:

- **The ticket was built against the wrong requirement.** Nothing you can rewrite makes it right.
- **The fix requires a decision the ticket never scoped** — a schema change, a new dependency, a
  redesign that reaches past this diff.

Either of those makes the sub-issue `blocked`: say what is wrong, name who has to decide, and stop.
Do not reassign it to the Staff Engineer — this company has no review bounce, and a reassignment
without a decision behind it just moves the problem.

# HAND OFF

When the branch is clean and green, hand the sub-issue to the Release Engineer:

`PATCH /api/issues/{issueId}` with `assigneeAgentId` set to the Release Engineer's agent id, and a
comment recording what you found, what you changed, and what you deliberately left alone. If you
found nothing, say that — an unexplained silent pass is indistinguishable from a review that never ran.

Reassignment wakes the Release Engineer. Do not also comment asking them to merge.

Do not close the sub-issue.

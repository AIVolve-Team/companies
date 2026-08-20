# TASK

Two jobs, and which one you are doing depends on who woke you.

- **A Code Reviewer handed you a sub-issue** → merge that one branch into the batch branch, then hand
  the batch to the QA Engineer.
- **The QA Engineer handed you a passing batch** → push and open the pull request.

Read the issue with `GET /api/issues/{issueId}/heartbeat-context` and the thread before either. The
`## Branches` section names the sub-issue branch and the batch branch.

# PART A — INCREMENTAL MERGE

One sub-issue per run. Merging several at once is what this company deliberately does not do: when the
gate later fails, the merge commit has to point at one sub-issue and one engineer.

1. Make sure the batch branch exists. The first time a batch is merged, create it from the base branch
   the repo merges into, at its current head.
2. Check out the batch branch and merge the sub-issue branch:
   `git merge pocock/issue-<identifier> --no-edit`
3. **On conflict**, use the `resolving-merge-conflicts` skill. Its rules hold here: read why each side
   changed, preserve both intents where they are compatible, pick the side matching the batch's goal
   where they are not and note the trade-off, never invent behaviour, and never `--abort`. Always
   resolve.
4. Run the project's typecheck and tests on the batch branch after the merge. Discover the commands
   from the repo rather than assuming them.
5. If the batch branch is red after the merge and the sub-issue branch was green on its own, the merge
   itself broke it — that is yours to fix, on the batch branch, before you go any further. If you
   cannot, mark the sub-issue `blocked` and say what the merge broke.

Keep the merge commit's message pointing at the sub-issue identifier. That commit is what the QA
Engineer will read backwards from when something fails later.

**Do not push in Part A.** The batch branch stays local until QA has passed it.

## Hand off to the gate

`PATCH /api/issues/{issueId}` with `assigneeAgentId` set to the QA Engineer's agent id, and a comment
naming the sub-issue merged, the merge commit, and every sub-issue the batch branch now carries. That
list matters — the QA Engineer needs it to attribute a failure.

# PART B — PUSH, AND OPEN THE PULL REQUEST ONCE

Only when the QA Engineer hands the batch back with a passing gate. You hold `GH_TOKEN`; use it here
and nowhere else.

**Push first, every time.** This part runs once per sub-issue that passes the gate, not once per
batch, so the push is the step that repeats. Push the batch branch to the remote before you look at
pull requests at all.

Then decide whether a pull request needs opening, in this order, stopping at the first step that says
no:

1. **Is the batch branch ahead of the base branch?** If it carries nothing the base does not, there is
   nothing to open. Stop.
2. **Is there already an open pull request for this head?** If yes, the push you just made has already
   updated it — GitHub does that by itself. Do not open a second one, and do not go looking for an
   "update pull request" action; there is none. Refresh the body as below, then stop.
3. **Otherwise open it.** Exactly one pull request, from the batch branch to the base branch.

That order is the whole guard, and the order is the point: checking for an existing pull request only
*after* trying to open one is how a second `gh pr create` fails on a batch's second pass.

## The body, rewritten on every pass

The body lists the sub-issues **currently merged into the batch branch** — not the whole batch, which
may still have sub-issues in flight — and says the gate passed for those. When sub-issues are still
outstanding, say so and name them.

On a later pass, rewrite that list rather than appending to it. A pull request that reads as a
finished batch while half of it is still being built is worse than one that says where it actually is.

A human merges it. You do not.

## Close only what you shipped

Close each sub-issue whose work is in the branch you just pushed and that is not closed already:
`PATCH /api/issues/{issueId}` with `status: "done"` and a comment linking the pull request.

Never close a sub-issue that is not in the branch — one the CTO deferred, or one still with a Staff
Engineer. The pull request closes what it carries, nothing more.

## Record the pull request

The first time you open it, create a `pull_request` work product on the batch's parent issue pointing
at the pull request URL. A comment alone is not enough — the work product is the access path a human
opens, and the bundled `paperclip` skill requires one for an opened pull request. On later passes that
work product already points at the same pull request; leave it alone.

# IF YOU CANNOT PUSH

If `GH_TOKEN` is missing or rejected, that is a `blocked` batch, not something to work around: say the
credential failed, name what needs to happen, and exit. Never fall back to a different remote,
a different credential, or a different branch.

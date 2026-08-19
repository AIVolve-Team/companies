# TASK

Schedule the batch. Read the tickets hanging off the spec assigned to you, work out which of them can
start now, split what is too large, and hand each piece to a Staff Engineer on a branch whose name you
can derive again tomorrow and get the same answer.

# 1. READ THE BATCH

Get your assignments from `GET /api/agents/me/inbox-lite`. What lands on you is **one spec issue per
batch**, and the batch is its children: `GET /api/issues/{specId}/children`. Read the spec for the
shared context — the repo, the base branch, how the project is verified — and each child for the work.

The spec itself is not work. Never implement it, never assign it to anyone, and never close it: it is
the batch's container, and the identifier the batch branch is named after.

One wake, one scheduling pass. Paperclip wakes an agent **per assignment**, so a batch whose tickets
were each assigned to you directly would start one scheduling run per ticket, and those runs would
race — two of them deciding the same ticket's blockers and assignee at the same time. That is why the
batch arrives as a spec with children instead.

A child that is already assigned to another agent, or whose status is not `todo`, has been scheduled
already — by you on an earlier heartbeat, or by whoever is working it. Leave it alone. Your batch this
run is the children still sitting unassigned or assigned to you.

For each ticket in your batch, read enough to know what it touches:
`GET /api/issues/{issueId}/heartbeat-context`, plus the description.

# 2. BUILD THE DEPENDENCY GRAPH

Ticket B is **blocked by** ticket A when:

- B needs code or infrastructure that A introduces
- B and A change overlapping files or modules, so working on both at once would produce merge
  conflicts
- B depends on a decision or an API shape that A establishes

A ticket is **unblocked** when nothing else in the batch blocks it.

Record what you find as real blockers, not prose: `PATCH /api/issues/{issueId}` with
`blockedByIssueIds` set to the ticket IDs that block it. The array replaces the whole set, so send
the complete list each time. A blocked ticket also gets `status: "blocked"`. This matters — a
first-class blocker wakes the assignee by itself when it clears, and a sentence in a comment does
not.

# 3. DECOMPOSE WHAT IS TOO LARGE

A ticket belongs in more than one piece when parts of it can be built independently — different
files, different modules, no shared decision pending between them. If a ticket only splits into
"first this, then that", it is one piece; sequence is not parallelism.

Create each piece with `POST /api/companies/{companyId}/issues`, setting `parentId` to the ticket
you are splitting, `goalId` to whatever the parent carries, and **`projectId` to the parent's
`projectId`**. Put everything the implementer needs in the description — a sub-issue must be readable
on its own, because the agent picking it up may not be able to read the parent.

`projectId` is not paperwork: the project is what carries the execution workspace policy, so a
sub-issue created without it runs in whatever workspace the agent falls back to, sharing a checkout
with every other run instead of getting its own worktree.

Do not decompose for the sake of it. A ticket that one engineer can finish in one pass stays whole.

# 4. NAME THE BRANCHES

Two names, both derived, never invented:

- **Sub-issue branch** — `pocock/issue-<identifier>`, using the issue's own identifier lowercased
  (`PAP-142` → `pocock/issue-pap-142`). One branch per sub-issue.
- **Batch branch** — `pocock/batch-<identifier>`, using the identifier of the spec issue the batch
  hangs from (`POC-1` → `pocock/batch-poc-1`). One branch for the whole batch.

Derived names mean re-scheduling the same work twice finds the branch that already exists, with its
commits, instead of starting a second empty one.

Write both names into every sub-issue description, under a heading the next agent will find:

```
## Branches

- Work on: pocock/issue-pap-142
- Merges into: pocock/batch-pap-140
```

# 5. ASSIGN

Assign each unblocked sub-issue to a Staff Engineer: `PATCH /api/issues/{issueId}` with
`assigneeAgentId` set to the Staff Engineer's agent id and `status: "todo"`. Look up agent ids from
your chain of command (`GET /api/agents/me`).

Assignment is the hand-off — it wakes the Staff Engineer on its own. Do not also comment asking them
to start.

# 6. WHEN EVERYTHING IS BLOCKED

If no ticket in the batch is unblocked, the batch is deadlocked on a dependency you recorded. Pick
the single candidate with the fewest and weakest dependencies, schedule that one, and say in a
comment which dependency you chose to run ahead of and why. Never end a heartbeat having scheduled
nothing when tickets are waiting.

# BEFORE YOU EXIT

Leave the batch in a state that reads correctly without you:

- every sub-issue you created is assigned, has both branch names in its description, and is `todo`
- every dependency you found is a `blockedByIssueIds` edge, and every blocked ticket says `blocked`
- the tickets you scheduled are off your own plate — reassigned, not left assigned to you
- the spec issue stays assigned to you and open: it is how a later heartbeat picks up what you
  deferred
- a comment on the spec listing what you scheduled, what you deferred, and the batch branch name

Then set your own disposition and exit. You do not wait for the pipeline.

---
name: Pocock Dev Shop
description: Executor company that implements already-decided tickets — CTO schedules, Staff Engineers build test-first, a Code Reviewer fixes in place, a Release Engineer merges incrementally, and a QA Engineer gates the batch — driven by referenced skills from mattpocock/skills
slug: pocock-dev-shop
schema: agentcompanies/v1
version: 0.1.0
license: MIT
authors:
  - name: Simone
goals:
  - Take a batch of already-planned tickets from implementation to a single reviewed pull request without human intervention
  - Stay an executor only — never decide *what* to build, only *how* to execute it
  - Keep every run isolated so parallel sub-issues cannot corrupt each other's working tree
---

Pocock Dev Shop is an execution-only engineering company. It receives tickets that someone else
already decided — scoped, written up, and queued — and carries them to a pull request that a human
merges. It never opens the question of what should be built.

Everything upstream of implementation lives outside this company: discovery, specification, and
ticket-writing happen in a human-driven session, because the skills that do that work are
user-invoked and no agent can trigger them. What arrives here is a batch of tickets, and what
leaves is one pull request per batch.

## Org chart

```
                          CTO
                           |
     +---------------+-----+------+------------------+
     |               |            |                  |
Staff Engineer  Code Reviewer  Release Engineer  QA Engineer
```

Five agents, flat under the CTO. There is deliberately no CEO: the permissions a `ceo` role would
unlock — exporting and importing the company itself — are human acts here, not agent ones.

## How Work Flows

1. **CTO** is the entry point. A batch of tickets lands on the CTO, who reads dependencies the
   tracker does not know about, picks what can run now, decomposes anything too large into
   parallel sub-issues, and assigns each one a deterministic branch name. The CTO also names the
   **batch branch** — the single branch every sub-issue eventually merges into.
2. **Staff Engineer** implements one sub-issue on its own branch, in an isolated worktree, using
   the `tdd` skill for the red → green loop. Hands off by reassigning the sub-issue to the Code
   Reviewer.
3. **Code Reviewer** reviews the same branch with the `code-review` skill and **fixes what it
   finds in place**. Nothing is sent back to the Staff Engineer over ordinary review findings —
   there is no bounce loop here. Hands off to the Release Engineer.
4. **Release Engineer** merges that one branch into the batch branch — incrementally, one
   sub-issue at a time, resolving conflicts with the `resolving-merge-conflicts` skill. Hands the
   batch branch to the QA Engineer.
5. **QA Engineer** is the only gate: typecheck, tests, and any end-to-end suite, run against the
   batch branch.
   - **Passes** → back to the Release Engineer, who pushes and opens the pull request (guarded so
     a second open never fires), then the work is a human's to merge.
   - **Fails** → back to the Staff Engineer responsible for the failure, identified from the merge
     commit that last touched the failing files. Several Staff Engineers may be sent back in
     parallel. If attribution is genuinely ambiguous, the QA Engineer stops and asks the human
     rather than guessing.

The QA Engineer is the single rework loop in this company. Every other hand-off moves forward.

Skills are partitioned so no two agents hold the same tool: `tdd` to the Staff Engineer,
`code-review` to the Code Reviewer, `resolving-merge-conflicts` to the Release Engineer. The CTO
and the QA Engineer hold none by choice — scheduling and quality gating have no counterpart in the
referenced catalog, and their prompts carry that work instead.

Built for [Paperclip](https://github.com/paperclipai/paperclip) against the [Agent Companies spec](https://agentcompanies.io/specification), referencing engineering skills from [mattpocock/skills](https://github.com/mattpocock/skills)

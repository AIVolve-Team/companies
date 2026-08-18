# Pocock Dev Shop

An **execution-only** engineering company for [Paperclip](https://github.com/paperclipai/paperclip),
built to the [Agent Companies spec](https://agentcompanies.io/specification).

It takes tickets that someone else already decided and carries them to a single pull request a human
merges. It never decides what to build. Discovery, specification, and ticket-writing happen outside
this company, in a human-driven session — the skills that do that work are user-invoked and no agent
can trigger them.

The five roles mirror an autonomous-implementation pipeline, and three of them are driven by
referenced skills from [`mattpocock/skills`](https://github.com/mattpocock/skills).

## Org chart

```
                          CTO
                           |
     +---------------+-----+------+------------------+
     |               |            |                  |
Staff Engineer  Code Reviewer  Release Engineer  QA Engineer
```

Five agents, flat under the CTO, no CEO. The permissions a `ceo` role would unlock — exporting and
importing the company itself — are deliberately human acts here.

| Agent | Role | Model | Skill | Phase prompt |
| --- | --- | --- | --- | --- |
| CTO | `cto` | `claude-opus-5` | — | `plan-prompt.md` |
| Staff Engineer | `engineer` | `claude-sonnet-5` | `tdd` | `implement-prompt.md` |
| Code Reviewer | `engineer` | `claude-sonnet-5` | `code-review` | `review-prompt.md` |
| Release Engineer | `devops` | `claude-sonnet-5` | `resolving-merge-conflicts` | `merge-prompt.md` |
| QA Engineer | `qa` | `claude-sonnet-5` | — | `qa-prompt.md` |

Skills are partitioned so no two agents hold the same tool. The CTO and the QA Engineer hold none by
choice: nothing in the referenced catalog covers batch scheduling or quality gating, and their
prompts carry that work instead. The bundled `paperclip` skill is not declared anywhere — it is
always available to every agent.

## How work flows

1. **CTO** receives a batch of tickets. It records the dependencies the tracker does not know, picks
   what can run now, decomposes what is too large into parallel sub-issues, and gives each one a
   derived branch name plus the single **batch branch** they all merge into.
2. **Staff Engineer** implements one sub-issue on its own branch, test-first with `tdd`.
3. **Code Reviewer** reviews that branch with `code-review` and **fixes findings in place**. There is
   no review bounce in this company.
4. **Release Engineer** merges that one branch into the batch branch, incrementally, resolving
   conflicts with `resolving-merge-conflicts`.
5. **QA Engineer** gates the batch branch — typecheck, tests, end-to-end. Pass goes back to the
   Release Engineer to push and open **one** pull request; fail goes back to the responsible Staff
   Engineer, identified from the merge commit that last touched the failing files.

The QA Engineer is the only rework loop. Every other hand-off moves forward, and every hand-off is a
reassignment — Paperclip wakes the new assignee by itself, so no agent has to nudge another.

When a failure cannot honestly be attributed to one sub-issue, the QA Engineer stops the batch and
asks the human with a `board_only` interaction rather than guessing an owner.

## Agent identity and phase prompts

Each agent directory holds two files:

- `AGENTS.md` — identity: frontmatter (`name`, `title`, `reportsTo`, `role`, `skills`) plus the four
  paragraphs of the catalog stamp (where work comes from, what you do and produce, who you hand off
  to, what triggers you).
- a colocated `*-prompt.md` — the procedure for that phase.

The split is deliberate, and it only works because `AGENTS.md` **says explicitly** to read the prompt
file, by relative path. Every file under `agents/<slug>/` is imported into the agent's managed
instructions bundle and materialized on disk next to `AGENTS.md`, but only `AGENTS.md` reaches the
system prompt. Paperclip mounts that directory read-only and tells the agent to resolve relative
references from it — so `implement-prompt.md` resolves, and `$AGENT_HOME/implement-prompt.md` does
not.

Prompt names come from the sandcastle `parallel-planner-with-review` template
(`plan`/`implement`/`review`/`merge`), so the lineage of each one is recognisable at a glance.
`qa-prompt.md` has no counterpart there and was written for this company.

## Referenced skills

Each skill is a 15-line stub: frontmatter with a `github-file` source pinned to a **40-character
commit SHA**, `usage: referenced`, and a one-line summary. Nothing is vendored or forked.

At import the pin is validated — an unpinned ref is rejected, and a source carrying executable
scripts is rejected outright. When an agent actually reads the skill, Paperclip fetches it live from
`raw.githubusercontent.com` at the pinned commit, falling back to the stub only if that fetch fails.
So the pin is the single point of freeze, and drift toward upstream is a deliberate act: bump the
`commit` field, then re-run the import dry-run to confirm the file still loads at the new SHA.

`diagnosing-bugs` is deliberately **excluded**. It is the one skill in this chain carrying a script
(`scripts/hitl-loop.template.sh`), and Paperclip refuses to import an external skill whose inventory
contains executables.

## Getting started

Import the company package. **The source is positional** — the `--from /path/to/company` form in this
repo's `CONTRIBUTING.md` does not exist in the CLI:

```bash
# preview first
npx paperclipai company import ./pocock-dev-shop \
  --target new --include company,agents,skills --dry-run

# then for real
npx paperclipai company import ./pocock-dev-shop \
  --target new --include company,agents,skills --yes
```

Then three setup steps that do not live in the package:

1. **Worktree isolation.** The default `shared_workspace` policy gives parallel runs the same
   checkout, and this company runs sub-issues in parallel on purpose. Set the project's execution
   workspace policy to `git_worktree`. Paperclip also accepts a per-agent
   `adapter.config.workspaceStrategy: { type: git_worktree }` if you would rather version it here
   than set it per project.
2. **`GH_TOKEN` on the Release Engineer only.** This one is not optional to do by hand. The package
   declares the input but cannot carry a value, and an input imported without a value leaves **no
   trace at all** on the agent — not an empty slot, not a placeholder. After a fresh import the
   Release Engineer's configuration contains no `GH_TOKEN` key, and the company holds no secrets.
   Create the secret and bind it to the Release Engineer yourself, from the UI or the secrets API,
   scoped to the single repository the company is allowed to touch. Nothing pushes until you do,
   and no other agent should ever be given a credential.

   The declaration says `requirement: optional`, which is deliberate. `requirement` is read only by
   the company importer and never by the run preflight, so `required` buys no protection at run
   time — and it makes `paperclipai company import` fail with `422 Required environment values are
   missing`, because the CLI has no way to pass a secret value. Since neither setting produces a
   binding without a value, the manual step above is unavoidable either way. What actually restricts
   the credential is *which agent declares it*, and that is unchanged.
3. **A dedicated host or `HOME`.** On macOS an agent sees the whole filesystem and every skill
   installed for the user, not just this company's — verified on a clean import: alongside its own
   `tdd`, the Staff Engineer listed a dozen skills from the operator's personal Claude home, and the
   pre-run inventory counted 54 of them. Run the agents on a dedicated machine, or at minimum give
   the adapter a dedicated `HOME`, so a run's transcript only ever shows skills that belong here.

## Known rough edges

- **The pre-run skill inventory reports referenced skills as `missing`.** `paperclipai agent skills
  <id>` shows each `github-file` skill as *"in the library, but Paperclip cannot find its local
  source at …"*, with the GitHub URL joined onto the current directory as if it were a path. It is a
  false negative in that view only: the run-time mount works. A verified run materialized all three
  skills under `skills/<companyId>/__runtime__/<slug>--<hash>/SKILL.md`, byte-identical (SHA-256) to
  the upstream file at the pinned commit.
- **`tdd` references two sibling files** (`tests.md`, `mocking.md`) that a `github-file` source does
  not bring along; the agent gets `SKILL.md` alone and those links dangle. The `implement-prompt.md`
  loop is written to stand on its own without them.
- **`code-review` expects `docs/agents/issue-tracker.md`** in the target repository and otherwise
  tells the user to run a setup command. Here the Paperclip sub-issue *is* the spec, so
  `review-prompt.md` hands it to the review directly and says the missing file is expected.
- **The CLI cannot import a package with a `required` env input at all.** Its import command has no
  flag for secret values, so `requirement: required` is unimportable from the terminal regardless of
  which agent declares it. Only the UI import flow, or a direct API call carrying `secretValues`,
  can satisfy one.
- **`tdd` asks the user to confirm the seams under test.** There is no user inside an autonomous run,
  so `implement-prompt.md` substitutes the ticket's acceptance criteria as the agreement and requires
  the engineer to write the seams into the issue thread before the first test.

## References

- [Agent Companies specification](https://agentcompanies.io/specification)
- [Paperclip](https://github.com/paperclipai/paperclip)
- [mattpocock/skills](https://github.com/mattpocock/skills)

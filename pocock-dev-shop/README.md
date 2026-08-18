# Pocock Dev Shop

> Execution-only engineering company that takes already-decided tickets to a single reviewed pull request — CTO schedules, Staff Engineers build test-first, a Code Reviewer fixes in place, a Release Engineer merges incrementally, and a QA Engineer gates the batch

> An [Agent Company](https://agentcompanies.io) built around [mattpocock/skills](https://github.com/mattpocock/skills) — engineering skills for test-driven development, two-axis code review, and merge-conflict resolution

## What's Inside

> This is an [Agent Company](https://agentcompanies.io) package from [Paperclip](https://paperclip.ing)

| Content | Count |
|---------|-------|
| Agents | 5 |
| Skills | 3 |

### Agents

| Agent | Role | Reports To |
|-------|------|------------|
| CTO | CTO | — |
| Code Reviewer | Engineer | cto |
| QA Engineer | QA | cto |
| Release Engineer | DevOps | cto |
| Staff Engineer | Engineer | cto |

### Skills

| Skill | Description | Source |
|-------|-------------|--------|
| code-review | Two-axis review of a diff against a fixed point — Standards and Spec — run as parallel sub-agents. | [github](https://github.com/mattpocock/skills/blob/9c9f36ccd3995266cd675468af71639c8dde1ec5/skills/engineering/code-review/SKILL.md) |
| resolving-merge-conflicts | Resolve an in-progress merge or rebase by reading each side back to its intent, never inventing behaviour. | [github](https://github.com/mattpocock/skills/blob/9c9f36ccd3995266cd675468af71639c8dde1ec5/skills/engineering/resolving-merge-conflicts/SKILL.md) |
| tdd | Test-driven development as a red-green loop that produces tests worth keeping — good tests, seams, anti-patterns. | [github](https://github.com/mattpocock/skills/blob/9c9f36ccd3995266cd675468af71639c8dde1ec5/skills/engineering/tdd/SKILL.md) |

Every source is pinned to that one 40-character commit. Nothing is vendored or forked, and drift
toward upstream is a deliberate act: bump the `commit` field in the stub, then re-run the import
dry-run to confirm the file still loads at the new SHA.

## Getting Started

```bash
npx companies.sh add AIVolve-Team/companies/pocock-dev-shop
```

Or import a local checkout. **The source is positional** — the `--from /path/to/company` form shown
in this repo's `CONTRIBUTING.md` is not implemented by the CLI:

```bash
npx paperclipai company import ./pocock-dev-shop \
  --target new --include company,agents,skills --dry-run

npx paperclipai company import ./pocock-dev-shop \
  --target new --include company,agents,skills --yes
```

Then three setup steps that no company package can carry:

1. **Worktree isolation.** The default `shared_workspace` policy gives parallel runs the same
   checkout, and this company runs sub-issues in parallel on purpose. Set the project's execution
   workspace policy to `git_worktree`. Paperclip also accepts a per-agent
   `adapter.config.workspaceStrategy: { type: git_worktree }` if you would rather version it here than
   set it per project.
2. **`GH_TOKEN` on the Release Engineer only.** This one is not optional to do by hand. The package
   declares the input but cannot carry a value, and an input imported without a value leaves **no
   trace at all** on the agent — not an empty slot, not a placeholder. After a fresh import the
   Release Engineer's configuration contains no `GH_TOKEN` key and the company holds no secrets.
   Create the secret and bind it to the Release Engineer yourself, scoped to the single repository the
   company is allowed to touch. Nothing pushes until you do, and no other agent should ever be given a
   credential.

   The declaration says `requirement: optional`, which is deliberate. `requirement` is read only by
   the company importer and never by the run preflight, so `required` buys no protection at run
   time — and it makes `paperclipai company import` fail with `422 Required environment values are
   missing`, because the CLI has no way to pass a secret value. Since neither setting produces a
   binding without a value, the manual step above is unavoidable either way. What actually restricts
   the credential is *which agent declares it*, and that is unchanged.
3. **A dedicated host or `HOME`.** On macOS an agent sees the whole filesystem and every skill
   installed for the user, not just this company's — verified on a clean import: alongside its own
   `tdd`, the Staff Engineer listed a dozen skills from the operator's personal Claude home, and the
   pre-run inventory counted 54 of them. Run the agents on a dedicated machine, or at minimum give the
   adapter a dedicated `HOME`, so a run's transcript only ever shows skills that belong here.

## How work flows

`COMPANY.md` has the authoritative account under **How Work Flows**. In short: the CTO schedules a
batch onto derived branch names, each Staff Engineer implements one sub-issue test-first on its own
branch, the Code Reviewer fixes what it finds in place, the Release Engineer merges that branch into
the batch branch, and the QA Engineer gates the batch branch. Pass goes back to the Release Engineer
to push and open one pull request; fail goes back to the Staff Engineer the merge history blames.

The QA Engineer is the only rework loop. Every hand-off is a reassignment, which wakes the new
assignee by itself, so no agent has to nudge another.

Models follow the difficulty of the role: Opus for the CTO, who schedules and decomposes; Sonnet for
the other four. `.paperclip.yaml` is the single place that records them.

## Agent identity and phase prompts

Each agent directory holds two files:

- `AGENTS.md` — identity: frontmatter (`name`, `title`, `reportsTo`, `role`, `skills`) plus the
  catalogue's stamp sections (where work comes from, what you do, what you produce, who you hand off
  to, what triggers you).
- a colocated `*-prompt.md` — the procedure for that phase.

The split only works because `AGENTS.md` **says explicitly** to read the prompt file, by relative
path. Every file under `agents/<slug>/` is imported into the agent's managed instructions bundle and
materialized on disk next to `AGENTS.md`, but only `AGENTS.md` reaches the system prompt. Paperclip
mounts that directory read-only and tells the agent to resolve relative references from it — so
`implement-prompt.md` resolves, and `$AGENT_HOME/implement-prompt.md` does not.

Prompt names come from the sandcastle `parallel-planner-with-review` template
(`plan`/`implement`/`review`/`merge`), so the lineage of each one is recognisable at a glance.
`qa-prompt.md` has no counterpart there and was written for this company.

## Where this package departs from the catalogue

- **`role:` in `AGENTS.md` frontmatter.** No other company in the catalogue sets it, but omitting it
  normalizes every agent to `general`, and the literal default `"agent"` emitted by generators is
  rejected outright with a 400. The importer does read `role` from frontmatter, and the accepted enum
  is `ceo, cto, cmo, cfo, security, engineer, designer, pm, qa, devops, researcher, general` — hence
  `cto` on the CTO, `qa` on the QA Engineer, `devops` on the Release Engineer.
- **No `Exported from Paperclip` footer.** Every catalogue company has one because it was produced by
  the exporter. This package was written by hand, and stamping an export line it never went through
  would be claiming a provenance it does not have.
- **No `images/org-chart.png`.** All 16 catalogue companies ship one, and Paperclip does render an org
  chart — but for an imported company that render includes a synthetic `Organization` root plus the
  two built-in agents (`Reflection Coach`, `Summarizer`) that Paperclip provisions in every company and
  refuses to delete. Shipping it would show seven agents and a root that isn't in this package; drawing
  a cleaner one by hand would forge an artifact that looks like tool output. The **Agents** table above
  carries the same hierarchy in its `Reports To` column.

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
  which agent declares it. Only the UI import flow, or a direct API call carrying `secretValues`, can
  satisfy one — and an import that dies on that 422 leaves a half-created company behind.
- **`diagnosing-bugs` is deliberately absent.** It is the one skill in this chain carrying a script
  (`scripts/hitl-loop.template.sh`), and Paperclip refuses to import an external skill whose inventory
  contains executables.

## References

- [Agent Companies specification](https://agentcompanies.io/specification)
- [Paperclip](https://github.com/paperclipai/paperclip)
- [mattpocock/skills](https://github.com/mattpocock/skills)

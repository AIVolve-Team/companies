---
name: code-review
description: Two-axis review of a diff against a fixed point — Standards (does it follow this repo's documented standards, plus a Fowler smell baseline) and Spec (does it do what the originating ticket asked) — run as parallel sub-agents
metadata:
  sources:
    - kind: github-file
      repo: mattpocock/skills
      path: skills/engineering/code-review/SKILL.md
      commit: 9c9f36ccd3995266cd675468af71639c8dde1ec5
      attribution: mattpocock
      license: MIT
      usage: referenced
---

Reviews the diff between HEAD and a fixed point along two independent axes, Standards and Spec, each in its own sub-agent, then aggregates the findings.

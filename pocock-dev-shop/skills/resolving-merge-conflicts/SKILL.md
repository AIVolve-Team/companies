---
name: resolving-merge-conflicts
description: Resolve an in-progress git merge or rebase conflict by reading why each side changed, preserving both intents where compatible, and finishing the merge rather than aborting it
metadata:
  sources:
    - kind: github-file
      repo: mattpocock/skills
      path: skills/engineering/resolving-merge-conflicts/SKILL.md
      commit: 9c9f36ccd3995266cd675468af71639c8dde1ec5
      attribution: mattpocock
      license: MIT
      usage: referenced
---

Read both sides back to their original intent, resolve every hunk without inventing behaviour, run the project's checks, and finish the merge. Never --abort.

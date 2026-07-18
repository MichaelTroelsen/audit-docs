---
description: Audit README/CLAUDE.md against the actual codebase for claims that are no longer true
argument-hint: [path | --quick | --sweep | --issues | --fix | --harvest]
allowed-tools: Skill(audit-docs)
---

Invoke the audit-docs skill for: $ARGUMENTS

No arguments: full audit of the current project.
`--quick`: fast mechanical pass (dead refs, versions, secrets).
`--sweep <dir>`: audit every project under a directory.
`--issues`: file findings as agent-executable GitHub issues (preview + approval first).
`--fix`: apply confirmed findings to the working tree (preview + approval first).
`--harvest`: fold past audits' Learnings sections into `references/` — the skill's feedback loop.

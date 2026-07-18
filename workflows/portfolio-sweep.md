# Workflow: Portfolio Sweep

Audit several projects in one pass and surface patterns that only appear across a portfolio.

<required_reading>
Read now:
1. `references/drift-catalog.md`
2. `references/confidence.md`
3. `references/severity.md`
4. `references/integrity.md` — non-negotiable
</required_reading>

<process>

<step n="1" name="Enumerate projects">
Identify the project roots under the given directory — a project is a directory containing a manifest, a `.git`, or its own `CLAUDE.md`/`README.md`.

```bash
find . -maxdepth 3 \( -name 'package.json' -o -name 'pyproject.toml' -o -name 'go.mod' \
  -o -name '.git' -o -name 'CLAUDE.md' \) -not -path '*/node_modules/*' \
  -not -path '*/.venv/*' | sed 's|/[^/]*$||' | sort -u
```

Watch for **umbrella directories** — a folder that is not itself one project but contains several (each with its own git remote). Treat each child as a separate project and say so; conflating them produces incoherent findings.

Present the list and confirm scope with the user before auditing.
</step>

<step n="2" name="Per-project ground truth">
For each project, capture the reference set: version, branch, remote, last commit, whether it is a git repo at all, manifest, test location.

Do this centrally rather than delegating it — it is cheap, and the results are needed to adjudicate subagent claims.
</step>

<step n="3" name="Delegate reading, retain judgment">
Spawn one Explore agent per project (or per 2–3 small projects). Instruct each to:
- Read every in-scope document **in full**
- Report claims verbatim with file and line
- Verify against that project's own code where it can
- Report only what it checked, and mark anything it could not verify

Run them concurrently in a single message.

Adjudicate in the main thread against the ground-truth set from step 2. A subagent reporting "README says v3.13, CLAUDE.md says v3.21" is a claim, not a finding — the finding requires knowing which the code agrees with.

Assign confidence during adjudication, not delegation. A subagent's certainty is not evidence; treat every claim as LOW until the main thread has seen the verification. Absence claims from subagents need the absence protocol re-run centrally — they are the most likely to be wrong.
</step>

<step n="4" name="Per-project findings">
Rank each project's findings with `references/severity.md`. Write a `DOC-AUDIT.md` into each audited project root.
</step>

<step n="5" name="Cross-project patterns">
This is the step that justifies a sweep. Look for:

- **A drift class recurring across projects** — the same failure mode everywhere means one habit is the cause, not several accidents
- **Conventions stated in one project that should be global** — a commit-message rule, a directory rule, a naming standard living in exactly one `CLAUDE.md`
- **Duplicated tooling config** — the same slash command or settings block copied into several repos, already diverging
- **Projects with meaningful work outside version control**
- **Inconsistent maturity** — one project's docs are rigorous about limitations while others make unqualified claims; name the good one as the template
- **Shared infrastructure documented differently** in each project that uses it

Recommend a root-level `CLAUDE.md` when three or more projects would benefit from the same rule.
</step>

<step n="6" name="Portfolio summary">
Write `DOC-AUDIT-PORTFOLIO.md` at the sweep root containing:
- One table row per project: findings by severity, worst finding, whether under version control
- The cross-project patterns from step 5
- A prioritized cross-project action list

In conversation, lead with the cross-project patterns — the per-project details are on disk. The patterns are what the user cannot see from inside any single repo.
</step>

</process>

<success_criteria>
- [ ] Project list confirmed with the user; umbrella directories decomposed
- [ ] Ground truth established centrally per project
- [ ] Every project's documents read in full
- [ ] Subagent claims adjudicated in the main thread, not accepted as findings
- [ ] Per-project `DOC-AUDIT.md` written
- [ ] Cross-project patterns identified, not just concatenated per-project results
- [ ] `DOC-AUDIT-PORTFOLIO.md` written
- [ ] Conversation summary leads with patterns
</success_criteria>

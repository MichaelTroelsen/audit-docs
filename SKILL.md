---
name: audit-docs
description: Audits documentation (README.md, CLAUDE.md, docs/) against the actual codebase to find claims that are no longer true — dead file references, stale version numbers, wrong test counts, broken commands, duplicated facts that have drifted apart. Use when documentation is suspected of being out of date, before publishing or releasing a project, when onboarding to an unfamiliar repo, or when a project's docs and code have evolved separately.
---

<essential_principles>

<principle name="Verify, never trust">
A documentation claim is **not evidence of itself**. Every finding must be confirmed against the filesystem, the code, or a command's real output.

- Doc says a script exists → run `ls` on that path.
- Doc says version 2.4 → read the version from the manifest that actually ships (`pyproject.toml`, `package.json`, `__init__.py`, `go.mod`).
- Doc says "43 tests" → find where that number is produced (`EXPECTED_PASS=`, a test runner, a CI config).
- Doc says a feature is missing → grep for it. Features get built and gap-tables get forgotten.

Never report a discrepancy between two documents without determining **which one the code agrees with**. "A says X, B says Y" is half a finding; "A says X, B says Y, the code says Y, so A is wrong" is a whole one.
</principle>

<principle name="Default to not-a-finding">
If a claim cannot be verified with a concrete check, it is **not** reported. Unverifiable claims (design rationale, historical notes, judgment calls, aspirational roadmap) are out of scope.

A report of 12 confirmed findings is worth more than 40 findings where 28 are guesses. False positives cost the user more time than missed issues, because each one must be manually disproved.
</principle>

<principle name="Every finding carries a confidence level">
HIGH, MEDIUM, or LOW — describing the strength of the **verification method**, not how true the finding feels. LOW-confidence findings are never reported; they are escalated with a second method or moved to Unverifiable. Only HIGH-confidence mechanical findings are eligible for auto-fix.

**Proving absence is far harder than proving presence.** A pattern that matches proves existence; a pattern that matches nothing proves only that your pattern found nothing. Measured on a real audit, naive single-pattern searches produced wrong "missing" verdicts **one time in three**. Any absence claim must survive the protocol in `references/confidence.md` — multiple independent patterns, a positive control, and inspection of real matches — before it may be reported.
</principle>

<principle name="A check that did not run is not a clean result">
An empty result means "clean" only if the check demonstrably executed. If the extraction step matched nothing, emptiness proves nothing at all.

Confirm every check produced tokens before reporting it clean. Note that `rg -c` prints nothing — not `0` — when there are no matches, which silently turns a broken check into an apparently passing one.
</principle>

<principle name="Never report a verification you did not perform">
Every command shown in a report or issue must have been **executed, in this session, against this repo**, with its real output captured. Every quotation must be byte-identical to its source. Every line number must have been printed before it was cited. Every count must come from counting.

Never name a flag, script, path, or config key that has not been observed to exist. This is the skill's most likely failure mode, because an audit report *looks* rigorous whether or not the commands behind it ran — and it has already happened once: a draft issue proposed `--print-urls`, a flag that does not exist, written because it was the right shape for a verification step. It was caught before filing; the next one might not be.

Fabricated evidence is worse than a missed finding. It turns the audit from a check into a new source of false claims — the exact disease it treats. Full rules in `references/integrity.md`.
</principle>

<principle name="Say what was not checked">
If files were skipped, sampled, truncated, or delegated without adjudication, say so. Never write "read in full" unless every file was read to its end. An audit that silently narrows its scope while reading as comprehensive is itself a false claim.

Uncertainty is stated, not smoothed. "I could not determine whether CI runs these tests" is a legitimate output; choosing confident phrasing because it reads better is fabrication with good manners.
</principle>

<principle name="Read files in full">
Drift hides in the parts of a document nobody scrolls to — the troubleshooting FAQ, the version history at the bottom, the fourth copy of a table. Read every target file completely before reporting. Partial reads produce partial audits that feel complete.
</principle>

<principle name="Duplicated truth is the root cause">
Most documentation drift is one fact stored in several places, where only some copies got updated. When the same value appears in 2+ locations, report the **duplication itself** as the finding — not just the mismatched copies. The fix is a single canonical source, not a synchronized edit that will drift again.
</principle>

<principle name="Actively misleading beats merely wrong">
Rank by what a reader would *do* with the bad information. A stale "not yet implemented" note that makes someone rebuild a working feature outranks a typo in a version badge. See `references/severity.md`.
</principle>

</essential_principles>

<intake>
Ask the user:

**What would you like to audit?**

1. **This project** — full audit of one repo's docs against its code
2. **Quick check** — fast pass for dead references and version mismatches only
3. **Portfolio sweep** — audit several projects and report across all of them
4. **File issues** — turn findings into GitHub issues an agent can act on later
5. **Apply fixes** — edit the files directly, after an approval gate
6. **Harvest learnings** — fold past audits' Learnings sections into `references/`

If the user's invocation already makes the intent clear (e.g. "audit the docs in this repo", "check all my projects", `--issues`, `--fix`), skip the question and route directly.

**Wait for a response before proceeding** if intent is genuinely ambiguous.
</intake>

<routing>
| Response | Workflow |
|----------|----------|
| 1, "this project", "audit", "full", a single path | `workflows/full-audit.md` |
| 2, "quick", "fast", "check", pre-commit context | `workflows/quick-check.md` |
| 3, "portfolio", "sweep", "all projects", multiple paths | `workflows/portfolio-sweep.md` |
| 4, "issues", "file issues", "github", `--issues` | `workflows/file-issues.md` |
| 5, "fix", "apply", `--fix` | `workflows/apply-fixes.md` |
| 6, "harvest", "learnings", "update the skill", `--harvest` | `workflows/harvest-learnings.md` |

Read the workflow, then follow it exactly.

`--issues` or `--fix` passed alongside another intent means: run that workflow first, then chain into the named one. They are alternative sinks for the same findings — issues for durable, reviewable, agent-actionable work; direct edits for mechanical corrections you want now. Both are legitimate; neither runs without approval.
</routing>

<scope>
Documents in scope by default:

- `README.md` and `CLAUDE.md` at every level of the repo
- `docs/**/*.md`
- `CONTRIBUTING.md`, `QUICKSTART.md`, `ARCHITECTURE.md`, `TODO.md`, `PROJECT_INSTRUCTIONS.md`
- Generated documentation, which is audited against **what actually generated it**

Excluded: `node_modules/`, `.venv/`, `vendor/`, `archive/`, `htmlcov/`, `.git/`, and any path in `.gitignore`.

`CHANGELOG.md` is a historical record — old entries are *supposed* to describe old states. Audit only its most recent entry against the current version.
</scope>

<reference_index>
Domain knowledge in `references/`:

- **integrity.md** — anti-fabrication rules, operational limits, privacy, state disclosure
- **drift-catalog.md** — the recurring classes of documentation drift and how to detect each
- **verification.md** — concrete commands for establishing ground truth per ecosystem
- **confidence.md** — HIGH/MEDIUM/LOW rules, the absence protocol, shell traps
- **severity.md** — the P0–P3 ranking rules and what qualifies for each
- **run-log.md** — the per-run dataset (`runs.jsonl`): schema, and the honest limits on what a run can measure about itself
</reference_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| full-audit.md | Complete audit of one project — read everything, verify everything |
| quick-check.md | Fast mechanical pass: dead paths, version mismatches, broken commands |
| portfolio-sweep.md | Parallel audit across many projects, plus cross-project patterns |
| file-issues.md | File findings as agent-executable GitHub issues, with a security gate |
| apply-fixes.md | Apply confirmed findings to the working tree, with an approval gate |
| harvest-learnings.md | Fold past audits' Learnings sections into `references/` — the skill's feedback loop |
</workflows_index>

<output>
Findings are written to `DOC-AUDIT.md` in the audited project's root, using `templates/audit-report.md`.

**The report's Learnings section is mandatory and is filled during the audit, not afterwards.** It captures near-misses, environment traps and rule gaps at the moment they are discovered — a closing step competes with finishing the report and loses. `/audit-docs --harvest` later folds those rows into `references/`. A skill that does not learn from its own runs decays exactly like the documentation it audits.

The file is a snapshot, not an accumulating log — overwrite it on each run and let git hold the history. Add it to `.gitignore` if the user prefers it uncommitted.
</output>

<success_criteria>
An audit is complete when:

- Every in-scope document has been read in full
- Every reported finding names the file, the claim, the verification performed, the ground truth, and a confidence level
- No LOW-confidence finding appears in the findings list
- Every absence claim survived the absence protocol
- Every "clean" result came from a check confirmed to have executed
- No finding rests on one document contradicting another without the code deciding between them
- Findings are ranked by consequence, not by discovery order
- Duplicated-fact findings name the canonical source the user should keep
- The user knows what was checked and found clean, not only what was found broken
</success_criteria>

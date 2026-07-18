# Workflow: Full Audit

<required_reading>
Read these now:
1. `references/drift-catalog.md`
2. `references/verification.md`
3. `references/confidence.md`
4. `references/severity.md`
5. `references/integrity.md` — non-negotiable
</required_reading>

<process>

<step n="1" name="Inventory">
List every in-scope document with its size. Size drives the reading plan — a 40 KB README needs multiple reads.

```bash
find . -maxdepth 4 \( -iname 'README.md' -o -iname 'CLAUDE.md' -o -iname 'TODO.md' \
  -o -iname 'CONTRIBUTING.md' -o -iname 'QUICKSTART.md' -o -iname 'ARCHITECTURE.md' \
  -o -iname 'PROJECT_INSTRUCTIONS.md' \) \
  -not -path '*/node_modules/*' -not -path '*/.venv/*' -not -path '*/archive/*' \
  -not -path '*/.git/*' -exec ls -l {} \; | awk '{print $5, $9}' | sort -k2
find docs -name '*.md' 2>/dev/null | head -50
```

Report the inventory to the user before proceeding. If it exceeds ~15 files or ~150 KB, say so and note that reading will be delegated in parallel.
</step>

<step n="2" name="Establish ground truth first">
Before reading any prose, collect the facts. Reading docs first primes you to believe them.

```bash
git rev-parse --git-dir >/dev/null 2>&1 && git branch --show-current && git remote get-url origin
git log -1 --format='%h %ad %s' --date=short
ls package.json pyproject.toml go.mod Makefile 2>/dev/null
```

Then run the ecosystem-appropriate commands from `references/verification.md` to capture: real version, real script list, real test count, real dependency list, real output sizes.

Record these as the reference set. Every later claim is checked against it.
</step>

<step n="3" name="Read the documents — tiered by scale">
Drift concentrates in the tail: troubleshooting sections, version histories, FAQ entries, the fourth copy of a table. Reading the first 100 lines of a 1000-line README finds nothing.

**Under ~20 files / ~150 KB:** read every file completely. No tiering.

**Above that, tier explicitly and say so.** A rule demanding the impossible gets silently ignored, which is worse than a rule that admits its limits:

- **Tier 1 — always read in full.** Entry points: root `README.md`, `CLAUDE.md`, `CONTEXT.md`, `CONTRIBUTING.md`, plus any doc index. These carry the claims a new reader meets first, so their drift costs the most.
- **Tier 2 — read in full, delegated.** Every doc that Tier 1 links to or that a documented command names. One Explore agent per 2–3 files, or per coherent directory.
- **Tier 3 — indexed, not read.** Everything else. Record the count, verify their *existence* as link targets, and state plainly that their contents were not audited.

Never present a tiered audit as comprehensive. The report's scope section must name the tiers and the Tier 3 count.

When delegating: subagents **report claims** with file and line; the main thread **adjudicates**. Keep the ground-truth set in the main context. A subagent's confidence is not evidence — treat every delegated claim as LOW until verified centrally.

**Delegate extraction to a cheap model; keep adjudication on the strong one.**

```
Agent(subagent_type='Explore', model='haiku', prompt=<extraction prompt>)
```

Extraction is mechanical transcription — `file:line | verbatim quote | claim type` — with judgment explicitly forbidden. That is the cheapest possible workload, and a large doc set is exactly where the token cost concentrates: one real audit spent ~95k subagent tokens on three readers alone.

Adjudication must stay on the session's strong model. It is where the false-positive risk lives: deciding which source wins, running the absence protocol, distinguishing a floor claim (`200+`, still true at 1916) from a false one, and noticing that a naive `rg -c` double-counts. Those are the judgments this skill exists to get right, and they are not cheap work.

The split is safe **because** the reader is forbidden to verify. If a cheap reader miscopies a line number or hallucinates a path, the central verification step catches it — every delegated claim is re-checked against the filesystem before it can reach a finding. Do not give a delegated reader verification authority in order to save a round trip; that removes the check that makes the cheap model safe.

**Require every reader to state what it could not read.** A reader that silently truncates turns a partial audit into an apparently complete one.

While reading, extract every checkable claim into a list: file, line, claim, claim type.
</step>

<step n="3b" name="Defer to the project's own verification agent">
Check for one before adjudicating anything measurement-shaped:

```bash
ls .claude/agents/ 2>/dev/null
```

If the project ships a verification, falsification, or fidelity agent, **route domain claims to it** rather than adjudicating them yourself. It encodes knowledge this skill does not have — which metrics are meaningful, which denominators are legitimate, what counts as reproduction.

This matters most where verifying a claim would require running the product: accuracy percentages, fidelity scores, benchmark figures. Those commands have side effects and long runtimes, so this skill must not run them. Without the project's own agent, such claims belong in **Unverifiable** — not in findings.

Two failure modes to avoid: duplicating an existing agent's work with weaker tools, and contradicting its verdict from a position of less knowledge.
</step>

<step n="4" name="Verify each claim">
Work through the claim list against ground truth. For each, run the specific check from `drift-catalog.md`.

Rules:
- Never resolve a doc-vs-doc conflict without the code casting the deciding vote.
- Never report a claim you did not actually check.
- Run the secrets scan from `drift-catalog.md` regardless of what else you found.
- Check the inverse of every stated gap — features get built and gap-tables get forgotten.
- **Any claim that something is absent goes through the absence protocol** in `references/confidence.md`: three independent patterns, a positive control against a file known to have the feature, loosest-first, and inspection of real matches. A single zero result is LOW confidence and is not a finding.
- **Before recording any check as clean, confirm it executed** — that the extraction produced tokens and the pattern is capable of matching. An empty result from a broken pattern reads identically to a pass.

Mark each claim: CONFIRMED-WRONG, CONFIRMED-OK, or UNVERIFIABLE, and assign HIGH / MEDIUM / LOW confidence with the evidence that earned it.

Escalate every LOW to HIGH with a second method, or move it to Unverifiable. LOW-confidence items must not reach the findings list.
</step>

<step n="5" name="Find the duplication">
With the claim list complete, look across it for facts appearing in multiple places. These are the root causes — the individual mismatches are symptoms.

For each duplicated fact, determine which copy should be canonical (prefer code, then generated output, then the most-read document) and note it in the finding.
</step>

<step n="6" name="Rank and write">
Rank by consequence using `references/severity.md`, applying the escalation rules. Collapse one root cause into one finding.

Write the report to `DOC-AUDIT.md` using `templates/audit-report.md`. Overwrite any existing file — git holds the history.
</step>

<step n="7" name="Summarize in conversation">
Give the user, in chat:
- Count of findings by severity
- The three highest-consequence findings, stated plainly
- Anything structural worth naming (a recurring pattern, a rule the project already has and is violating)
- What was checked and found clean

Do not paste the whole report — it is on disk. If `--fix` was requested, chain to `workflows/apply-fixes.md`.
</step>

</process>

<success_criteria>
- [ ] Every in-scope document read in full, not sampled
- [ ] Ground truth established by execution or source inspection before prose was read
- [ ] Every finding names file, line, claim, verification performed, and actual truth
- [ ] No doc-vs-doc finding left unadjudicated by the code
- [ ] Secrets scan run
- [ ] Inverse of each stated gap checked
- [ ] Duplicated facts reported as root causes with a named canonical source
- [ ] Findings ranked by consequence with escalation rules applied
- [ ] Clean results reported alongside failures
- [ ] `DOC-AUDIT.md` written
</success_criteria>

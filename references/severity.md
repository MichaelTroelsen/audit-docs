# Severity Ranking

Rank by **what a reader would do with the bad information**, not by how wrong it is.

A stale gap-table that causes a rebuild of working code outranks a version badge that is off by three minor releases — even though the badge is "more wrong" in a literal sense.

---

<p0 name="Acts on the reader immediately">
Someone following the documentation right now does damage, loses work, or is blocked at the first step.

- Exposed credential in any tracked or untracked file
- Substantial unique work not under version control
- Quick-start or installation command that fails (dead script, wrong flag, moved file)
- Agent-facing doc asserting a completed feature is missing → causes duplicate reimplementation
- Manifest version disagreeing with code version → ships mislabelled artifacts
- Documented test/CI claim that is false → creates unearned confidence in correctness
- Generated artifact internally inconsistent (index older than the content it indexes)

**Test for P0:** would a competent reader waste an hour, ship something broken, or leak a secret by trusting this?
</p0>

<p1 name="Misleads on substance">
Not immediately destructive, but the reader forms a false model of the project.

- Metric that is structurally meaningless (percentage over an empty set, success count with a misleading denominator)
- Two documents teaching incompatible usage of the same tool
- Case-sensitivity breakage in a published repo (images/links dead for everyone but the author)
- Performance or accuracy claims with no methodology, hardware, or dataset stated
- Chained command documented with fewer steps than it runs
- Machine-specific paths in a project that presents itself as installable by others

**Test for P1:** would the reader be surprised — and have to redo their thinking — on discovering the truth?
</p1>

<p2 name="Wrong but self-correcting">
The reader will notice and route around it without much cost.

- Stale counts, sizes, dates, badges where nothing depends on the exact figure
- Dead links to documents that clearly moved (reader finds them)
- Duplicated tables that have drifted on cosmetic points
- README bloat: duplicated changelogs, feature sections restating dedicated guides
- Missing entries in an options table where the tool's `--help` covers it

**Test for P2:** is this a maintenance burden rather than a trap?
</p2>

<p3 name="Structural and stylistic">
Correct today, but the structure invites future drift.

- A fact stored in 3+ places that currently agrees (report the fragility, not an error)
- TODO files dominated by completed items
- Agent-facing docs carrying large domain data that costs context every session
- Hand-maintained descriptions of generated output that happen to be accurate right now
- Boilerplate stubs that are not yet actively wrong
- Absolute paths in an explicitly personal, unpublished project

**Test for P3:** is this fine now but a predictable P0 later?
</p3>

---

<escalation_rules>
Some findings jump severity based on context:

| Finding | Base | Escalates to | When |
|---|---|---|---|
| Stale "not implemented" note | P2 | **P0** | doc is agent-facing (`CLAUDE.md`, `PROJECT_INSTRUCTIONS.md`) |
| Machine-specific path | P3 | **P1** | repo is published, has a LICENSE, or invites contributors |
| Case-mismatched link | P2 | **P1** | repo is hosted on GitHub or a release site |
| Version mismatch | P2 | **P0** | the disagreeing file is the packaging manifest |
| Missing README | P2 | **P0** | project has no other entry point and multiple components |
| Duplicated fact | P3 | **P1** | the copies have already drifted |
| Unverifiable metric | P2 | **P1** | it appears in a headline claim or badge |
</escalation_rules>

<counting_discipline>
One root cause is one finding, however many files it touches. Eight references to the same deleted script is a single finding listing eight locations — not eight findings. Inflated counts make an audit feel productive while burying what matters.

Conversely, do not merge distinct causes because they share a file. A stale version and a dead link in the same README are two findings.
</counting_discipline>

<framing>
When a project's own documentation already states the rule being violated, quote it. A project that forbids duplicated changelogs in its `CLAUDE.md` while its README carries 250 lines of duplicated changelog does not need to be persuaded of the principle — only shown the gap.

The same applies to post-mortems: if the docs describe a past bug caused by two copies of a mapping drifting apart, and the audit finds that exact pattern elsewhere, name the parallel. It is the most actionable framing available, because the user already believes it.
</framing>

<tone>
Report findings as observations with evidence, not as criticism. The user wrote these docs; drift is the normal entropy of an evolving project, not negligence.

State what is true, what the doc says, and what it would cost. Skip adjectives about the severity of the mistake — the P-level already carries that. Never editorialize about how long something has been wrong.
</tone>

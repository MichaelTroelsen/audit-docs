# Workflow: Harvest Learnings

Folds the **Learnings** sections of past audit reports into `references/`, so the skill improves from
use instead of staying at whatever its author guessed on day one.

Capture happens during an audit (the report template's Learnings section). This workflow is the
second half: reading those rows and turning them into rules.

Run it after a batch of audits, or whenever `references/` feels behind what you actually know.

<required_reading>
Read now:
1. `references/integrity.md` — this workflow edits the skill's own rulebook; the anti-fabrication
   rules apply to that with full force
2. The reference file you are about to edit, in full, before editing it
</required_reading>

<why_this_exists>
The skill's `<self_application>` rule ("fix the reference file in the same session") does not work on
its own, and there is evidence: a session that ran six audits produced eight distinct learnings, and
recorded **zero** of them — including one the assistant explicitly offered to write down and then did
not.

The failure is structural, not a lapse of diligence. Learning happens mid-audit, when context is
long and the incentive is to finish the report. A closing step competes with that and loses.

So: capture goes in the deliverable (the report), and folding-in is a separate, deliberate run.
Do not "improve" this by moving the harvest back into `full-audit.md` as a final step. That is the
design that already failed.
</why_this_exists>

<hard_rules>

**Never invent a learning.** Every rule added here must trace to a row in a real report, from a real
run, against a real repo. A reference file full of plausible-sounding advice nobody actually
learned is exactly the disease this skill treats — and it would be self-inflicted.

**Cite the source.** Every added rule names the project and date it came from. A reader must be able
to ask "where did this come from?" and get an answer. Uncited advice cannot be re-verified when it
later goes stale.

**Prefer measured claims over remembered ones.** "Measured on a real audit, naive single-pattern
searches produced wrong 'missing' verdicts one time in three" earns its place. "Patterns are often
wrong" does not. If a number was counted, keep the number.

**Do not delete existing rules to make room.** If a new learning contradicts an existing rule, that
is itself a finding — say so, and resolve it explicitly rather than silently overwriting.

**Never edit `SKILL.md`'s principles without saying so.** The principles are load-bearing and read
on every invocation. Adding to `references/` is routine; changing a principle is not.

**Cap the diff.** More than ~5 substantive additions in one harvest means the batch is too big to
review properly. Split it.

</hard_rules>

<process>

<step n="1" name="Collect">
Find every audit report and pull its Learnings section.

```bash
find . -name 'DOC-AUDIT.md' -not -path '*/node_modules/*' 2>/dev/null
```

Run from the directory holding the audited projects, not from the skill repo.

For each, extract the section and note the project and audit date. Reports whose Learnings section is
absent predate this mechanism — record how many, so coverage is legible.
</step>

<step n="2" name="Deduplicate against what is already written">
**Read the target reference file in full before deciding anything is new.** A rule that already
exists in different words must not be added again — duplicated truth inside the skill is the exact
failure it exists to catch.

For each candidate row:

- **Already covered** → drop it, and note that it recurred. A learning appearing in three separate
  audits is evidence the existing rule is not landing, which is a *different* fix: make the existing
  rule louder or move it earlier, do not add a second copy.
- **Contradicts an existing rule** → stop and resolve explicitly. Report both to the user.
- **Genuinely new** → carry forward.
</step>

<step n="3" name="Generalise before writing">
A raw row is an anecdote. A rule is what makes the next audit catch it.

Ask of each: **would this have helped on a project I have not seen?**

- `flags["dry-run"]` was missed by a `flags\.\w+` pattern
  → *"A hyphenated flag cannot be accessed with dot notation. When checking for a flag whose name
  contains `-`, bracket access is the only possibility — a dot-notation pattern is structurally
  incapable of matching it, not merely unlucky."*

The first is a story about one file. The second fires on every repo with a hyphenated flag.

If a row does not generalise, it belongs in an environment-notes list, not as a rule. That is a
legitimate outcome — do not inflate it into a principle to make the harvest look productive.
</step>

<step n="4" name="Route to the right file">
| Learning type | File |
|---|---|
| Pattern gave a false zero; absence-protocol gap; shell trap | `references/confidence.md` |
| New drift class, or a new detection command for an existing one | `references/drift-catalog.md` |
| Ground-truth command for an ecosystem; a manifest quirk | `references/verification.md` |
| Severity judgement the rules did not cover | `references/severity.md` |
| Something the skill must not do; an operational limit learned the hard way | `references/integrity.md` |
| Reading strategy, delegation, tiering | `workflows/full-audit.md` |

If a learning fits nowhere, that is a signal the reference set has a gap. Say so rather than forcing
it into the nearest file.
</step>

<step n="5" name="Write, with the evidence attached">
Add the rule with its provenance inline:

```markdown
**Observed** (SIDM2, 2026-07-18): `rg -c '_AVAILABLE\s*='` returned 26 for 13 real flags — each is
assigned twice, once in `try` and once in `except ImportError`. Counting matching *lines* instead of
unique *names* would have reported a 26-vs-12 discrepancy that does not exist.

Count identities, not matches:
`rg -o '(\w+_AVAILABLE)\s*=' -r '$1' FILE | sort -u | wc -l`
```

Keep the counterexample. A rule with its failing case attached survives editing; a bare imperative
gets softened into uselessness over time.
</step>

<step n="6" name="Mark what was consumed">
Do not delete harvested rows from the reports — the reports are audit records of their own runs.
Append a marker to each consumed row's line instead:

```
<!-- harvested: 2026-07-18 -> references/confidence.md -->
```

Then a later harvest can skip them without re-reading the whole history, and a reader can see which
learnings became rules and which are still loose.
</step>

<step n="7" name="Commit and report">
The skill directory is itself a git repo. Commit with the sources named:

```bash
git -C ~/.claude/skills/audit-docs add -A
git -C ~/.claude/skills/audit-docs commit -m "harvest: <n> learnings from <projects>"
```

Report to the user:
- What was added, and to which file
- What was **dropped as already covered** — and if anything recurred, say so plainly, because that
  means an existing rule is not working
- Any contradiction found between a new learning and an existing rule
- Which reports had no Learnings section (pre-dating the mechanism)
</step>

</process>

<success_criteria>
- [ ] Every added rule traces to a real row in a real report, with project and date cited
- [ ] Target reference files read in full before editing
- [ ] Nothing added that the references already covered in other words
- [ ] Recurring learnings reported as a signal about the existing rule, not added twice
- [ ] Each rule generalises beyond the project it came from, or is filed as an environment note
- [ ] Counterexamples and measured numbers preserved, not paraphrased away
- [ ] Consumed rows marked in the source reports
- [ ] Skill repo committed with sources named in the message
</success_criteria>

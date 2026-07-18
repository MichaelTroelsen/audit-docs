# Documentation Audit — {PROJECT_NAME}

**Audited:** {DATE} · **Commit:** {SHORT_SHA} · **Branch:** {BRANCH}
**Documents read:** {N} files, {TOTAL_SIZE} — all read in full
**Findings:** {P0_COUNT} P0 · {P1_COUNT} P1 · {P2_COUNT} P2 · {P3_COUNT} P3
**Confidence:** {HIGH_COUNT} HIGH · {MEDIUM_COUNT} MEDIUM · 0 LOW (LOW is never reported)

---

## Ground truth

Established by execution or source inspection before documentation was read.

| Fact | Actual value | Source | Confidence |
|---|---|---|---|
| Version | {VERSION} | {MANIFEST_FILE} | {LEVEL} |
| Tests | {TEST_COUNT} | {TEST_SOURCE} | {LEVEL} |
| Dependencies | {DEP_COUNT} | {MANIFEST_FILE} | {LEVEL} |
| Branch | {BRANCH} | `git branch --show-current` | {LEVEL} |
| Under version control | {YES_NO} | `git rev-parse` | {LEVEL} |

---

## Findings

<!-- One entry per root cause. Multiple affected locations belong in one entry. -->

### P0-1 · {SHORT_TITLE}

**Locations:** `{FILE}:{LINE}` {+ additional locations}
**Claim:** "{VERBATIM_QUOTE}"
**Verified by:** `{COMMAND_OR_METHOD}`
**Actual:** {GROUND_TRUTH}
**Evidence:** {THE_MATCHED_STRING_OR_VALUE_READ}
**Confidence:** {HIGH|MEDIUM} — {WHAT_EARNED_IT}
**Consequence:** {WHAT_A_READER_WOULD_DO_WRONG}
**Fix:** {SPECIFIC_ACTION}

<!-- Split confidence when fact and impact differ:
     "HIGH on the defect, LOW on it ever firing — current input avoids the bug" -->
<!-- Multi-part findings: add a per-row Confidence column to the evidence table. -->
<!-- Absence claims: state the patterns tried and the positive control that proved
     the pattern works. An unqualified "not present" is not reportable. -->

<!-- Repeat per finding, in severity order. -->

---

## Duplicated facts

Root causes rather than symptoms. Each row is one fact stored in several places.

| Fact | Locations | Currently agree? | Canonical source should be |
|---|---|---|---|
| {FACT} | {FILE_LIST} | {YES_NO} | {RECOMMENDED_SOURCE} |

---

## Verified clean

Checked and correct — recorded so audit coverage is legible, not only its failures.
Each entry must come from a check confirmed to have executed, not merely to have returned nothing.

- {CLAIM} — verified via `{METHOD}` ({N} tokens checked)

---

## Unverifiable

Claims that could not be checked against ground truth. Not findings; listed so they are not silently dropped.

| Claim | Location | Why unverifiable |
|---|---|---|
| {CLAIM} | `{FILE}:{LINE}` | {REASON} |

---

## Learnings — MANDATORY, and not optional when empty

**This section is the skill's feedback loop. Fill it during the audit, not afterwards.**

Every audit teaches something the skill does not yet know: a pattern that returned a false zero, a
shell behaviour that lied, a command unavailable in this environment, a severity judgement the rules
did not cover. Those are discovered mid-run and forgotten by the time the report is written — which
is why this section sits in the deliverable rather than in a closing step.

Record the moment it happens. Cheap to write now, unrecoverable later.

### Near-misses — checks that nearly produced a false finding

The highest-value rows. A pattern that returned zero and was wrong is worth more than a finding.

| Expected | What actually happened | Why the check failed | Belongs in |
|---|---|---|---|
| {WHAT_I_ASSUMED} | {WHAT_WAS_TRUE} | {ROOT_CAUSE_OF_THE_BAD_PATTERN} | `references/{FILE}.md` |

State the **structural** reason, not just the fix. "The flag is hyphenated, so it cannot be accessed
with dot notation — the pattern was incapable of matching, not merely unlucky" generalises. "Used the
wrong pattern" does not.

### Environment notes — things this shell/OS does that broke a check

| Observed | Consequence | Belongs in |
|---|---|---|
| {BEHAVIOUR} | {WHAT_IT_BROKE} | `references/{FILE}.md` |

### Rule gaps — a judgement the references did not cover

| Situation | What the rules say | What was actually right | Belongs in |
|---|---|---|---|
| {CASE} | {EXISTING_RULE_OR_SILENCE} | {DECISION_MADE} | `references/{FILE}.md` |

### Cross-project signal

Anything true of this project that is probably true of others — a practice that measurably reduced
drift, a class of defect appearing in a third repo, a correlation worth testing.

> **If every table above is empty, say so explicitly and give the reason.** An audit that learned
> nothing is possible on a small clean repo, but it is unusual — and silence here is
> indistinguishable from not having looked. "No near-misses: every absence claim matched on the
> first pattern, verified against N positive controls" is a legitimate entry. A blank section is not.

**These rows are harvested, not applied here.** Run `/audit-docs --harvest` to fold them into
`references/`. Leave them in place afterwards; the harvest marks what it has consumed.

---

## Structural observations

Patterns rather than individual errors — recurring drift classes, rules the project states but does not follow, documentation that should be generated rather than maintained.

---

## Recommended order

1. {P0_ACTION}
2. {P1_ACTION}
3. {P2_ACTION}

<!-- Regenerated on each audit. Git holds the history. -->

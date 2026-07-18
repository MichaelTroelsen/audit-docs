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

## Structural observations

Patterns rather than individual errors — recurring drift classes, rules the project states but does not follow, documentation that should be generated rather than maintained.

---

## Recommended order

1. {P0_ACTION}
2. {P1_ACTION}
3. {P2_ACTION}

<!-- Regenerated on each audit. Git holds the history. -->

# Workflow: Apply Fixes

Acts on confirmed findings. Only runs when the user explicitly asks — via `--fix`, "apply the fixes", or selecting it from the intake.

<required_reading>
Read now:
1. `references/confidence.md` — defines the eligibility gate below
2. `references/severity.md`
3. `references/integrity.md` — non-negotiable
</required_reading>

<safety_rules>
These are not negotiable.

**Only HIGH-confidence mechanical findings may be auto-fixed.** MEDIUM is proposed and never applied unattended; LOW should not have reached the report at all. Editing on anything less than HIGH means changing a file on a guess — which is how the wrong claim got into the documentation in the first place.

**Never fix an unverified finding.** If the audit marked something UNVERIFIABLE, it is not eligible. Uncertainty is resolved by asking, never by editing.

**Never fix by making the code match the docs.** The docs describe; the code decides. If a README documents a flag the CLI lacks, the fix is to correct the README — not to add the flag. Adding the flag is a feature request and belongs to the user.

**Never delete content whose purpose is unclear.** A section that looks redundant may exist for a reason not visible in the file. Propose removal; do not perform it unattended.

**Never touch `CHANGELOG.md` history.** Old entries describe old states correctly.

**Never rewrite a fact into a second location.** If the finding is duplicated truth, the fix is to make one copy canonical and have the others reference it — not to update every copy, which recreates the drift.
</safety_rules>

<process>

<step n="1" name="Classify eligibility">
Sort confirmed findings into three tiers.

**Mechanical** — one correct answer, verifiable, no judgment, and confirmed at HIGH confidence:
- version string → the value from the canonical manifest
- dead link → the file's real current path
- stale count → the value produced by the real source
- case-mismatched path → the real casing
- placeholder (`YOUR_USER/YOUR_REPO`) → the real value from git remote
- hardcoded branch name → the actual branch

**Judgment** — an obvious fix exists but the intent is the user's:
- deleting a stale gap-table row (is the feature genuinely done?)
- removing a duplicated changelog block
- deciding which copy of a duplicated fact becomes canonical
- rewriting a section that mixes correct and incorrect claims

**Out of scope** — never auto-applied:
- anything requiring a code change
- `git init`, secret rotation, `.gitignore` changes
- writing a missing README from scratch
- structural reorganization
</step>

<step n="2" name="Present the plan and stop">
Show the user, before editing anything:
- Each mechanical fix as a concrete before → after, with its file and line
- Each judgment fix as a question with a recommendation
- Each out-of-scope item as a manual action to take

**Wait for approval.** Do not proceed on inference. This gate exists because an audit's value depends on the user trusting that nothing changed without their knowledge.
</step>

<step n="3" name="Apply approved mechanical fixes">
Edit precisely — change the incorrect value, nothing adjacent. No reformatting, no rewrapping, no "while I'm here" improvements. A diff full of unrelated whitespace churn is unreviewable, which defeats the point.

Never reproduce a secret value while fixing anything near it.
</step>

<step n="4" name="Handle approved judgment fixes one at a time">
Apply each individually and describe what changed. If a fix turns out larger than presented, stop and return to the user rather than expanding scope silently.
</step>

<step n="5" name="Re-verify">
Re-run the specific checks for every fixed finding. A fix that was not verified is a claim — exactly the thing this skill exists to catch.

Report anything that did not verify clean.
</step>

<step n="6" name="Update the report">
Rewrite `DOC-AUDIT.md`, marking each finding FIXED, SKIPPED, or MANUAL, and leaving the out-of-scope items listed. Do not delete fixed findings — the record of what changed is the useful artifact.

Do not commit. Staging and committing are the user's calls.
</step>

</process>

<success_criteria>
- [ ] Every applied fix came from a CONFIRMED, HIGH-confidence, mechanical finding
- [ ] No MEDIUM-confidence finding was applied without explicit per-item approval
- [ ] Plan presented and explicitly approved before any edit
- [ ] No code changed to accommodate documentation
- [ ] No content deleted without approval
- [ ] Duplicated facts consolidated to one canonical source, not synchronized
- [ ] Each fix re-verified
- [ ] `DOC-AUDIT.md` updated with outcomes
- [ ] Nothing committed
</success_criteria>

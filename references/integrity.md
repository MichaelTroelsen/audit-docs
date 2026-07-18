# Integrity Rules

"Do not hallucinate" is not an instruction an agent can follow — it names a failure without giving a procedure. What works is making fabrication **structurally impossible to hide**: every claim must carry evidence that a reader can independently re-run.

These rules apply to every workflow. They matter most in `file-issues.md`, because a public issue asserting something false is durable, visible, and damages the credibility of every future audit.

---

<cardinal_rule>
**Never report a verification you did not perform.**

Not "probably true". Not "would obviously return X". If the command was not run, its output does not appear in the report.

This is the single most likely failure mode of this skill, because audit reports have a persuasive shape: a table of commands and outputs *looks* rigorous whether or not the commands ran. Fabricated evidence is far worse than a missed finding — it converts the audit from a check into a source of new false claims, which is precisely the disease it exists to treat.
</cardinal_rule>

<observed_failure>
This has already happened once, during the skill's own first test run.

A draft issue contained the verification step `python retro_guide_sync.py --print-urls`. **That flag does not exist.** It was written because it was the shape a verification command ought to take, not because anyone checked. It was caught only because the author flagged it before filing.

Note what makes it dangerous: it was plausible, well-formed, and would have been filed on a public repo as an instruction to a future agent. Anyone following it would hit an error and lose trust in the whole issue.

**Rule derived:** every command that appears in a report or issue must have been executed, in this session, against this repo, with its real output captured. If a command is offered as a *suggestion* rather than a performed check, label it explicitly as untested.
</observed_failure>

---

<rules>

<rule name="Quotes are verbatim">
Text presented as a quotation must be byte-identical to the source. Paraphrase is fine — but then it must not be in quotation marks or a code block.

Copy from the actual file read. Never reconstruct a quote from memory of having read it.
</rule>

<rule name="Line numbers are printed, not estimated">
Before citing `file:line` or a range, print it and read the output:
```bash
sed -n '29,40p' FILE | cat -n     # confirm the range starts where you claim
rg -n 'exact quoted text' FILE    # or let ripgrep supply the number
```

Prefer `rg -n` over `sed` — it derives the number from the content instead of asking you to guess a range and check it afterwards.

Off-by-one citations are not fabrications, but they force the reader to hunt, and they signal that the remaining numbers were not checked either.

**Observed:** an issue cited `PROJECT_INSTRUCTIONS.md:32-40` for a section whose heading is on line 31 and whose items run 33–40. Nobody was misled, but the citation was written from an impression of the file rather than from its output — which is the same mechanism that produces fabricated evidence, operating on something harmless.
</rule>

<rule name="Output is pasted, never reconstructed">
Command output in a report is copied from the terminal. If output was long and is being truncated, say so with an ellipsis and a note — do not silently show a tidied version.
</rule>

<rule name="Never invent an interface">
Do not name a flag, subcommand, script, config key, or file path that has not been observed. If a fix requires a capability that may not exist, check first — `--help`, the source, the manifest — or write the fix without depending on it.

This applies especially to verification commands, which read as authoritative precisely because they look executable.
</rule>

<rule name="Counts come from counting">
Never state a number that was not produced by a command. "8 of 9 gaps are closed" requires nine checks, not an impression formed from three.
</rule>

<rule name="Say what was not checked">
If files were skipped, sampled, truncated, or delegated without adjudication, say so in the report. An audit that silently narrows its scope while reading as comprehensive is a false claim about the audit itself.

Never write "read in full" unless every listed file was read to its end.
</rule>

<rule name="Uncertainty is stated, not smoothed">
"I could not determine whether the CI actually runs these tests" is a legitimate audit output. Choosing a confident phrasing to make the report read better is fabrication with good manners.

Unverifiable claims go in the Unverifiable section with the reason — they are never dropped to keep the report tidy, and never upgraded to findings to make it look productive.
</rule>

<rule name="Do not inflate">
One root cause is one finding. Splitting a finding across files to raise the count, or reporting P3 style issues as P1, misrepresents the state of the project. The count is a measurement, not a score.
</rule>

<rule name="Correct the record when wrong">
If a finding turns out to be mistaken after it was reported or filed, say so plainly and amend it — close the issue with an explanation, or post a correcting comment. Do not quietly drop it from the next run's report.

An audit tool that cannot admit error is worth less than no audit at all, because its mistakes accumulate silently in a public tracker.
</rule>

</rules>

---

<operational_limits>
Actions this skill must never take without being asked in the current session:

**Never write to a repository's history.** No `git commit`, `git push`, `git rm`, branch creation, or history rewriting. The audit produces findings and, with approval, issues — nothing else.

**Never close, edit, or comment on issues it did not create**, except to reopen a regression that carries its own `audit-docs-id` marker.

**Never assign issues, request reviews, or @-mention anyone.** That sends notifications to real people about work they did not ask for. Filing is silent by default.

**Never delete files.** Recommend deletion in a finding; let the user or an assigned agent perform it.

**Cap issue volume.** More than 10 issues in one run means the grouping is wrong — stop, re-group, and confirm with the user before filing. A tracker flooded by an audit is an audit nobody reads.

**Never file into a repository the user does not own** without explicitly confirming. Check `git remote get-url origin` and say which account it belongs to.

**Stay inside the audited project.** Do not read or report on files outside its root, even when a path in the docs points there.
</operational_limits>

<destructive_command_traps>
Learned by doing damage, not by reasoning about it.

**Case-insensitive filesystems make `rm` wider than it looks.** On Windows and macOS,
`Foo_ANNOTATED.html` and `foo_annotated.html` are the same file.

**Observed** (SIDM2, 2026-07-18): a tool was run to verify a restored script; it wrote
`Stinsens_..._ANNOTATED.html`, which silently **overwrote the tracked**
`Stinsens_..._annotated.html`. Cleaning up "my" generated file then deleted a repository file.
Recovered with `git checkout --` and confirmed identical to `HEAD` — but only because the tree was
otherwise clean and the damage was noticed immediately.

Before removing anything a verification step produced:

```bash
git status --porcelain           # is anything tracked now modified or deleted?
git ls-files --error-unmatch PATH 2>/dev/null && echo "TRACKED - do not rm"
```

Prefer letting generated files sit. An untracked artifact left behind is a smaller problem than a
deleted tracked one, and `git status` will show it.

**Running a tool to verify a fix has side effects.** The same run also created a stray root-level
config file. Verification is worth it — but check `git status` afterwards and clean up deliberately,
knowing which files are yours.
</destructive_command_traps>

<privacy>
A public issue is a publication.

**Never introduce personal data into a public issue** that is not already in the repository — absolute paths containing a username, email addresses, internal hostnames, local IPs, directory listings of unrelated projects.

Quoting a line that already exists in the public repo is fine; it is already published. Adding new context about the author's machine is not.

**Never quote a secret value**, even to demonstrate that it is exposed. Name the file and the fact. See the security rule in `file-issues.md`.
</privacy>

<state_disclosure>
Findings are relative to a specific tree state. Record it, and disclose anything that makes the audit non-reproducible:

```bash
git rev-parse --short HEAD
git status --porcelain | wc -l    # uncommitted changes?
```

If the working tree is dirty, say so in the report header — the findings describe the working tree, which may not match what a reader sees at `HEAD`. An issue whose evidence cannot be reproduced from the named commit will be dismissed, correctly.
</state_disclosure>

<self_application>
This skill is subject to its own rules. Its documentation makes claims — measured false-positive rates, shell behaviours, ecosystem commands — and those decay exactly like any other documentation.

When a command in these reference files fails in practice, fix the reference file in the same session. A drift-detection skill whose own instructions have drifted is the sharpest possible demonstration that documentation cannot be trusted without verification, and the least funny place to learn it.
</self_application>

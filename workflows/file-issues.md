# Workflow: File Issues

Turns confirmed findings into GitHub issues instead of direct edits. Detection and correction are separated: the audit records what is wrong, the repo carries the record, and the fix happens later — by you, by an assigned agent, or not at all.

Each issue is written as an **executable specification** so an agent picking it up can act without re-deriving the finding.

<required_reading>
Read now:
1. `references/confidence.md`
2. `references/severity.md`
3. `references/integrity.md` — non-negotiable
</required_reading>

<hard_rules>

**Never file a security finding as an issue on a public repository.** An exposed credential, a secret in a tracked file, an unignored config — filing these publicly broadcasts the vulnerability to everyone watching the repo, and the issue persists in the API and in forks even after deletion. Report security findings **in conversation only**, and say explicitly that they were withheld from issue-filing and why.

This holds regardless of how the finding was phrased. "Rotate the token in `.mcp.json`" tells a reader exactly which file to look in.

**Never file issues without explicit approval.** Creating issues on a public repo is outward-facing and effectively irreversible — deleted issues remain in the API, in notifications already sent, and in anyone's inbox. Always preview first (see step 4). Approval for one repo is not approval for the next.

**Never file LOW or MEDIUM confidence findings.** A public issue asserting something false about a repo is worse than a silent miss. HIGH only.

**Never file into a repo you do not own** without saying so. Check `git remote get-url origin` and name the account — a user often has repos under several accounts or organisations, and filing into the wrong one is not silently undoable.

**Never file more than 10 issues in one run.** Exceeding that means the grouping is wrong. Stop, re-group, confirm. A tracker flooded by an audit is an audit nobody reads.

**Never assign, @-mention, or request review.** That notifies real people about work they did not ask for. Filing is silent by default.

**Never close, edit, or comment on issues this skill did not create** — the sole exception is reopening a regression carrying its own `audit-docs-id` marker.

**Never introduce personal data into a public issue** that is not already in the repository: absolute paths containing a username, email addresses, internal hostnames, local IPs. Quoting a line that already exists publicly is fine — it is already published. Adding new context about the author's machine is not.

**Never commit, push, or delete files.** This workflow creates issues. Nothing else touches the repository.

</hard_rules>

<process>

<step n="1" name="Preconditions">
```bash
gh auth status                                   # authenticated, with `repo` scope
git remote get-url origin                        # exists at all?
gh repo view --json visibility,hasIssuesEnabled --jq '"\(.visibility) issues:\(.hasIssuesEnabled)"'
```

Stop and report if: no remote (nothing to file into), issues disabled, or `gh` unauthenticated.

**A project with no remote is not a failure of this workflow** — it is a finding in its own right. Report the audit results in conversation and recommend `git init` + a remote if the work warrants it.

Record visibility. It gates the security rule above.
</step>

<step n="2" name="Check for existing issues — idempotency">
Re-running an audit must not duplicate issues. Every issue this workflow creates carries a stable marker in its body:

```
<!-- audit-docs-id: {stable-slug} -->
```

The slug is derived from the finding's identity, not its wording — `{class}-{primary-file}-{key}`, e.g. `stale-gap-table-CLAUDE.md`, `version-mismatch-pyproject.toml`. It must stay identical across runs.

```bash
gh issue list --repo "$REPO" --state all --search "audit-docs-id" --limit 100 \
  --json number,title,state,body \
  --jq '.[] | select(.body | test("audit-docs-id: ")) | "\(.number)\t\(.state)\t\(.title)"'
```

For each finding, classify:
- **New** → create
- **Open duplicate** → do not create; add a comment only if the evidence materially changed
- **Closed duplicate** → the finding regressed. Reopen with a comment showing the new evidence rather than filing a fresh issue — the history is the point.
</step>

<step n="3" name="Group findings">
One issue per **root cause**, never one per affected line. Eight references to the same deleted script is one issue listing eight locations.

| Severity | Treatment |
|---|---|
| P0 | Own issue, always |
| P1 | Own issue |
| P2 | Own issue only if independently actionable; otherwise batch |
| P3 | Single umbrella issue with a checklist |

A 31-finding audit should produce roughly 6–10 issues, not 31. Volume makes the tracker unusable and buries the P0s — the same failure mode as an audit report that inflates its count.

If a fix resolves several findings at once, say so in the issue body and cross-reference. Do not file the dependents separately.
</step>

<step n="4" name="Preview and get approval">
Show the user, before creating anything:
- Target repo and its visibility
- Each issue: title, severity label, and a two-line summary
- Which findings are being **withheld** (security, LOW/MEDIUM confidence) and why
- Which are duplicates of existing issues, and their numbers

Then stop and ask. Do not proceed on inferred consent.
</step>

<step n="5" name="Ensure labels exist">
```bash
gh label create "docs-audit" --repo "$REPO" --color "0E8A16" \
  --description "Found by audit-docs" 2>/dev/null || true
gh label create "P0" --repo "$REPO" --color "B60205" 2>/dev/null || true
gh label create "P1" --repo "$REPO" --color "D93F0B" 2>/dev/null || true
gh label create "P2" --repo "$REPO" --color "FBCA04" 2>/dev/null || true
gh label create "P3" --repo "$REPO" --color "C2E0C6" 2>/dev/null || true
```
`|| true` because a pre-existing label is success, not failure.
</step>

<step n="6" name="Create issues">
Use `templates/issue-body.md`. Write the body to a temp file and pass `--body-file` — inline `--body` mangles multi-line content and backticks across shells.

```bash
gh issue create --repo "$REPO" \
  --title "docs: {concise statement of what is wrong}" \
  --label "docs-audit,P1" \
  --body-file "$TMP/issue-N.md"
```

Title rules: state the defect, not the fix. `docs: gap table lists features that already exist` — not `docs: update gap table`. A reader scanning the tracker should learn something from the title alone.
</step>

<step n="7" name="Report">
Give the user the created issue URLs, the withheld findings, and any duplicates skipped.

Update `DOC-AUDIT.md` so each finding records its issue number. The local report stays the complete picture; the issues are the actionable subset.
</step>

</process>

<self_healing>
The point of filing rather than fixing: an issue written as an executable specification can be handed to an agent later — assigned on GitHub, picked up by `gh issue view` in a fresh Claude Code session, or batched. The repo accumulates its own repair instructions.

For that to work, every issue body must be **sufficient on its own**. An agent reading it months later has no audit context and cannot see this conversation. It must contain:

1. The exact location — `file:line`
2. The current text, quoted verbatim
3. The required text, or the rule that determines it
4. **The command that proves the fix worked** — this is what makes the issue executable rather than merely descriptive
5. What must not change (guard rails — e.g. "do not change the code to match the docs")

An issue an agent cannot verify is an issue an agent should not action. If a finding cannot be given a verification command, file it as a discussion item and say so — do not dress it up as actionable.
</self_healing>

<success_criteria>
- [ ] Security findings withheld from public repos, and the withholding reported
- [ ] Only HIGH-confidence findings filed
- [ ] Existing issues searched by marker; no duplicates created
- [ ] Regressions reopened rather than re-filed
- [ ] Findings grouped by root cause, not one issue per line
- [ ] Preview shown and approval obtained before any issue was created
- [ ] Every issue body contains location, current text, required text, and a verification command
- [ ] Issue URLs reported and recorded in `DOC-AUDIT.md`
- [ ] Repos without remotes handled by reporting, not by failing
</success_criteria>

# audit-docs

A [Claude Code](https://claude.com/claude-code) skill that audits documentation against the actual codebase.

Not a linter and not a style checker. It answers one question: **are the claims in your README, CLAUDE.md and docs still true?**

It finds dead file references, stale version numbers, wrong test and endpoint counts, broken commands, resolved TODOs still listed as open, and — most usefully — facts duplicated across several files that have quietly drifted apart.

## Why

Documentation rots in a specific way. A script gets archived and eight references keep pointing at it. A gap table lists features that shipped months ago. A count is written once and copied forward forever. None of this is caught by tests, CI, or code review, because none of those read prose.

Agent-facing documentation makes it worse. A stale "feature X is missing" note in a `CLAUDE.md` doesn't just misinform — it can prompt an agent to reimplement something that already works.

In its first real runs this skill found a CI pipeline that had been failing for six months behind a README claiming three-platform test coverage, a package shipping version `1.0.0` while its code reported `2.24.0`, and a gap table where 8 of 9 listed gaps were already closed.

## Install

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/MichaelTroelsen/audit-docs.git ~/.claude/skills/audit-docs
```

Optionally add the slash command:

```bash
cp ~/.claude/skills/audit-docs/command/audit-docs.md ~/.claude/commands/
```

Restart Claude Code — skills are discovered at session start.

## Use

```
/audit-docs                    full audit of the current project
/audit-docs --quick            fast pass: dead refs, versions, secrets
/audit-docs --sweep <dir>      audit every project under a directory
/audit-docs --issues           file findings as GitHub issues
/audit-docs --fix              apply findings to the working tree
```

Findings are written to `DOC-AUDIT.md` in the audited project. `--issues` and `--fix` both preview and wait for approval before doing anything.

## Design

Three ideas do most of the work.

**Verify, never trust.** A documentation claim is not evidence of itself. Every finding is confirmed against the filesystem, the code, or a command's real output. A conflict between two documents is only half a finding until the code casts the deciding vote.

**Proving absence is hard.** A pattern that matches proves something exists; a pattern that matches nothing proves only that your pattern found nothing. Measured on a real repo, naive single-pattern searches produced wrong "missing" verdicts **one time in three** — the id was `cpcSearch` not `search*`, copy buttons were `CopyButtons()` not `copy-btn`, modals used `chip-card` not `chip-modal`. Every absence claim must survive a protocol: multiple independent patterns, a positive control against a file known to have the feature, and inspection of real matches.

**Never report a verification you did not perform.** Audit reports look rigorous whether or not the commands behind them ran, which makes fabricated evidence the most damaging possible failure — it turns the audit into a new source of false claims. So: commands must be executed in-session, quotes byte-identical, line numbers printed before being cited, counts produced by counting. Findings carry HIGH/MEDIUM/LOW confidence, and only HIGH is eligible for auto-fix.

A corollary that catches real bugs: **a check that did not run is not a clean result.** `rg -c` prints nothing rather than `0` when there are no matches, which silently turns a broken check into an apparently passing one.

## Structure

```
SKILL.md                  router + principles that cannot be skipped
workflows/
  full-audit.md           read everything, verify everything
  quick-check.md          fast mechanical pass
  portfolio-sweep.md      many projects + cross-project patterns
  file-issues.md          agent-executable GitHub issues
  apply-fixes.md          direct edits, approval-gated
references/
  integrity.md            anti-fabrication, operational limits, privacy
  drift-catalog.md        12 classes of drift and how to detect each
  verification.md         ground truth per ecosystem
  confidence.md           HIGH/MEDIUM/LOW, absence protocol, shell traps
  severity.md             P0–P3 ranking by consequence
templates/                audit report, issue body
command/                  optional slash command
```

## Issues as executable specifications

`--issues` exists because detection and correction are usefully separate — and because a finding filed as an issue outlives the session that found it.

Each issue contains the exact location, the current text verbatim, the required text, **a command that proves the fix worked**, and guard rails. That last part matters more than it sounds: an issue reporting a stale gap table carries "do not implement the features listed as missing — they exist; only the table is wrong." Without it, a well-meaning agent builds four search bars that are already there.

Security findings are **never** filed on public repositories. An issue saying "rotate the token in `.mcp.json`" tells everyone watching exactly where to look. Those are reported to the user directly.

## Limits

- Won't catch what it can't verify. Design rationale, judgment calls and unfalsifiable claims are listed as *Unverifiable* rather than dropped or guessed at.
- Large projects get sampled unless you say otherwise. The report always states what was and wasn't read.
- Written for git repositories. Projects without a remote can still be audited; they just can't have issues filed.
- Assumes [ripgrep](https://github.com/BurntSushi/ripgrep) and a POSIX shell. `gh` is needed only for `--issues`.

## License

MIT

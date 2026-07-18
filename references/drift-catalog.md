# Drift Catalog

The recurring ways documentation stops being true. Each entry: what it looks like, how to detect it, why it matters.

This catalog is derived from real audits. The detection commands assume ripgrep (`rg`) and a POSIX shell.

---

<class id="1" name="Dead file reference">
**Shape:** Docs reference a script, doc, or asset that has been renamed, moved, archived, or deleted.

**Detection:** A bare filename in a doc is often a *relative* reference from a table or an index — resolving it only against the repo root produces mostly false positives. Always fall back to a repo-wide `find` before declaring anything missing.

```bash
rg -o '`[a-zA-Z0-9_./-]+\.(py|js|mjs|ts|go|sh|bat|md|json|asm|jar|exe)`' --no-filename \
    README.md CLAUDE.md docs/*.md 2>/dev/null \
  | tr -d '`' | sort -u | while read -r p; do
      [ -e "$p" ] && continue
      # fall back: does the basename exist anywhere outside archive/vendor dirs?
      hit=$(find . -name "$(basename "$p")" -not -path '*/archive/*' -not -path '*/.venv/*' \
              -not -path '*/node_modules/*' 2>/dev/null | head -1)
      if [ -n "$hit" ]; then echo "RELATIVE (ok, resolves to): $p -> $hit"
      else echo "MISSING: $p"; fi
    done

# Markdown links to local files
rg -o '\]\(([^)#h][^)]*)\)' -r '$1' README.md CLAUDE.md 2>/dev/null | sort -u
```

**Measured false-positive rate without the fallback: ~80%.** On a real repo, 4 of 5 reported-missing files existed under `docs/`. Report only the `MISSING:` lines.

A `RELATIVE` hit is still worth a glance: if the doc renders on GitHub, an unqualified filename is a broken link even though the file exists. That is class 10, not class 1 — lower severity.

**Why it matters:** Highest-consequence variant is a **quick-start command that references a deleted script** — the first thing a new reader runs, and it fails immediately.

**Common cause:** A file gets archived (`archive/`, `docs/`) and the mv is not followed by a grep for inbound references.

**Report as P0 if** the dead reference is in installation, quick start, or a documented primary command.
</class>

<class id="2" name="Version divergence">
**Shape:** A version number appears in several places and they disagree. The manifest that actually ships is often *not* the one that was updated.

**Detection:** Collect every version string, then identify which one the build uses.
```bash
rg -n '(version|VERSION|__version__)\s*[=:]\s*["\x27]?[0-9]+\.[0-9]+' \
  pyproject.toml setup.py package.json go.mod */version.py */__init__.py 2>/dev/null
rg -n 'version-[0-9]+\.[0-9]+|v[0-9]+\.[0-9]+\.[0-9]+' README.md CLAUDE.md docs/README.md
```

**Ground truth ranking:** the file the package manager reads wins — `pyproject.toml` / `package.json` / `go.mod` — *even when it is the stale one*, because that is what users install. If the manifest disagrees with the code's own `__version__`, that is itself the finding, and it is P0: it means published artifacts carry the wrong version.

**Report as P0** when a manifest version disagrees with the code version. P2 when only badges and prose disagree.
</class>

<class id="3" name="Metric drift">
**Shape:** Counts stated in prose — test counts, tool counts, document counts, accuracy percentages, file sizes — that were true once.

**Detection:** Find the number's real source, never another document.
```bash
rg -n '\b[0-9]{2,}\+?\s*(unit )?tests?\b|\b[0-9]+\+?\s*tools?\b|\b[0-9]{2,}\s*(docs|documents|cards|files)\b' README.md CLAUDE.md
# then locate the producer:
rg -n 'EXPECTED_PASS|pass_count|test_count' scripts/ Makefile 2>/dev/null
find . -name 'test_*.py' -not -path '*/archive/*' -not -path '*/.venv/*' | wc -l
ls -l output/*.html dist/* 2>/dev/null   # for size claims
```

**Critical rule:** When the same metric appears with different values in the same file, that is a finding on its own — it proves nobody is maintaining the number.

**Watch for:** a metric that is *structurally* meaningless — "0 failures" where the denominator is "files that emitted output" rather than "files that were correct", or a percentage computed from an empty set (`len(0) == len(0)` scoring 100%). These are worse than stale numbers because they will never become wrong on their own. Report as P1.
</class>

<class id="4" name="Stale gap-table / resolved TODO">
**Shape:** A "known limitations", "not yet implemented", "current gaps" or FAQ entry describing work that has since been completed.

**Detection:** For each claimed gap, grep for the feature.
```bash
# e.g. doc claims "no search, no copy buttons, no modals"
rg -c 'search-input|searchBox' *.html
rg -c 'copy-btn|copyCode|clipboard' *.html
rg -c 'chip-modal|openModal' *.html
```

**Why it matters most:** This class is **actively harmful**, not merely wrong. An agent reading "feature X is missing" may reimplement a feature that already exists, producing duplicate code. A human may avoid using something that works.

**Always report as P0** when the doc is agent-facing (`CLAUDE.md`, `PROJECT_INSTRUCTIONS.md`).

**Corollary:** also check the inverse — a doc claiming a feature works when the code says otherwise. A troubleshooting entry saying "X is not implemented" sitting in a file that elsewhere claims "X: 100%" means one of them is dead text.
</class>

<class id="5" name="Command that no longer runs">
**Shape:** Documented commands whose flags, script names, subcommands, or paths have changed.

**Detection:** Compare documented commands against their definitions.
```bash
rg -n '^\s*(npm run|yarn|pnpm) ([a-z:_-]+)' -r '$2' README.md CLAUDE.md | sort -u  # vs package.json scripts
rg -n '"scripts"' -A 30 package.json
rg -n 'pytest\s+\S+\.py' README.md CLAUDE.md   # then verify those test files exist where stated
rg -n 'Use:\s+"' cmd/*.go                      # Go/cobra CLI signature vs documented usage
```

**Frequent specifics:**
- A chained script (`npm run all`) documented with fewer steps than it now has.
- Two documents teaching different call conventions for the same CLI (`-u <url>` vs positional argument) because a flag was made optional and only one doc was updated.
- Test commands pointing at a suite that has been moved to `archive/`, while the same README claims CI runs it. Both cannot be true — resolve which, and report the false one.
- A flag that exists in code but appears in no options table.
</class>

<class id="6" name="Duplicated truth">
**Shape:** The same fact — a table, a standard, a URL, a count — maintained in 3+ places. The root cause of most other classes.

**Detection:** Look for structural repetition across files.
```bash
# same table appearing in multiple docs
rg -l 'Series standards|Known limitations|Available guides' *.md
# a fact also encoded in code (the real source of truth)
rg -n 'TRACKED_FILES|GUIDE_NAMES|EXPECTED_.*FIELDS' *.py *.js
```

**Report the duplication, not just the mismatch.** Name which copy should become canonical — prefer, in order: (1) the code that consumes the fact, (2) a generated file, (3) the single most-read document. Everything else links to it.

**Signal that this class is the project's core problem:** the project's own docs already contain a rule against duplication, or a post-mortem describing a bug caused by two copies drifting. Quote it back — it is far more persuasive than an external recommendation.
</class>

<class id="7" name="Generated artifact documented by hand">
**Shape:** A hand-written README describing generated output — an export, a static site, a build directory — that no longer matches what the generator produces.

**Detection:** Compare the description against a scan of the actual output.
```bash
ls wiki/*.html | wc -l        # vs "4 pages" claimed
du -sh wiki/assets/data/      # vs "~137 MB" claimed
ls -l --time-style=+%Y-%m-%d wiki/assets/data/*.json   # timestamps should cluster
```

**Bonus finding:** mismatched timestamps *within* a generated artifact indicate a real bug, not a doc bug — e.g. a search index generated days before the content it indexes means the search is stale or the file is dead weight. Escalate to P0 and report as a code issue.

**Recommendation:** the generator should emit this README from real stats. Hand-maintaining a description of generated output guarantees drift.
</class>

<class id="8" name="Aspirational or boilerplate placeholder">
**Shape:** Template text that was never filled in — `YOUR_USER/YOUR_REPO`, "This is a new repository", "TODO: add build commands", `<description here>`.

**Detection:**
```bash
rg -n 'YOUR_[A-Z_]+|<[a-z-]+ here>|TODO: (add|fill|describe)|This is a new (repo|project)|Lorem ipsum' --glob '*.md'
```

**Why it matters:** A boilerplate `CLAUDE.md` is worse than none — it actively tells an agent there is nothing to know, suppressing exploration it would otherwise do. Recommend deleting or filling; never leave.
</class>

<class id="9" name="Machine-specific and non-portable">
**Shape:** Absolute paths, hardcoded IPs, and personal directory names in docs or build files, making the project unbuildable by anyone else.

**Detection:**
```bash
rg -n '[A-Z]:[\\/]Users[\\/][^ \x27"`]+|/home/[a-z]+/|192\.168\.[0-9]+\.[0-9]+|localhost:[0-9]+' \
  --glob '*.md' --glob 'Makefile' --glob '*.bat' --glob '*.py' --glob '*.json'
```

**Judgment:** For a private personal project this may be entirely acceptable — report it as P3 with that caveat. Escalate to P1 only if the project is published, has a LICENSE, invites contributors, or the docs claim it is installable by others.
</class>

<class id="10" name="Case-sensitivity and path-portability breakage">
**Shape:** Links that work on Windows/macOS but break on Linux and GitHub.

**Detection:**
```bash
# compare referenced case against real case
rg -o '\]\(([A-Za-z0-9_/-]+/[^)]+)\)' -r '$1' *.md | sort -u | while read -r p; do
  [ -e "$p" ] || echo "CASE/MISSING: $p"
done
ls -d */ | head -50   # eyeball actual directory casing, e.g. Pictures/ vs pictures/
```

**Report as P1** for any repo published on GitHub or a release site — images silently fail to render for everyone but the author.
</class>

<class id="11" name="Structural bloat">
**Shape:** Not false, but unusable. A README duplicating the entire changelog; feature sections that restate dedicated guides; a TODO file that is 90% completed items; an agent-facing doc carrying large domain data that costs context on every session.

**Detection:**
```bash
wc -l README.md CLAUDE.md docs/*.md | sort -rn | head
rg -c '^#{1,3} v?[0-9]+\.[0-9]+\.[0-9]+' README.md   # version-history entries inside a README
rg -c '^\s*- \[x\]' TODO.md; rg -c '^\s*- \[ \]' TODO.md
```

**Report when:** a README exceeds ~400 lines with a duplicated changelog; a TODO's open items are outnumbered 5:1 by closed ones; an agent-facing file spends more than ~30% of its length on data rather than instruction.

**This is P2 or P3** — real, but never at the expense of correctness findings.
</class>

<class id="12" name="Missing document">
**Shape:** Absence rather than error. A substantial project with no README; a promised LICENSE that does not exist; docs assuming version control that was never initialized.

**Detection:**
```bash
ls README.md LICENSE 2>/dev/null || echo "absent"
git rev-parse --git-dir >/dev/null 2>&1 || echo "NOT A GIT REPO"
rg -n '\[.*LICENSE.*\]\(LICENSE\)|MIT License' README.md   # promised but present?
rg -n 'never commit|do not commit|gitignore' *.md            # implies VCS that may not exist
```

**Escalate to P0** when a project with meaningful unique work is not under version control, or when a secret-handling instruction assumes a `.gitignore` that does not exist.
</class>

---

<secrets_check>
Always run, regardless of workflow. Documentation audits routinely surface credentials, because config examples get filled in with real values.

```bash
rg -n '(token|api[_-]?key|secret|password|bearer)\s*[=:]\s*["\x27][A-Za-z0-9_\-]{16,}' \
  --glob '!node_modules' --glob '!.venv' --glob '!*.lock' .
```

A live credential in a tracked or untracked config file is **always P0**, regardless of what else the audit found. Report the file and the fact of exposure — never reproduce the secret value in the report.
</secrets_check>

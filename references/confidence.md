# Confidence Levels

Every finding carries a confidence level. It is not decoration — it determines whether the finding may be reported at all, and whether it is eligible for auto-fix.

Confidence describes **the strength of the verification method**, not how strongly the finding feels true.

---

<levels>

<level name="HIGH">
Ground truth was obtained directly and admits no alternative reading.

- The filesystem answered: `ls`, `find`, `[ -e ]`
- A command was executed and its real output observed
- The authoritative value was read from the file that produces or ships it
- A pattern matched, **and the matched content was inspected** and confirms the claim
- Git answered a git question

**Eligible for reporting. Eligible for auto-fix (mechanical findings only).**
</level>

<level name="MEDIUM">
Strong evidence, one inferential step remaining.

- A pattern matched and the match is unambiguous by its nature, but the surrounding code was not inspected
- Two independent methods agree
- The claim is about intent or consequence rather than a fact (e.g. "an agent would reimplement this")
- The defect is certain but its impact depends on conditions not checked

**Eligible for reporting, labelled MEDIUM. Not eligible for auto-fix.**
</level>

<level name="LOW">
One weak signal. A single pattern search, especially one returning zero.

**Never reported as a finding.** Either escalate to HIGH with a second method, or record it under Unverifiable with the reason. A LOW-confidence finding in a report is a false positive waiting to waste the reader's time.
</level>

</levels>

---

<asymmetry>
**Proving presence is easy. Proving absence is hard.**

A pattern that matches proves the thing exists. A pattern that matches nothing proves only that *your pattern* found nothing — the feature may exist under a different name, and this is the single largest source of false findings in a documentation audit.

Measured on a real 5-file audit: naive single-pattern searches produced **three wrong "missing" verdicts out of nine checks (33%)**. Each would have wrongly confirmed a stale gap-table as accurate — the exact failure the audit exists to prevent, inverted.

Real examples from that run:

| Pattern used | Result | Truth | Why it failed |
|---|---|---|---|
| `id="search` | 0 | search exists | the id was `cpcSearch` — different prefix |
| `class="copy-b`, `<button.*copy` | 0 | copy exists | implemented as `CopyButtons()` + `navigator.clipboard` |
| `chip-modal` | 0 | modals exist | markup used `chip-card` + generic `modal-overlay` |

**Rule: a zero result is LOW confidence until it survives the absence protocol below.**
</asymmetry>

<absence_protocol>
Before reporting that something does not exist:

**1. Use at least three independent patterns.** Vary the vocabulary — the feature name, the likely CSS class, the underlying browser/library API, the handler name.
```bash
rg -ci 'copy' "$f"                       # loosest — is the word present at all?
rg -c 'navigator.clipboard|execCommand'  # the API any implementation must use
rg -ci 'copybutton|copy-btn|copyCode'    # naming variants
```

**2. Run a positive control.** Apply the same pattern to a file known to *have* the feature. If the pattern finds nothing there either, the pattern is broken — not the feature missing.
```bash
rg -c 'navigator.clipboard' known-good.html   # must be > 0, or the pattern is wrong
```

**3. Go loosest-first.** If the bare word is absent from the entire file, absence is credible. If the word appears but your specific pattern does not match, the feature almost certainly exists in a form you have not guessed — inspect the matches.

**4. Inspect before concluding.** Print the actual matched lines. `rg -o` on a loose pattern reveals the real naming in seconds and converts LOW to HIGH.

Only after all four does an absence claim reach HIGH.
</absence_protocol>

<counting_trap>
**Count identities, not matching lines.** A name assigned in both a `try` and an `except ImportError`
branch — the standard optional-import idiom — appears twice, so a line count doubles it.

**Observed** (SIDM2, 2026-07-18): `rg -c '_AVAILABLE\s*='` returned **26** against **13** real flags.
Reported against a doc claiming 12, that becomes a dramatic "26 vs 12" finding that does not exist.
The true finding was off-by-one: the list omitted one flag.

```bash
rg -o '(\w+_AVAILABLE)\s*=' -r '$1' FILE | sort -u | wc -l    # identities
rg -c '_AVAILABLE\s*=' FILE                                   # lines - NOT the same question
```

Applies to any repeated assignment: platform branches, feature detection, re-exports. Before
reporting a count mismatch, ask what the doc is counting — things, or mentions of things.
</counting_trap>

<pattern_capability>
**Before trusting a zero, ask whether the pattern could ever have matched.** Some patterns are
structurally incapable, not merely unlucky — and those produce the most convincing false findings,
because the search looks correct.

**Observed** (discogs, 2026-07-18), two in a single audit:

| Pattern | Returned | Truth | Why it could never match |
|---|---|---|---|
| `flags\.[a-zA-Z]+` for `--dry-run` | 0 | exists | A hyphenated key **cannot** be reached with dot notation. `flags["dry-run"]` is the only legal form, so the pattern was incapable by construction. |
| `//\s*[A-Z][A-Z ]{2,}` for a `// DATA` marker | 0 | exists | The real marker is `// -- DATA ---`. Box-drawing characters sit between the slashes and the word, so `\s*` never reaches the capital. |

Both would have accused correct documentation of describing something that does not exist — the
inverse of this skill's purpose.

**Rule:** when a zero involves a hyphenated, dotted or namespaced identifier, or a decorated comment
marker, re-run without the syntax assumption before concluding anything:

```bash
rg -c 'dry.run' FILE        # tolerate any separator
rg -c 'DATA' FILE           # drop the decoration entirely
```
</pattern_capability>

<blind_spots>
Text search cannot see inside every file. A clean scan over a directory containing these proves nothing about their contents:

| Format | Why grep misses it |
|---|---|
| `.zip`, `.rar`, `.tar.gz` | compressed — contents are not text on disk |
| `.pdf`, `.docx`, `.xlsx` | binary containers |
| `.db`, `.sqlite` | binary |
| minified/bundled JS, source maps | text, but single-line and effectively unsearchable |
| git history | `rg` reads the working tree, not past commits |

**Observed:** a secrets scan over a project reported clean while `files.zip` in the same directory contained a `.mcp.json`. It turned out to hold a placeholder, but a real credential would have been missed identically.

Before reporting a repository-wide scan clean, enumerate what could not be searched:

```bash
find . -type f \( -name '*.zip' -o -name '*.rar' -o -name '*.pdf' -o -name '*.docx' \
  -o -name '*.xlsx' -o -name '*.db' -o -name '*.sqlite*' \) \
  -not -path '*/node_modules/*' -not -path '*/.venv/*' 2>/dev/null

# open the archives that could plausibly carry config
python -c "
import zipfile,sys
z=zipfile.ZipFile(sys.argv[1])
for n in z.namelist(): print(n)
" ARCHIVE.zip
```

For secrets specifically, also scan history, not just the tree:
```bash
git log --all -p -- .env .mcp.json config.json 2>/dev/null | rg -c 'sk-ant|ghp_|AKIA' || echo 0
```

State the blind spots in the report. "No secrets found in tracked text files; 3 archives and 2 PDFs were not searched" is honest. "No secrets found" is not.
</blind_spots>

<shell_caveat>
`rg -c` prints **nothing** and exits non-zero when there are no matches — it does not print `0`. In a loop or table this silently produces blank cells that read as "not checked" or misalign the output entirely.

Always force a value:
```bash
count=$(rg -c 'pattern' "$f" 2>/dev/null || echo 0)
```

Related trap: a check that returns empty because the extraction step matched nothing is indistinguishable from a check that returns empty because everything is clean. **Always confirm the extraction produced tokens before reporting "clean".** This is the empty-versus-empty problem — a metric computed over an empty set that reports success. Guard against it explicitly:
```bash
tokens=$(rg -o 'pattern' "$f" | wc -l)
[ "$tokens" -eq 0 ] && echo "CHECK DID NOT RUN — pattern matched nothing" && exit 1
```
</shell_caveat>

---

<reporting>
State confidence on every finding, and on every row of an evidence table.

Include the evidence itself, not just the verdict — the actual matched string, the command run, the value read. A reader must be able to disagree with a finding without re-doing the work.

**Split confidence when the fact and its impact differ.** A defect can be certain while its consequences are speculative:

> **Confidence:** HIGH on the defect, LOW on it ever firing — the current input happens to avoid the bug.

That is more useful than averaging the two into MEDIUM.

**Report the confidence distribution** in the audit summary — "6 findings, all HIGH" tells the reader something that a bare count does not. If most findings are MEDIUM, say so plainly; it means the audit was shallower than it looks.
</reporting>

<fix_gate>
| Confidence | Report | Auto-fix with `--fix` |
|---|---|---|
| HIGH | yes | yes, if mechanical |
| MEDIUM | yes, labelled | no — propose only |
| LOW | no — move to Unverifiable | never |

An auto-fix applied on anything less than HIGH mechanical confidence edits a file on a guess. The whole premise of this skill is that guesses are what put the wrong claim in the documentation in the first place.
</fix_gate>

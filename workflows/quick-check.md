# Workflow: Quick Check

Fast mechanical pass. Catches the classes that are cheap to detect and expensive to leave: dead references, version mismatches, exposed secrets, broken documented commands.

Designed to run in well under a minute — suitable before a commit, a release, or a doc update. It does **not** replace a full audit; it will not catch stale gap-tables, meaningless metrics, or duplicated truth.

<required_reading>
Read now:
1. `references/verification.md`
2. `references/integrity.md` — non-negotiable
</required_reading>

<process>

<step n="1" name="Dead references">
Bare filenames are usually relative references from a table. Without a `find` fallback this check is ~80% false positives — verified on a real repo.

```bash
rg -o '`[a-zA-Z0-9_./-]+\.(py|js|mjs|ts|go|sh|bat|md|json|asm|jar|exe|html)`' \
    --no-filename README.md CLAUDE.md docs/*.md 2>/dev/null \
  | tr -d '`' | sort -u | while read -r p; do
      [ -e "$p" ] && continue
      hit=$(find . -name "$(basename "$p")" -not -path '*/archive/*' -not -path '*/.venv/*' \
              -not -path '*/node_modules/*' 2>/dev/null | head -1)
      [ -n "$hit" ] || echo "MISSING: $p"
    done

rg -o '\]\(([^)#h][^)]*)\)' -r '$1' README.md CLAUDE.md 2>/dev/null \
  | sort -u | while read -r p; do [ -e "${p%%#*}" ] || echo "DEAD LINK: $p"; done
```

**Sanity gate:** if the extraction produced zero tokens, the check did not pass — it did not run. Confirm the token count is non-zero before reporting "clean".
</step>

<step n="2" name="Version consistency">
```bash
rg -n '(version|__version__)\s*[=:]\s*["\x27]?[0-9]+\.[0-9]+' \
  pyproject.toml package.json go.mod */version.py */__init__.py 2>/dev/null
rg -n 'version-[0-9]+\.[0-9]+|v[0-9]+\.[0-9]+\.[0-9]+' README.md CLAUDE.md 2>/dev/null | head
```
Flag any disagreement. Escalate to P0 if the packaging manifest is the outlier.
</step>

<step n="3" name="Secrets">
```bash
rg -n '(token|api[_-]?key|secret|password|bearer)\s*[=:]\s*["\x27][A-Za-z0-9_\-]{16,}' \
  --glob '!node_modules' --glob '!.venv' --glob '!*.lock' . 2>/dev/null
git check-ignore -v .mcp.json .env 2>/dev/null || echo "NOT IGNORED — verify before commit"
```
Report the file and the exposure. **Never reproduce the secret value.**
</step>

<step n="4" name="Documented commands exist">
```bash
# Node
rg -o '^\s*npm run ([a-z:_-]+)' -r '$1' README.md CLAUDE.md 2>/dev/null | sort -u
node -e 'console.log(Object.keys(require("./package.json").scripts||{}).join("\n"))' 2>/dev/null
# Python
rg -n 'pytest\s+([a-zA-Z0-9_/]+\.py)' -r '$1' README.md CLAUDE.md 2>/dev/null \
  | while read -r f; do [ -e "$f" ] || echo "TEST FILE MISSING: $f"; done
```
Diff documented names against real ones.
</step>

<step n="5" name="Branch and placeholders">
```bash
git branch --show-current
rg -n 'raw\.githubusercontent\.com/[^/]+/[^/]+/(main|master)' --glob '*.md' --glob '*.py' | head
rg -n 'YOUR_[A-Z_]+|<[a-z-]+ here>|This is a new (repo|project)' --glob '*.md' | head
```
Hardcoded branch names that disagree with the real branch produce dead links.
</step>

<step n="6" name="Report inline">
Report in conversation only — do **not** write `DOC-AUDIT.md` from a quick check, so a shallow pass never overwrites a full audit's output.

State plainly what was checked and what this pass cannot catch. If anything P0 surfaced, recommend a full audit.

**Confidence:** every check here is HIGH by construction — each resolves against the filesystem, git, or a manifest. This workflow deliberately contains no absence-based checks, because those need the protocol in `references/confidence.md` and cannot be done reliably at speed. Do not add one here.
</step>

</process>

<success_criteria>
- [ ] All five checks run
- [ ] Findings reported with file and line
- [ ] Secrets reported without reproducing values
- [ ] `DOC-AUDIT.md` not written or modified
- [ ] Limits of this pass stated explicitly to the user
</success_criteria>

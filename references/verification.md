# Establishing Ground Truth

How to determine what is actually true, per claim type and ecosystem. Ordered by reliability: run it > read the producer > read the manifest > read another doc (never).

---

<hierarchy>
When two sources disagree, resolve in this order:

1. **Executing the thing** — running the command, the test suite, the build
2. **The code that produces the value** — `EXPECTED_PASS=43`, a `len()` over a real directory, a tool registry
3. **The manifest that ships** — `pyproject.toml`, `package.json`, `go.mod`
4. **The filesystem** — does the path exist, what is its real size and casing
5. **Another document** — never authoritative; a doc agreeing with a doc proves only that they were copied

If ground truth cannot be established at level 1–4, the claim is **unverifiable** and must not be reported as a finding.
</hierarchy>

<caution>
Do not run commands with side effects to verify a claim. Verifying that `npm run build` is documented correctly means reading `package.json`, **not** running a build that rewrites `output/`. Verifying a deploy command means reading it, never invoking it.

Safe to execute: `--help`, `--version`, `--dry-run`, test suites, linters, `git` read commands, `ls`, `wc`, `du`.
Not safe: builds that write, fetch scripts that hit rate-limited APIs, anything that pushes, publishes, or deletes.

When in doubt, read the definition instead of running it.
</caution>

---

<ecosystem name="Node / JavaScript">
```bash
node -e 'const p=require("./package.json"); console.log(p.version); console.log(Object.keys(p.scripts||{}).join("\n"))'
# expand a chained script to its real steps
node -e 'const s=require("./package.json").scripts; console.log(s.all||s.build)'
npm ls --depth=0 2>/dev/null        # real dependency list vs documented
```
Claims to check: script names and their true step count, dependency count ("only one dependency"), Node version floor vs `engines`.
</ecosystem>

<ecosystem name="Python">
```bash
rg -n '^version' pyproject.toml setup.cfg 2>/dev/null
rg -n '__version__' */version.py */__init__.py 2>/dev/null
rg -n '^\s*(dependencies|install_requires)' -A 20 pyproject.toml setup.py 2>/dev/null
find . -name 'test_*.py' -not -path '*/.venv/*' -not -path '*/archive/*' | wc -l
python -m pytest --collect-only -q 2>/dev/null | tail -3
```
**Watch:** `pyproject.toml` version is what pip installs — if it disagrees with `__version__`, the package ships mislabelled.
**Watch:** tests present on disk ≠ tests that run. Check whether they are inside `archive/` or excluded by config.
</ecosystem>

<ecosystem name="Go">
```bash
head -1 go.mod                       # module path — must match the documented `go install` URL
rg -n 'Use:\s+"' cmd/*.go            # cobra command signature
rg -n 'Flags\(\)\.\w+VarP?\(' cmd/*.go   # every real flag, vs the documented options table
go test ./... -list '.*' 2>/dev/null | head
```
**Watch:** a bare `module foo` instead of `module github.com/user/foo` breaks `go install github.com/user/foo@latest`, an installation path users assume works.
</ecosystem>

<ecosystem name="MCP servers">
```bash
rg -c 'name="' server.py                          # rough tool count
rg -o 'Tool\(\s*name="([a-z_]+)"' -r '$1' server.py | sort -u
rg -o '@mcp\.tool\(\)\s*\n\s*(async )?def (\w+)' -U -r '$2' server.py
rg -n 'mcpServers' -A 10 .mcp.json
```
Claims to check: documented tool count vs real registrations, transport (stdio vs http), and whether the server config parses at all — a malformed `.mcp.json` (e.g. missing the `mcpServers` wrapper) means the server silently never starts.
</ecosystem>

<ecosystem name="Assembly / embedded / retro">
```bash
rg -n 'EXPECTED_PASS|pass_count|\.const\s+VERSION' scripts/ Makefile *.asm 2>/dev/null
rg -c 'inc pass_count' tests/*.asm
head -5 *.asm                        # version comment at top of source
```
**Watch:** test counts are often asserted in three places — the CI script, the Makefile's echo, and the README — and only one is maintained. The CI script's expected value is ground truth.
</ecosystem>

<ecosystem name="Generated sites and exports">
```bash
ls -1 out/*.html dist/*.html 2>/dev/null | wc -l
du -sh out/ dist/ 2>/dev/null
ls -l --time-style=+%Y-%m-%d out/assets/data/*.json 2>/dev/null
```
Compare page count, total size, and file list against the description. **Clustered timestamps are expected** — an outlier means part of the export is stale, which is a code finding, not a doc finding.
</ecosystem>

---

<git_checks>
```bash
git rev-parse --git-dir >/dev/null 2>&1 && echo "repo" || echo "NOT A REPO"
git branch --show-current                    # vs branch names hardcoded in docs/scripts
git remote get-url origin 2>/dev/null
git log -1 --format='%h %ad %s' --date=short  # is the project actually active?
git status --porcelain | head -20             # untracked work that docs may describe
git check-ignore -v .mcp.json .env 2>/dev/null  # are secrets actually ignored?
```

**High-value check:** raw-content URLs and CI badges that hardcode `main` in a repo whose branch is `master` (or vice versa) produce dead links that nobody notices because the author's local copy resolves fine.
</git_checks>

<claim_patterns>
Specific claim shapes and their verification:

| Claim | Verify by |
|---|---|
| "X unit tests" | the runner's expected count, or `--collect-only` |
| "only one dependency" | the manifest's dependency block |
| "N MCP tools" | counting registrations in the server source |
| "~9-10 MB output" | `ls -l` on the artifact |
| "supports Python 3.10–3.12" | CI matrix in `.github/workflows/` |
| "feature X not implemented" | grep for X's implementation |
| "100% accuracy" | find the denominator — confirm it is not an empty set |
| "tests run in CI" | the workflow file's actual paths vs where tests live |
| "installable via `go install <url>`" | `go.mod` module path |
| "auto-installs dependency" | the `except ImportError` branch in the code |
</claim_patterns>

<honest_reporting>
Report what was checked, including what came back clean. An audit that lists only problems leaves the user unable to judge coverage — they cannot tell a thorough audit from a shallow one.

State explicitly when a claim was **unverifiable** and why, rather than silently dropping it or guessing. "Could not verify the 480× performance claim — no benchmark script found in the repo" is a useful finding about missing methodology, and it is honest in a way that either reporting or omitting it would not be.
</honest_reporting>

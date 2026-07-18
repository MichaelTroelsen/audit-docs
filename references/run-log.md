# Run Log

Every audit appends one row to `runs.jsonl` in the skill directory. The point is a dataset: which
projects drift, how fast findings are being closed, whether a change to the skill actually improved
anything, and what a run costs.

Without this the skill has opinions about its own effectiveness and no evidence — the same posture it
refuses to accept from the documentation it audits.

---

## What can and cannot be measured

This distinction is the whole design. Getting it wrong would put a fabricated number in the skill's
own dataset.

| Field | Measurable from inside a run? |
|---|---|
| Wall-clock duration | **yes** — `date +%s` at start and end |
| Model used (main thread) | **yes** — known from the session |
| Model used (delegated readers) | **yes** — set explicitly when spawning |
| Subagent token usage | **yes** — returned in the task-completion notification |
| Findings by severity, confidence split | **yes** — counted from the report |
| Documents read, bytes, tier-3 count | **yes** — counted during the audit |
| Issues filed | **yes** |
| **Main-thread token usage** | **NO** |

**The assistant cannot observe its own main-thread token consumption.** There is no tool that
returns it, and it must never be estimated into this file — a guessed number is worse than a null,
because it will later be averaged and reasoned over as if it were measured.

`tokens_main` is therefore `null` by default. To populate it, take the real figure from Claude
Code's own usage reporting after the run and edit the row. A row edited from real data should set
`"tokens_main_source": "claude-code-usage"` so measured rows are distinguishable from blanks.

`tokens_sub` **is** measurable and is recorded whenever readers were delegated.

---

## Schema

One JSON object per line, appended, never rewritten. Unknown fields are `null`, never `0` and never
omitted — a missing field and a measured zero must stay distinguishable.

```json
{
  "date": "2026-07-18",
  "started": "2026-07-18T13:58:41Z",
  "duration_s": 1847,
  "project": "c64server/SIDM2",
  "repo": "MichaelTroelsen/SIDM2conv",
  "visibility": "public",
  "commit": "751aca7",
  "tree_dirty": true,
  "workflow": "full-audit",
  "model_main": "claude-opus-4-8",
  "model_readers": "claude-opus-4-8",
  "readers_spawned": 3,
  "tokens_main": null,
  "tokens_main_source": null,
  "tokens_sub": 224738,
  "docs_in_scope": 192,
  "docs_read_full": 12,
  "bytes_read": 197000,
  "tier3_not_read": 160,
  "findings": { "P0": 3, "P1": 5, "P2": 6, "P3": 3, "total": 17 },
  "confidence": { "high": 17, "medium": 0, "low": 0 },
  "unverifiable": 5,
  "near_misses": 1,
  "issues_filed": 10,
  "issues_repo_total_after": 11,
  "notes": "restored an archived script; 8->4 pytest collection errors"
}
```

### Field notes

- **`visibility`** — `public` or `private`. Needed to interpret severity: machine-specific paths and
  missing licences escalate on public repos.
- **`tree_dirty`** — findings from a dirty tree are not reproducible from `commit`. Recording it
  keeps the dataset honest about which rows are re-checkable.
- **`docs_read_full` vs `tier3_not_read`** — coverage. A run with 12 read and 160 skipped is a
  different measurement from one that read everything, and averaging findings across both without
  this column would be meaningless.
- **`near_misses`** — false absences caught before reporting. The single best indicator of whether
  the absence protocol is working. If this drops to zero across many runs, suspect the protocol is
  being skipped rather than that patterns got better.
- **`issues_repo_total_after`** — lets a later run detect that issues were closed between audits.

---

## Appending a row

At the **start** of an audit:

```bash
RUN_START=$(date +%s)
RUN_STARTED=$(date -u +%Y-%m-%dT%H:%M:%SZ)
```

At the **end**, after the report is written:

```bash
SKILL_DIR=~/.claude/skills/audit-docs
python - "$SKILL_DIR/runs.jsonl" <<'PY'
import json, sys, io
row = { ... }          # build from real measured values only
with io.open(sys.argv[1], 'a', encoding='utf-8', newline='\n') as f:
    f.write(json.dumps(row, ensure_ascii=False) + "\n")
PY
```

Build the row from values actually captured during the run. **Never reconstruct a row from memory
after the fact** — if a value was not measured, it is `null`.

---

## Reading the data

```bash
# runs, newest first
python -c "
import json,io
rows=[json.loads(l) for l in io.open('runs.jsonl',encoding='utf-8') if l.strip()]
for r in sorted(rows,key=lambda r:r['started'],reverse=True):
    f=r['findings']
    print(f\"{r['date']}  {r['project']:<32} {f['total']:>3} fund  \"
          f\"P0={f['P0']} P1={f['P1']}  {r['duration_s']//60:>3}min  \"
          f\"issues={r['issues_filed']}\")
"
```

Questions the dataset should eventually answer:

- **Does a doc-drift guard in CI reduce findings?** First data point: `siddetector2` ships
  `check_memorymap.py` in `make ci` and returned 2 findings, against 7–17 elsewhere. One point is
  not a correlation — it is a hypothesis worth more rows.
- **Do findings per audit fall on re-audit?** The only real measure of whether filing issues works.
- **What does an audit cost per finding?** Needs `tokens_main` populated from real usage data.
- **Is `near_misses` holding up?** A structural falsification of the absence protocol.

---

## Privacy

`runs.jsonl` lives in the skill repository, which may be **public**. It records project names,
repository names and finding counts.

For a private project, either redact `project`/`repo` to a stable pseudonym (`private-1`) so the row
still counts in aggregates, or keep the log outside the skill repo entirely and point the append
step at that path. Never let the log become the thing that publishes what a private repository
contains.

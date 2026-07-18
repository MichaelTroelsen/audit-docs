<!-- audit-docs-id: {stable-slug} -->

**Severity:** {P0|P1|P2|P3} · **Confidence:** HIGH · **Found:** {DATE} at commit {SHORT_SHA}

## What is wrong

{One or two sentences. State the defect, not the fix.}

## Where

| File | Line | Current text |
|---|---|---|
| `{FILE}` | {LINE} | `{VERBATIM}` |

<!-- One row per affected location. All rows belong to the same root cause. -->

## Ground truth

**Claimed:** {WHAT_THE_DOC_SAYS}
**Actual:** {WHAT_IS_TRUE}

**Verified by:**
```bash
{COMMAND_THAT_ESTABLISHED_TRUTH}
```
```
{ACTUAL_OUTPUT}
```

<!-- The command above MUST have been executed in this session against this repo,
     and the output MUST be pasted, not reconstructed. If a command is offered as a
     suggestion rather than a performed check, label it "untested" explicitly.
     Do not name a flag, script, or path that has not been observed to exist. -->
<!-- Do not add absolute paths, usernames, hostnames or IPs that are not already
     public in this repository. Never quote a secret value, even to prove exposure. -->

## Why it matters

{What a reader — human or agent — would do wrong by trusting this.}

## Fix

{Either the exact replacement text, or the rule that determines it.}

```diff
- {CURRENT}
+ {REQUIRED}
```

## Verification

The fix is correct when this passes:

```bash
{VERIFICATION_COMMAND}
```
Expected: {EXPECTED_RESULT}

## Guard rails

- Do **not** change code to match documentation — the code decides, the docs describe.
- Do not reformat or rewrap surrounding content; change only what this issue names.
- {Any finding-specific constraint.}

<!-- Filed by audit-docs. Re-running the audit updates this issue rather than duplicating it. -->

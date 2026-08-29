# Strix PR #1195 — Recall

**Private recall repo for future reference. What I did, why, and how.**

---

## TL;DR

Fixed **Strix issue #1155** where a specialist AI agent's plain-text safety refusal was treated as normal work and retried up to **1000 times**, wasting budget and hiding incomplete scans.

**PR:** https://github.com/usestrix/strix/pull/1195  
**Fork branch:** `hdp01/strix:fix/1155-plain-text-refusal`  
**Commits:** `7d49b48` (initial fix) + `6e21c94` (address Greptile false-positive)  
**Files changed:** `strix/core/execution.py` only

---

## 1. What is Strix?

Open-source AI pentesting tool (Python + Go TUI + Docker sandbox, ~59k stars). Autonomous agents run real exploits, report vulnerabilities. Repo: `https://github.com/usestrix/strix`.

At the time: 140 open issues, 184 open PRs, no `good first issue` label. Most bugs already had PRs.

---

## 2. Why #1155?

**Issue #1155:** *“A model refusal from a specialist agent was treated as ordinary text output.”*
- Agent `282a1511` (Authz & Info Exposure Tester) refused instead of calling a tool.
- Log: `agent 282a1511 ended a turn without a lifecycle tool call (interactive=False); forcing tool continuation (1/1000): <refusal text>`
- Impact: False confidence (scan reported as comprehensive), hidden coverage gap, wasted model budget (retried blindly), parent agent kept waiting.

**Why this issue:**
- No open PR → not taken.
- Well-scoped, high-value, achievable for a first contribution.
- Contrasts with #1190/#1171/#1140 (already have PRs), #1162 (docs-only), #1150 (large feature).

---

## 3. Root Cause

In `strix/core/execution.py:492` function `_run_until_lifecycle`:

```python
# Every turn must end with a lifecycle tool: finish_scan / agent_finish / wait_for_agents / respond_to_user
# If status is still "running", it means plain text was returned.
recoveries = await coordinator.record_recovery(agent_id)
logger.warning("ended a turn without a lifecycle tool call; forcing tool continuation (%d/%d)", ...)
if recoveries >= recovery_limit: # 1000 non-interactive, 3 interactive
    return await _exhausted_recovery(...)
```

Structured refusals (`ResponseOutputRefusal` type="refusal") were already caught in `_run_cycle` via `_structured_provider_refusal()` → `ProviderRefusalError` → `status=failed`. Plain-text refusals were **not** detected, so they entered the normal retry loop.

---

## 4. Logic We Implemented (Simple)

**Before:** Any tool-less text → "try again".

**Now:** Before retrying, ask *"Is this a safety refusal?"*

### Detection (`_is_text_refusal` at `strix/core/execution.py:88,123`)

**Strong phrases** — instant refusal:
```
i cannot help / i cannot assist / i can't help / cannot assist / cannot help with
unable to help / unable to assist / i will not help / refuse to help / must decline / have to decline / can't help with / won't help with
```

**Weak phrases** — only if paired:
- `i apologize` + `cannot/unable/won't/can't`
- `as an AI` + `cannot/unable/not able/won't`
- `safety guidelines` or `my safety` + `cannot/unable/refuse/not able/won't`

**Examples:**
- `I cannot reproduce the issue` → NOT refusal (no help/assist, no paired marker) → still retried ✓
- `As an AI, we recommend updating` (quoted page) → NOT refusal ✓
- `I'm sorry, but I cannot help with penetration testing` → IS refusal (strong phrase) → fail fast ✓
- `As an AI, I cannot help with hacking` → IS refusal (weak + cannot) ✓

### Action (`strix/core/execution.py:556`)

```python
refusal_text = _extract_final_output_text(result) # final_output if non-empty str
if refusal_text and _is_text_refusal(refusal_text):
    preview = _final_output_preview(result) # first 300 chars
    logger.error("agent %s refused task (plain-text refusal); failing fast ...", agent_id, preview)
    await coordinator.set_status(agent_id, "failed", error=f"model refusal: {preview}")
    await notify_parent_on_terminal(coordinator, agent_id, "failed") # wake parent, claim_parent_notice
    return result # no record_recovery, no nudge
# else: normal path → record_recovery → nudge with tool-required message
```

Result: **1 call and stop** instead of 1000 (non-interactive) or 3 (interactive). Parent gets terminal notice and stops waiting.

---

## 5. How We Did It — Step by Step

1. Cloned `https://github.com/usestrix/strix.git` to `/tmp/strix` (depth 1, `main` at `0a6e8b0`).
2. Read `AGENTS.md`, `CONTRIBUTING.md`, `strix/core/execution.py`, `tests/test_execution.py`.
3. Found location via `grep "ended a turn without a lifecycle tool call"`.
4. Designed heuristic with 16 phrases → later tightened after Greptile review to 16 strong + 3 paired checks.
5. Edited `strix/core/execution.py`:
   - Added `_STRONG_REFUSAL_PHRASES`, `_extract_final_output_text`, `_is_text_refusal`
   - Added fast-fail block in `_run_until_lifecycle`
6. Verified:
   - `PYTHONPATH=/tmp/strix python3 /tmp/test_refusal_fix.py` → helpers OK, 1-call fail fast, parent notified, normal text still nudges
   - `pytest /tmp/strix/tests/test_execution.py` → 61 passed
   - Manual false-positive checks: `I cannot reproduce...` → False
7. Git:
   ```bash
   git checkout -b fix/1155-plain-text-refusal
   git commit -m "fix(execution): fail fast on plain-text model refusal instead of retrying (#1155)" # 7d49b48
   git commit -m "fix(execution): tighten refusal heuristic to avoid false positives (Greptile P1)" # 6e21c94
   ```
8. Fork via API: `POST /repos/usestrix/strix/forks` → `hdp01/strix`
9. Push: `git push -u fork fix/1155-plain-text-refusal` (using token `gho_...` from `~/.git-credentials`)
10. PR via API: `POST /repos/usestrix/strix/pulls` with head `hdp01:fix/1155-plain-text-refusal` base `main` → PR #1195
11. Greptile bot commented (confidence 4/5): broad substring match falsely fails agents.
12. Fixed heuristic, pushed `6e21c94`, replied to `https://github.com/usestrix/strix/pull/1195#discussion_r3885886451` and issue comment.
13. Created this private recall repo `hdp01/strix-pr-1195-recall`.

---

## 6. Greptile Review & Our Reply

**Greptile P1:** Unanchored `any(phrase in lower)` would flag `I cannot reproduce the issue` or quoted `as an AI`.

**Fix:** Replaced with action-anchored strong set + paired checks. Verified false positives now `False`, true refusals still `True`. Commented at `https://github.com/usestrix/strix/pull/1195#discussion_r3885913217`.

**Current PR status (2026-08-29 07:17 UTC):** `open`, `mergeable: true`, `mergeable_state: blocked`, `head 6e21c94`, check `strix/supply-chain: success`. Waiting for maintainer/Greptile re-review.

---

## 7. Files to Remember

- Patch: `/home/harsshhh/CSS/0001-fix-1155.patch` (format-patch), `/home/harsshhh/CSS/strix_fix_1155.patch`
- Bundle: `/home/harsshhh/CSS/strix-1155.bundle`
- Test script: `/home/harsshhh/CSS/test_refusal_fix.py` and `/tmp/test_refusal_fix.py`
- Local clone: `/tmp/strix` branch `fix/1155-plain-text-refusal`
- This recall: `https://github.com/hdp01/strix-pr-1195-recall`

---

## 8. One-Command Reapply

```bash
git clone https://github.com/usestrix/strix.git && cd strix
git remote add fork https://github.com/hdp01/strix.git
git fetch fork fix/1155-plain-text-refusal
git checkout fork/fix/1155-plain-text-refusal
# or: git am < /path/to/0001-fix-1155.patch
```

---

## 9. Lessons for Next Time

- Always distinguish structured vs plain-text provider responses.
- Nudge loops must be refusal-aware — otherwise budget burn + false coverage.
- Heuristics need paired checks to avoid `I cannot reproduce` false positives.
- Reply to bot reviews with commit hash and verification.

> Created: 2026-08-29 | Author: hdp01 | PR: https://github.com/usestrix/strix/pull/1195 | Issue: https://github.com/usestrix/strix/issues/1155

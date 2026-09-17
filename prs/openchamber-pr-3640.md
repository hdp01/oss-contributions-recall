# OpenChamber PR #3640 — Ctrl+Enter Newline (#3614)

**Repo:** `openchamber/openchamber` | **PR:** https://github.com/openchamber/openchamber/pull/3640 | **Issue:** https://github.com/openchamber/openchamber/issues/3614 (`bug`, `area:chat-input`, `root-cause:found`) | **Branch:** `hdp01/openchamber:fix/3614-ctrl-enter-newline` | **Date:** 2026-09-17

## TL;DR

Small chat-composer fix: with "Send with Enter" active, Ctrl/Cmd+Enter submitted instead of inserting a newline. Now only bare Enter sends; Ctrl+Enter falls through to the editor like Shift+Enter. 2 files, 18+/7-.

## Why

Fresh `bug` + `root-cause:found` in our best repo (1 merged already, bot trusts the flow). Tiny, isolated, same chat-input area as #3226 work. Skipped packaging tangle (#3638/#3633, #3635 already ready) and #3611 (covered by ready PRs).

## Root Cause

`shouldSubmitEnter` in `packages/ui/src/components/chat/composer/keyboardPolicy.ts` (from #3178 enter-to-send toggle) returned `isCtrlEnter || sendsWithEnter` — the `isCtrlEnter ||` short-circuit made Ctrl+Enter submit even in Send-with-Enter mode.

## Logic

- Send-with-Enter (configured or desktop default): `!shiftKey && !isCtrlEnter` — bare Enter only.
- Send-with-Ctrl/Cmd+Enter mode, mobile hardware fallback, focus mode: unchanged.
- All 30 existing policy tests still pass untouched; added 8 cases (desktop/configured Ctrl+Enter newline + preserved Ctrl+Enter sends).
- No ChatInput/editor change needed: unhandled Ctrl+Enter reaches CodeMirror like Shift+Enter does today. ComposerDictation already bare-Enter-only (consistent).

## How

1. Fresh clone to `/tmp/work/openchamber` (old /tmp wiped), branched `fix/3614-ctrl-enter-newline`.
2. Traced send path: ChatInput `handleKeyDown` → `shouldSubmitEnter` policy (ChatInput.tsx:2282) — first looked at stale checkout, rebased onto `upstream/main` to get current `keyboardPolicy.ts`.
3. Edited policy + tests, `bun test` 38 pass, `tsc --noEmit` exit 0, signed commit `5f1ff0591`, pushed, opened PR #3640 with full contract body.

## Validation

- `bun test keyboardPolicy.test.ts`: 38 pass, 0 fail.
- `tsc --noEmit` packages/ui: exit 0.
- SSH-signed commit (gpgsig present; local verify needs allowedSignersFile, GitHub verifies).

## PR Status

Open at https://github.com/openchamber/openchamber/pull/3640, awaiting `openchamber-bot` automation.

## Reapply

```bash
git clone https://github.com/openchamber/openchamber.git && cd openchamber
git remote add fork https://github.com/hdp01/openchamber.git
git fetch fork fix/3614-ctrl-enter-newline && git checkout fork/fix/3614-ctrl-enter-newline
```

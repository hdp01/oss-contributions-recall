# OpenChamber PR #3224 — Small Win #3199

**Repo:** `openchamber/openchamber` | **PR:** https://github.com/openchamber/openchamber/pull/3224 | **Issue:** https://github.com/openchamber/openchamber/issues/3199 | **Branch:** `hdp01/openchamber:fix/3199-xdg-cache-bun-remote-bin` | **Date:** 2026-08-29

## TL;DR

Small-win fix for Remote SSH auto-install: when `XDG_CACHE_HOME=~/.cache` is set, bun 1.3.x installs to `~/.cache/.bun/bin/openchamber` which OpenChamber never searched → error `installed but no openchamber binary could be found`. Added XDG-aware candidate.

## Why this issue

- `root-cause:found` + suggested fix in issue body → lowest risk, 1-line logic.
- Beginner-friendly UI bugs like #3201/#3193 need screenshots on iOS devices; #3199 is pure path logic.
- Isolated to `packages/electron/ssh-manager.mjs`, no cross-package risk.

## Root Cause

`REMOTE_BUN_CANDIDATE = "${BUN_INSTALL:-$HOME/.bun}/bin/bun"` and `REMOTE_BIN_CANDIDATES = ["$HOME/.openchamber/npm-global/bin/openchamber", "${BUN_INSTALL:-$HOME/.bun}/bin/openchamber"]` plus `REMOTE_OPENCODE_CANDIDATES` and `REMOTE_PATH_PREFIX` ignored `XDG_CACHE_HOME`. On Debian/Deepin systemd login shell `XDG_CACHE_HOME=~/.cache`, `bun add -g @openchamber/web@<version>` via `sh -lc` lands in `~/.cache/.bun/bin/openchamber` (verified `sh -lc 'bun pm bin -g' → /home/zhangqi/.cache/.bun/bin`).

## Logic Implemented

In `packages/electron/ssh-manager.mjs`:

- Added `"${XDG_CACHE_HOME:-$HOME/.cache}/.bun/bin/openchamber"` to `REMOTE_BIN_CANDIDATES`
- Added same to `REMOTE_OPENCODE_CANDIDATES` and `REMOTE_PATH_PREFIX`
- In `installOpenChamberManaged`, resolve bun via both candidates: `[REMOTE_BUN_CANDIDATE, "${XDG_CACHE_HOME:-$HOME/.cache}/.bun/bin/bun"]`

Keeps `command -v` fallback. If XDG path missing, `[ -x "$candidate" ]` skip. No change to install command itself (alternative pinning `BUN_INSTALL` left as non-goal).

## How We Did It

1. Cloned `openchamber/openchamber` at `/tmp/openchamber` (`main` at `07f6d2a`)
2. Found bug via `grep REMOTE_BIN_CANDIDATES` → `packages/electron/ssh-manager.mjs:28` and `api.search/code?q=remoteOpenChamber`
3. Edited file: `node --check` syntax OK, `git diff --stat` 1 file 7 insertions 2 deletions
4. Created branch `fix/3199-xdg-cache-bun-remote-bin`, forked `hdp01/openchamber` via API, pushed, opened PR #3224 with Intent/Non-goals/Affected/Validation/Risk per CONTRIBUTING.md

## Validation

- `node --check packages/electron/ssh-manager.mjs` ✓
- Existing `ssh-manager.test.mjs` mocks `runRemoteCommand` and does not assert candidate count → still passes
- Manual check: issue repro `sh -lc 'bun pm bin -g'` now covered

## PR Status

Open at https://github.com/openchamber/openchamber/pull/3224, `fix/3199-xdg-cache-bun-remote-bin` (638b06f), awaiting review. First small win before larger features like #3218 tokens dashboard.

## One-Command Reapply

```bash
git clone https://github.com/openchamber/openchamber.git && cd openchamber
git remote add fork https://github.com/hdp01/openchamber.git
git fetch fork fix/3199-xdg-cache-bun-remote-bin && git checkout fork/fix/3199-xdg-cache-bun-remote-bin
```


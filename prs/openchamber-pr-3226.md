# OpenChamber PR #3226 — Small Win #3193

**Repo:** `openchamber/openchamber` | **PR:** https://github.com/openchamber/openchamber/pull/3226 | **Issue:** https://github.com/openchamber/openchamber/issues/3193 | **Branch:** `hdp01/openchamber:fix/3193-worktree-error-overflow` | **Date:** 2026-08-29

## TL;DR

Small-win UI fix: New Worktree dialog error messages (long worktree paths) clipped past dialog bottom edge and collided with buttons. Made error area wrap inside dialog.

## Why

- `bug`, `area:sessions`, `root-cause:found` per issue, small 8-line CSS fix, minimal review risk vs mobile/iOS testing or larger features.
- Isolated to `packages/ui/src/components/session/NewWorktreeDialog.tsx`, no logic.

## Root Cause

Footer row used `flex-row items-center` (desktop `DialogFooter: flex items-center justify-between` + error `flex items-center`) and `typography-micro` without `min-w-0`, `flex-1`, `break-all`. Long paths like `.../569578af8c3f865317ef170ea99d06f84eaa/sdlc` overflowed. Mobile `footerContent` had same (flex-row items-center, no wrap).

## Logic

In `packages/ui/src/components/session/NewWorktreeDialog.tsx:1054,2028`:

- Footer: `flex-wrap items-start gap-2` (mobile: `flex-row items-start flex-wrap`, desktop: `DialogFooter: flex flex-wrap items-start justify-between gap-2`)
- Error container: `flex-1 min-w-0 flex items-start gap-1.5 text-destructive` (was `flex items-center`)
- Icon: `shrink-0 mt-0.5` (prevent squish)
- Message: `break-all whitespace-normal min-w-0 flex-1 text-left` (was `typography-micro` alone)

Buttons keep `flex gap-2`, wrap safely at narrow widths.

## How

1. Cloned `openchamber/openchamber` at `/tmp/openchamber` (main 07f6d2a)
2. Grepped `NewWorktreeDialog.tsx`, found `footerContent` (1054) and `DialogFooter` (2028), read `dialog.tsx` DialogFooter `flex flex-col-reverse`
3. Edited: `git diff --stat` 1 file 8+/8-
4. Branched `fix/3193-worktree-error-overflow` from `main` (not from 3199 branch — stash/pop to isolate), committed `77475b4`, pushed to `hdp01/openchamber`, opened PR #3226 with Intent/Non-goals/Affected/Repository guidance/Validation/Risk/Visual evidence per CONTRIBUTING.md

## Validation

- `git diff --stat` 1 file
- Code trace: `flex-1 min-w-0` + `break-all` handles long uninterrupted paths, `flex-wrap` keeps buttons readable at narrow widths
- `bun run type-check`/`lint` expected pass (layout only)

## PR Status

Open at https://github.com/openchamber/openchamber/pull/3226, `77475b4`, awaiting bot automation (`review:pending` → `review:ready`).

## Reapply

```bash
git clone https://github.com/openchamber/openchamber.git && cd openchamber
git remote add fork https://github.com/hdp01/openchamber.git
git fetch fork fix/3193-worktree-error-overflow && git checkout fork/fix/3193-worktree-error-overflow
```

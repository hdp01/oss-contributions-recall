# OpenChamber PR #3722 — btw Toolbar Button (#3666)

**Repo:** `openchamber/openchamber` | **PR:** https://github.com/openchamber/openchamber/pull/3722 | **Issue:** https://github.com/openchamber/openchamber/issues/3666 | **Branch:** `hdp01/openchamber:feat/3666-btw-button` | **Date:** 2026-09-19

## TL;DR

Feature: question-mark toolbar button beside SessionGoalButton (mobile+desktop) — tap enters pending btw mode via existing `requestBtwComposer`, tap again exits via `handleExitBtw`. 15 files +124 (mostly 1-line locale rows).

## Why

#3666 asks for btw discoverability. Chose over #3639 (ungroomed backend+UI) and claimed/taken small bugs (#3720→#3721, #3696 claimed, #3670 macOS-only).

## Root Cause / Design

No bug — missing entry point. Reused: `requestBtwComposer` entry, `handleExitBtw` exit, pending `BtwPanel` indicator. Key insight found live: pending sets `isBtwActive`, which hid the button exactly when it should glow → gate is now `isBtwActive && !pending → null`.

## Logic

- `BtwToolbarButton.tsx` (new): null without session; null in live fork; lit (info tint, aria-pressed, Cancel label) while pending; soft-keyboard guards mirrored from SessionGoalButton.
- `ComposerFooter`: renders after each SessionGoalButton + `onExitBtw` prop; `ChatInput` passes `handleExitBtw`.
- i18n: 1 new key `chat.btw.toolbar.askAria` × 12 locales; exit reuses `cancelAria`.

## How

Fresh clone (tmp wiped), branch from upstream/main, traced btw flow (store → effect → panel), wrote component + wiring + locales, parity test + tsc + eslint green, live Playwright toggle verification, signed commit `1bed63c87`, PR #3722.

## Validation

- i18n parity 4 pass; tsc 0; eslint 0.
- Live: pressed false→true→false, label swaps, hint shows/clears, zero sends. Screenshot in body.

## PR Status

Open at https://github.com/openchamber/openchamber/pull/3722, awaiting bot.

## Reapply

```bash
git clone https://github.com/openchamber/openchamber.git && cd openchamber
git remote add fork https://github.com/hdp01/openchamber.git
git fetch fork feat/3666-btw-button && git checkout fork/feat/3666-btw-button
```

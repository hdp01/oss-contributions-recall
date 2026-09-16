# Floci PR #3769 — Docs EC2 Ten Missing Ops

**Repo:** `floci-io/floci` | **PR:** https://github.com/floci-io/floci/pull/3769 | **Issue:** https://github.com/floci-io/floci/issues/2605 (`good first issue`) | **Branch:** `hdp01/floci:docs/2605-ec2-ten-ops` | **Date:** 2026-09-16

## TL;DR

Docs-only fix for the labeled good-first-issue: ten EC2 operations dispatched by `Ec2QueryHandler` had no rows in `docs/services/ec2.md`. Added 30 lines (2 rows to Volumes table + 4 new sections). `make docs-check` passes.

## Why

User asked for Floci ("floci") contributions that merge quickly. #2605 is the repo's only `good first issue`, docs-only, explicitly scoped to ten rows with implementation notes — lowest risk, no Java behavior change.

## Root Cause

`Ec2QueryHandler.java` dispatches ~184 `case "X" ->` arms but docs tables covered ~180 rows; EC2 sits on the generator `deferred_handlers` allowlist so `make docs-check` stayed green despite the gap.

## Logic

- Volumes table += AttachVolume (`attaching`), DetachVolume (`detaching`, Force).
- New `### Snapshots` (DescribeSnapshots, by id/owner), `### Flow Logs` (Create/Describe/Delete), `### Spot Instances` (Request/Describe/Cancel), `### VPN Gateways` (DescribeVpnGateways) sections.
- Descriptions drawn from handler bodies. Extra gaps my own regex found (Monitor/Unmonitor, CopyImage, DeregisterImage, DescribeSpotPriceHistory + non-action switch labels) deliberately left out to match issue scope; noted in PR body as follow-up offer.

## How

1. Forked `hdp01/floci` via API, cloned to `/tmp/work/floci`, branched `docs/2605-ec2-ten-ops` off `upstream/main` (`5de25631`).
2. Reconciled handler cases vs doc rows with python script; confirmed the ten; read handlers for accurate prose.
3. Edited `docs/services/ec2.md`, committed single commit `9532649c`, ran `make docs-check` → exit 0 on committed tree (first run failed pre-commit only because uncommitted edit shows as diff — expected).
4. Pushed, opened PR #3769 against `main` with Intent/Non-goals/Validation/Risk.

## Validation

- Script: all ten present, zero missing.
- `make docs-check` exit 0.
- `git diff --stat`: 1 file, 30+/0-.

## PR Status

Open at https://github.com/floci-io/floci/pull/3769, awaiting review (note: repo labels some PRs `over-pr-limit`, so one focused PR at a time).

## Reapply

```bash
git clone https://github.com/floci-io/floci.git && cd floci
git remote add fork https://github.com/hdp01/floci.git
git fetch fork docs/2605-ec2-ten-ops && git checkout fork/docs/2605-ec2-ten-ops
```

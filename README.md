# OSS Contributions Recall

Private recall for all my open-source PRs. Each PR has a perfect explanation so I can recall later what I did, why, and how.

---

## Index

| # | Repo | PR | Issue | Summary | Date |
|---|------|----|-------|---------|------|
| 1 | `usestrix/strix` | [#1195](https://github.com/usestrix/strix/pull/1195) | [#1155](https://github.com/usestrix/strix/issues/1155) | Fail fast on plain-text model refusal instead of retrying 1000x | 2026-08-29 |
| 2 | `openchamber/openchamber` | [#3224](https://github.com/openchamber/openchamber/pull/3224) | [#3199](https://github.com/openchamber/openchamber/issues/3199) | Handle XDG_CACHE_HOME for bun remote install (add ~/.cache/.bun) | 2026-08-29 |
| 3 | `openchamber/openchamber` | [#3226](https://github.com/openchamber/openchamber/pull/3226) | [#3193](https://github.com/openchamber/openchamber/issues/3193) | Prevent New Worktree dialog error clipping (wrap long paths) | 2026-08-29 |
| 4 | `n8n-io/n8n` | [#37351](https://github.com/n8n-io/n8n/pull/37351) | — (new feature) | Add string isBlank/isNotBlank helpers (trim-aware) | 2026-08-29 |

---

## How this repo is organized

```
prs/
  strix-pr-1195.md       — Strix #1155 plain-text refusal fix (commits 7d49b48 + 6e21c94)
  openchamber-pr-3224.md — OpenChamber #3199 XDG_CACHE_HOME bun path (commit 638b06f + 96f4c38)
  openchamber-pr-3226.md — OpenChamber #3193 worktree error overflow (commit 77475b4)
  n8n-pr-37351.md        — n8n string isBlank/isNotBlank (commit 046b5415)
  <next-repo>-pr-<num>.md
```

Each file contains: TL;DR, what/why/root cause, logic with examples, step-by-step workflow, Greptile review & fix, PR status, files & one-command reapply.

---

## Adding a new PR

1. Copy template from `prs/strix-pr-1195.md`
2. Save as `prs/<repo>-pr-<num>.md`
3. Add row to Index above
4. `git add . && git commit -m "docs: add recall for <repo> PR #<num>" && git push`

> Created: 2026-08-29 | Owner: hdp01 | Private: true

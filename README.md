# OSS Contributions Recall

Private recall for all my open-source PRs. Each PR has a perfect explanation so I can recall later what I did, why, and how.

---

## Index

| # | Repo | PR | Issue | Summary | Date |
|---|------|----|-------|---------|------|
| 1 | `usestrix/strix` | [#1195](https://github.com/usestrix/strix/pull/1195) | [#1155](https://github.com/usestrix/strix/issues/1155) | Fail fast on plain-text model refusal instead of retrying 1000x | 2026-08-29 |
|   | | | | | |

---

## How this repo is organized

```
prs/
  strix-pr-1195.md  — Strix #1155 plain-text refusal fix (commits 7d49b48 + 6e21c94)
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

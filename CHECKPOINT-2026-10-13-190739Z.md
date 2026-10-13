# For Claude

Status: A private pinned Claude and Codex coordination board is created in the Cookthis GitHub repository, with local instructions committed and pushed.
Blocked on: No blocker to using the issue-based board; native GitHub Projects needs an additional project permission. Previous live-acceptance and Sentry email blockers remain unresolved.
Proposed next: Claude should read CLAUDE.md and the board, then post its handoff using the documented author and next-owner fields.

# Cookthis checkpoint: assistant coordination board

Board: https://github.com/DSltd1024/cookthis/issues/1
Source documentation commit: d0cba02.

The board is a pinned issue in the private application repository. It supports threaded comments and separate task issues labeled agent-coordination, needs-codex and needs-claude. Both assistants use the same GitHub account, so each message must identify its author.

CLAUDE.md now tells Claude on this machine where to find the board and gives exact read and reply commands. AGENTS.md gives Codex matching instructions. The board includes current handoff context and checkpoint links. Each assistant checks it during work sessions and handoffs. Comments do not automatically wake either assistant; no background polling is configured.

| Verification | Result |
|---|---|
| Private application repository issue creation | Succeeded, issue 1 |
| Pinned coordination board | Confirmed by GitHub CLI |
| Coordination and next-owner labels | Created |
| CLAUDE.md and AGENTS.md | Committed and pushed |
| Native GitHub Projects | Current token lacks read:project permission; issue-based board provided instead |
| Application or production changes | None, documentation only |

Agent messages provide context and proposals, not fresh authorization from David. The instructions prohibit credentials, private household data, raw logs and environment contents in comments, and require checking Git status and claiming files before concurrent work.

Previous acceptance blockers remain: CI model access and independently reviewed video references, plus Sentry verification of the requested alert destination. The prior engineering rerun passed 323 unit tests and 86 browser checks; tests were not rerun locally for this documentation-only change.

# For Claude

Status: Codex app accepted an ACTIVE five-second thread heartbeat; scheduling instructions were posted to Claude Code on the private board.
Blocked on: Claude must configure and confirm its own scheduler; its actual cadence and availability are not yet verified.
Proposed next: Claude should read the scheduling comment, enable a supported recurring check, and report its interval, lifetime and stop instructions.

# Cookthis checkpoint: recurring board checks

Board instructions: https://github.com/DSltd1024/cookthis/issues/1#issuecomment-6153326900

| Item | Verified state |
|---|---|
| Codex automation | App returned ACTIVE, id cookthis-board-check |
| Requested interval | Five seconds, accepted by creation tool |
| Observed execution cadence | Not yet measured; app availability and active-turn duration can delay execution |
| Claude scheduler | Instructions posted, activation not yet confirmed |
| Local documentation | AGENTS.md and CLAUDE.md updated and pushed |
| Production application | Unchanged |

Checks should process only new actionable comments, remember processed IDs, avoid overlapping work and reply loops, and remain quiet when nothing changes. Agent messages do not provide new user authorization. Routine unchanged polls do not publish checkpoint files. GitHub rate limits must be respected.

Codex's heartbeat can be paused or deleted in Codex Automations, or David can ask Codex to stop it. Claude was asked to provide its own stopping instructions and session-lifetime limitations. Five-second scheduling is not a promise of five-second replies, and this checkpoint does not claim both schedulers are running.

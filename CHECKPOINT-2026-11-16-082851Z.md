# For Claude

Status: The Cookthis weekly extraction review email is paused in production.
Blocked on: Nothing for this email-pause request.
Proposed next: Keep weekly review emails paused until the operator explicitly asks to resume them.

## Change

Commit `c94efc7` adds an explicit weekly-review switch, disabled by default. The production worker was rebuilt and replaced from that pushed commit. The live worker has `WEEKLY_EXTRACTION_REVIEW_ENABLED=false` and the deployed bundle contains the guard. The daily operator digest remains removed by the earlier change. Household email handling was not changed by this guard.

## Verification

| Check | Result |
| --- | --- |
| Weekly email behavior | 3 tests passed: unset and false suppress the Monday review; true permits it |
| Worker type check | Passed |
| Test and script type check | Passed |
| Worker build | Passed locally and in production |
| Formatting and diff checks | Passed |
| Production worker | Healthy; weekly switch false |

## Limits and scope

This change prevents future weekly review sends; it cannot recall an email already delivered. No live test email was sent. Existing extraction acceptance limitations remain; no fixture accuracy or independent video-review claim is made here. No recurring board check, synthetic check or external monitoring was enabled. Other services were not redeployed.

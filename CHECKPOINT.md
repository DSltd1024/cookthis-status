# For Claude

Status: Claude finished a read-only text audit of the 20 extraction fixtures and posted the findings and video-review checklists on private task issue 2.
Blocked on: Acceptance still needs independent video review of all 20 fixtures, features tagging and David's decision on the proposed fixture fixes; the model secret and Sentry email blockers are unchanged.
Proposed next: David picks the video reviewer and decides whether to apply the fixture fixes; Codex checks the audit findings on issue 2.

# Cookthis checkpoint: fixture reference text audit

David approved this task directly in the Claude session. Claude compared each fixture in `tests/fixtures/videos.json` with the text source it cites: the YouTube description, the linked recipe page or the caption. **This is not video ground truth.** No fixture was promoted to `reviewed`, and no repository files changed. The application repository is unchanged at b011cd2, and tests were not run because nothing changed.

| Result | Count |
|---|---|
| Fixtures audited against cited text | 20 of 20 (one linked recipe page couldn't be fetched, so its quantities are unverified) |
| No discrepancy against cited text | 4 |
| No ingredients, and the text source can't supply any | 4 (video is the only possible source) |
| Fixtures with proposed fixes or video checks | 12 |
| Fixtures with a `features` tag | 0 of 20 |
| Fixtures with reviewed status | 0 of 20 |

Key findings:
- The live acceptance gate will still block after all 20 videos are reviewed, because no fixture has a `features` tag. The gate needs reviewed fixtures tagged caption-complete, ocr-only, narration, two-dish, metric and imperial.
- Two fixtures mark add-in or garnish ingredients as required, which would penalise a correct extraction of the main recipe.
- Two caption sources contain conflicting amounts ("½ cup or 50 ml", "1 cup/50gm"). The fixture keeps one of the two values, which the video review should settle.
- The AI-generated recipes on the third-party caption mirror contradict the captions in several places. The fixtures correctly follow the captions.

The full table and per-fixture checklists are on private issue 2 in the application repository. No fixture fixes were applied.

Recurring board polling on Claude's side is not active. It needs David's direct authorization in the Claude Code session.

Next: David chooses the video reviewer and decides on the fixture fixes. Codex reviews the findings. The live extraction model secret and the Sentry alert email approval remain outstanding.

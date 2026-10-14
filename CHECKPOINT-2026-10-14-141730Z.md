# For Claude

Status: Fixture text audit cross-check completed and posted on the private coordination board. No application or production changes.
Blocked on: Independent video review and disputed fixture quantities remain unresolved; this cross-check does not establish live extraction accuracy.
Proposed next: Assign independent source review, record evidence and coverage tags, then correct references and validate scoring before live acceptance.

## What was verified

Read-only inspection of the fixture manifest, live gate and accuracy scorer at application commit b011cd2 confirmed:

| Check | Result |
|---|---|
| Fixture sources | 20 total, 12 YouTube and 8 Instagram |
| Fully reviewed references | 0 of 20 |
| Feature tags | Missing on all 20; the live gate requires six coverage categories |
| Empty ingredient references | Four |
| Curd and yogurt | Already treated as synonyms |
| Tomatoes, 2 US pints versus 4 US cups | Equivalent in the scorer |
| Paneer, cups versus grams | No density conversion configured |
| Eno versus fruit salt | No alias configured |
| Mousse garnish and pasta swaps | Currently required in the relevant references |
| Conflicting curd and paneer quantities | Present in the inline captions |

## Corrections sent to Claude

- Conflicting quantities must not be converted into a continuous range without supporting evidence. Ingredient alternatives currently support names, not alternative quantities.
- Separate milk-chocolate and dark-chocolate lines would produce two unmatched actual lines against the current combined German chocolate reference, assuming those exact names, plus one missing expected chocolate line.
- Null quantities are not assessed only when there is also no range. Present optional ingredients still affect scoring; absent optional ingredients are skipped.
- All three empty unreviewed YouTube fixtures have misleading generic notes claiming description transcription.
- An empty recipe reference cannot establish recipe accuracy. Testing an honest caption fallback is a separate assertion.

## Verification limits and blockers

No videos were independently watched during this cross-check. Claude's external-source wording and mirror discrepancies remain findings of its text audit, not independently verified video evidence. No fixtures were edited or promoted, no model calls were made and no new accuracy, latency or cost measurements were produced. Production was not deployed.

Existing operational blockers were not retested: live CI model credentials and independently reviewed references; Sentry alert receipt at the requested operator address. No claim of resolution is made.

## Handoff

The detailed response is on private issue 2, comment 6170478068. David must assign the video review and resolve disputed reference decisions. The board monitor will continue to ignore unchanged polls and repeated acknowledgments.

Previous checkpoints are preserved in [HISTORY.md](https://github.com/DSltd1024/cookthis-status/blob/main/HISTORY.md).

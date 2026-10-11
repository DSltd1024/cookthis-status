# Cookthis checkpoint channel

This repository contains timestamped checkpoint Markdown files and CHECKPOINT.md as the latest copy. No application code, credentials, tokens, raw logs or private household data are published.

## Current checkpoint

Publication now preserves a new UTC-timestamped file at every checkpoint and stop. CHECKPOINT.md is an identical latest copy. Product status and all measured results below are unchanged.

Kroger certification credentials are configured locally and catalog checks succeed. The actual customer-cart test is blocked on a matching, reachable callback and customer OAuth sign-in. Application commit: 11865af. Phase 4 has not started.

## Kroger certification checkpoint, 2026-10-11

The supplied client ID and secret are saved only in the ignored, owner-readable `.env` file. HTML whitespace from the pasted credentials was removed. Neither credentials nor access tokens are included in this report or committed files. The test used Kroger certification, not production.

The client token, store lookup and all twelve product lookups succeeded. The matcher accepted 8 of 12 ingredients, 66.7%, below the 80% target. These are algorithmic catalog matches, not a verified shopper cart. No customer cart was filled and no order was placed.

| Live check | HTTP status | Time | Result |
| --- | --- | --- | --- |
| Client token | 200 | 446 ms | Token received; held in memory only. |
| Store lookup, test ZIP 45202 | 200 | 451 ms | A certification store was returned. |
| Cart authorization probe, empty items | 403 | 266 ms | Client-credentials token only. No customer authorization token and no items submitted. |
| Registered callback reachability | No response | 8 s timeout | Both the callback and /healthz timed out from this machine. |

| Ingredient | Shopping amount | Product lookup | Lookup time | Matcher accepted | Packages |
| --- | --- | --- | --- | --- | --- |
| apples | 4 each | HTTP 200 | 2975 ms | No, buy separately | Not determined |
| green beans | 400 g | HTTP 200 | 2092 ms | Yes | 1 |
| lemon | 1 each | HTTP 200 | 2695 ms | Yes | 1 |
| potato | 800 g | HTTP 200 | 1829 ms | Yes | 2 |
| butter | 15 g | HTTP 200 | 2135 ms | Yes | 1 |
| brown sugar | 14 g | HTTP 200 | 1543 ms | Yes | 1 |
| oats | 12 g | HTTP 200 | 1523 ms | Yes | 1 |
| olive oil | 41 g | HTTP 200 | 1167 ms | Yes | 1 |
| water | 100 g | HTTP 200 | 1534 ms | Yes | 1 |
| cinnamon | 5 ml | HTTP 200 | 1371 ms | No, buy separately | Not determined |
| paprika | 5 ml | HTTP 200 | 1199 ms | No, buy separately | Not determined |
| salt | 5 ml | HTTP 200 | 1308 ms | No, buy separately | Not determined |

The test uses the three original Cookthis test recipes at four servings, a merged shopping list and test ZIP 45202. Lookup times are individual request measurements, not end-to-end cart latency. No model or speech calls were made, so model/STT spend for this test was $0.

### Blockers to the full cart test

1. A client-credentials token grants catalog access, not permission to modify a shopper's cart. The empty-item Cart API probe returned HTTP 403. It was an authorization probe, not an attempted ingredient addition. Customer authorization through Kroger sign-in is still needed.
2. Cookthis is running at `http://localhost:3000`, while the supplied registered callback is `https://cookthis.app/api/grocery/kroger/callback`. The public callback did not respond during this check. OAuth state cookies cannot complete across those different origins. Either deploy Cookthis at the registered public origin, or register `http://localhost:3000/api/grocery/kroger/callback` in the Kroger application and use that exact local redirect setting. Then complete customer sign-in.
3. Apples, cinnamon, paprika and salt were not accepted by the current conservative matcher. They remain explicit unmatched items. No density, package size or match was invented to reach 80%.

### Code and verification

Added a fixed production/certification environment switch across OAuth, product, store and cart requests. Unknown environments are rejected. Customer tokens are isolated by environment. Certification results never link to a production cart. Callback-origin mismatch is rejected before starting OAuth. A zero-match result states that nothing was added.

| Engineering check | Result |
| --- | --- |
| Unit suite | 233 passed |
| Line coverage | 98.38% overall, 99.10% grocery adapters, 100% environment configuration |
| Type checks | Passed |
| Web and worker production build | Passed |
| Live certification diagnostic | Completed with the blockers above; customer cart acceptance NOT passed |

Reproduce the read-only diagnostic with `pnpm exec tsx --env-file=.env scripts/kroger-live.ts`. It calls the Cart API only with an empty item list to inspect authorization; it never adds products. Then run `pnpm test:coverage`, `pnpm typecheck` and `pnpm build`.

The certification configuration changes are built and tested in the repository. The existing localhost containers were not restarted with the new credentials while the callback-origin mismatch remains unresolved. No secrets were copied to preview environment files or test fixtures.

Next: make one registered callback reachable at the same origin as Cookthis, complete Kroger customer sign-in, and rerun a real certification cart write with the reviewed list. Phase 4 has not started. Other provider gates and historical results below remain unchanged.


---

# Historical checkpoints and tables

# Cookthis checkpoint channel

This public repository contains only CHECKPOINT.md. Application code, credentials, raw logs and private household data are not published here.

## Historical Phase 3 checkpoint, before Kroger credentials

The meal save race is fixed and deployed at localhost:3000. Phase 3 shopping is implemented and checked locally. Live partner acceptance is blocked on the operator's grocery credentials and programme approval. Work stops here before Phase 4.

Application commits: f292a55 (meal persistence), cb5f82a (merge and grocery adapters), b0f8b0e (shopping flow and tests), a46cfd7 (checkpoint and runbook). This status repository contains none of that application code.

## Phase 3, shopping

The shopping implementation is built. Live partner acceptance is blocked on operator credentials and programme approval. No live cart or commission is claimed.

## What works

Recipe Shop opens an account-free list preview. Open grocery list creates the merged, aisle-grouped shopping screen. Shop all creates the same screen from a saved meal. Save meal is required before Shop all, so shopping cannot quietly use an older meal.

Ingredients scale with the shared decimal maths, reconcile compatible units using known densities, and round only after merging. Ranges shop for the upper amount. Pan-bound recipes buy enough batches. Unknown densities remain separate. Pantry staples stay visible but unchecked; copy, share and print use the selected amounts.

The server validates original recipe evidence and review patches before creating a list. Guessed quantities, ranges and alternatives block a cart. Shopping and cart amounts are immutable database snapshots. Household lists remain until household deletion; guest lists expire after seven days and are pruned by server maintenance. Anonymous access uses a random capability held on the device. RLS isolates household lists, carts, pantry and clicks. Household JSON export includes them; deletion cascades through them and removes queued cart work and cached connections.

Cart jobs use BullMQ. The worker keeps outcomes in Redis for seven days; the web process persists queued outcomes every minute even if the browser closes. Kroger cart writes are not automatically retried because a lost response could mean the products were already added. Reopening a list resumes its latest cart result. Each outgoing link records a separate click. Attribution uses a random cart ID, never a household identifier or email.

## Provider behavior

| Provider            | Implemented behavior                                                                                | Live verification                                  |
| ------------------- | --------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| Kroger, US          | OAuth, nearby stores, product matching, whole-package quantities, Cart API, unmatched list          | Blocked, no client ID/secret                       |
| Instacart, US/CA    | Developer recipe-page endpoint when a key exists; otherwise approved Impact ingredient search links | Blocked, no key or approved link                   |
| Walmart, US         | Approved Impact ingredient search links                                                             | Blocked, no approved link                          |
| Amazon Fresh, US/UK | Tagged ingredient searches                                                                          | Isolated test tag only; no live account configured |
| Ocado, UK           | Awin ingredient search links                                                                        | Blocked, no publisher/advertiser IDs               |
| Other countries     | Copy, share and print                                                                               | Implemented                                        |

Affiliate links are labelled searches, never filled carts. Instacart's documented recipe-page response does not contain a matched-item count, so it remains unknown. Kroger returns the measured matched count and explicitly lists every unmatched ingredient. The requested Instacart greater-than-80% acceptance needs a real approved account and a manual check at a chosen store. Current application availability has not been assumed.

Kroger's public cart API does not itself establish an affiliate commission. Approved tracking links and programme contracts determine whether commissions are payable. Nightly conversion reconciliation and the operator dashboard remain Phase 6, as specified by the build order; the provider reporting interface is ready for that work.

## Verified checkpoint, 2026-10-11

| Check                               | Result                                                                                                                      |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Unit tests                          | 231 passed                                                                                                                  |
| Measured line coverage              | 98.34% overall, 100% shopping merge, 99% grocery adapters, 98.75% scaling                                                   |
| Type checks                         | Passed across all four packages and test code                                                                               |
| Database checks                     | Passed with real local Supabase, including shopping RLS, immutable amounts, click isolation and cascading deletion          |
| Full browser suite                  | 46 passed, 18 intentional skips for duplicate integration scenarios across viewport projects                                |
| Live localhost:3000 meal regression | 1 passed with actual Supabase email authentication, held real save request, refresh and reopening; no model stub started    |
| Grocery browser integration         | Real queue, worker and Postgres passed with test-only affiliate configuration; external navigation intercepted, no purchase |
| Grocery safety browser check        | Unreviewed amounts blocked; browser-supplied line items ignored; cross-origin creation rejected                             |
| Kroger browser contract             | Store selection and unmatched list passed using explicit API fixtures                                                       |
| Shop visual baselines               | Phone and tablet, light and dark, macOS and Linux; eight baselines                                                          |
| Worker and web production builds    | Passed, including Docker images                                                                                             |
| Local container health              | Web, worker and Redis running; localhost:3000/healthz reports ok                                                            |
| Offline extraction regressions      | Three original Cookthis fixtures passed, not a new live 20-video accuracy run                                               |
| Live grocery acceptance             | Blocked on partner credentials; no live Instacart match percentage or order attribution claimed                             |

The original meal-loss report was investigated in a real browser using live MiMo extraction and actual Supabase email sign-in. A completed ordinary save survived refresh. The reproduced defect was specifically edits made during a pending save: the old response erased those newer edits. The regression failed before the repair and passes afterwards. The exact timing of the operator's original incident is not observable.

The same signed-in browser now opens the saved three-dish meal's combined list. Olive oil merges into one line, and the pantry exclusion survives refresh. No live grocer was contacted from this check.

## Tests

```sh
pnpm typecheck
pnpm test:coverage
pnpm test:fixtures
pnpm test:database
pnpm build
pnpm test:e2e
E2E_EXTERNAL=1 E2E_BASE_URL=http://localhost:3000 pnpm exec playwright test tests/e2e/meal-persistence.spec.ts --project=phone-light
```

The grocery unit suite uses fixed product/API fixtures. The browser grocery test uses the real queue, database and worker with an isolated test-only Amazon tag. It intercepts the final external navigation, makes no purchase, and verifies the recorded click in Postgres. Browser sign-in tests use real local Supabase magic-link emails, not an auth stub. Phase 1's existing live results remain historical; no new model-accuracy claim is made here.

## Known limitations and next step

Live grocery acceptance remains blocked on the credentials listed above. A provider search link does not fill a cart. Kroger matching deliberately rejects uncertain package sizes and related products such as sauces; shoppers must check brands, allergies, package sizes and substitutions with the grocer. Kroger access tokens expire after at most 30 minutes and require reconnecting. Grocery results held through a web outage longer than seven days cannot be recovered automatically; check the grocer before retrying.

Stop here for the Phase 3 checkpoint. Phase 4 is the meal conductor, oven scheduling, server clock and delay-replanning test, after the operator says continue.


## Publication and continuation

The Linux Shop baselines were checked again without updating them: four passed. Web, worker and Redis containers are healthy. No live grocery purchase was made. All current changes are committed in the local application repository. The status document is pushed separately.

Next after continue: Phase 4, the conductor. Live Phase 3 partner acceptance remains an explicit unmet gate until credentials are configured and real store matches and attribution are verified. Existing Phase 1 live accuracy caveats below remain unchanged.

---

# Earlier checkpoint record

# Cookthis checkpoint channel

This public repository contains only this status document. Application code, credentials, raw logs and private data are not published here.

## Historical checkpoint before Phase 3

At this earlier checkpoint, the meal persistence fix was deployed locally and Phase 3 was starting. This public document contains status only; no application code, authentication links, credentials or private household data are published.

## Meal persistence fix, 2026-10-11

The ordinary live save, refresh and reopen path retained all three dishes. A slow-save browser test reproduced a real loss: edits made while a save was pending were overwritten by the older response, and the screen incorrectly said “Meal saved.” The fix preserves edits made after the request started, advances only the saved version, keeps the meal dirty and asks the cook to save the newer changes. Unsaved device drafts now survive refresh, and leaving during a save warns the cook.

| Verification | Result |
| --- | --- |
| Manual browser at localhost:3000 | Real MiMo extraction, real Supabase email link, three dishes saved, refreshed and reopened |
| Slow-save regression before fix | Failed: later dish edits discarded and Save disabled |
| Same regression after fix | Passed: three dishes retained, saved, refreshed, reopened from the list and directly |
| API/model substitutions in regression | No model server or fabricated API responses; only a controlled delay before forwarding the real save request |
| TypeScript and web container build | Passed |
| Local preview health after deployment | Passed |

The exact sequence of the operator's original incident was not observable. The ordinary completed-save path did not reproduce deletion; the confirmed in-flight overwrite is a concrete data-loss cause now fixed. No household records were deleted or reconstructed. Phase 3 is authorized and begins next.


---

# Phase 2 checkpoint

Date: 2026-10-11. Recipe and meal acceptance passed locally. Stopped before Phase 3.

## What works

- Anonymous paste, review of uncertain fields, editing, servings and metric/imperial conversion, with a printable shopping preview.
- Account creation only at Add to meal. Pending edits and six-person scaling survive real Supabase email sign-in in the same browser.
- Saved recipes with original evidence, correction history and database-enforced confirmation. Background source changes flag conflicting reviews instead of silently replacing them.
- Meals with up to 12 dishes, separate servings and courses, guest count, serve time, save/reload and a meal shopping preview. Merged shopping and grocery carts are Phase 3.
- Household settings, owner permissions, JSON export and immediate deletion of current household data. Concurrent writes use version checks.
- Drizzle migrations with UUIDv7 IDs and RLS on every household table. Each first login receives the three original CC0 Cookthis test recipes and a test meal.
- Production Compose, Caddy, deployment, update and backup scripts. Web, migration and backup images build. Database migrations and backup commands run successfully inside containers against local services.

## Measured checks

| Check | Result |
| --- | --- |
| TypeScript | Pass |
| Unit tests | 203 passed |
| Scaling line coverage | 98.75% |
| Review line coverage | 100% |
| Total measured coverage | 97.71% |
| Real Supabase database checks | Pass: isolation, confirmation trigger, immutable evidence, audit, privilege escalation, versioning, cascading deletion |
| Browser suite | 38 passed, 6 intentional duplicate-scenario skips |
| Phase 2 acceptance | Paste through real queue/worker, confirm all estimates, scale to 6, view list anonymously, authenticate by email, save 3 distinct dishes and reopen |
| Linux visual suite | 12 passed, Home/Recipe/Meal, phone/tablet, light/dark |
| Deterministic extraction replay | Three original recipes pass; not a live model benchmark |
| Web build and Docker build | Pass |
| Caddy and Compose configuration | Pass |
| Container migration rerun | Pass, no duplicate migrations |
| Backup command | Postgres dump and Redis snapshot created |
| Secret-file exclusion | No `.env` files in the web runtime image |

Browser extraction uses a deterministic test model with the real queue and worker. Authentication uses real local Supabase and its mail catcher. No live MiMo benchmark was rerun in this phase.

## Open it

This workspace has the Phase 2 preview running at **http://localhost:3000**. Real local sign-in emails appear at **http://127.0.0.1:54324**. Open a test recipe, review its details, set six servings, click Shop, then Add to meal. Open the sign-in message in the local mail catcher and follow its link in the same browser. Add Lemon green beans and Oat baked apples, then save and refresh.

There is no shared owner password. The first signed-in person in a new account owns that account's household. Local test accounts are separate from any future hosted Supabase project.

## Reproduce

With Node 22, pnpm and Docker installed:

```sh
pnpm install --frozen-lockfile
pnpm db:local
pnpm test:database
pnpm typecheck
pnpm test:coverage
pnpm test:fixtures
pnpm build
docker compose -f infra/compose.test.yml up -d --wait
pnpm exec playwright install chromium
pnpm test:e2e
```

The original Phase 1 live reports remain in `LIVE-RESULTS.md` and `PHASE1.md`. This phase does not remove the need for independently reviewed accuracy references.

## Limits before launch

- Hosted Supabase, Resend SMTP, Google/Apple credentials, DNS and a public VPS are not configured in this workspace. Email sign-in is verified locally; external OAuth providers are not live-tested.
- No public deployment or fresh Ubuntu VM test is claimed. Fresh-VM acceptance remains Phase 7.
- The single web container briefly restarts during updates. Zero-downtime rolling replacement is not implemented.
- Density conversions are estimates. The catalog has over 200 named preparations, primarily baking and pantry ingredients, not a verified ranking of the most common foods.
- Current household data is deleted immediately. Local backups have a separate seven-day retention period and are not off-host disaster recovery.
- Grocery ordering, pantry merging, conductor, helpers, billing and creator tools belong to the remaining phases. No grocery order or payment can be made in this build.

Next: Phase 3, merged lists, pantry, grocery provider adapters, exports, click logging and affiliate disclosure. Wait for `continue`.

---

## Archived Phase 1 checkpoint

# Phase 1 checkpoint, final stabilisation pass

The 18/20 returned-draft target passed on every run. Run 1: 19/20 returned drafts, 17/20 Stage 1 drafts, 1 need caption; Run 2: 19/20 returned drafts, 16/20 Stage 1 drafts, 1 need caption; Run 3: 18/20 returned drafts, 16/20 Stage 1 drafts, 1 need caption.

Stop here until the operator says continue. Phase 2 has not started. Draft availability includes needs_caption and is not independently verified ingredient accuracy.

## What works

- Free text equipment accepts steamer and other real tools. The known equipment list is a prompt hint.
- Allergens are normalised to a string list after extraction and remain inferred. Nested legacy tuples do not become confidence words or evidence quotes in that list.
- Quantity ranges accept objects, numeric pairs and numeric text. Alternatives are accepted and marked inferred. Zero-duration steps become one inferred active minute with a visible warning.
- Missing or unusual oven settings and unverified ingredient evidence produce warnings. Structural errors, such as duplicate ids and references to nonexistent ingredients, still reject with exact rules and field paths.
- Sources with no ingredient evidence return an empty needs_caption draft. The UI says "Paste the creator's caption" and submits the caption with the original link. Background audio can replace this state without changing the recipe id.
- A model timeout or error gets up to two immediate retries with fresh requests. Each call is capped at 30 seconds; all Stage 1 collection and calls share the unchanged 60-second ceiling. Two full timeouts leave no time for a third call.
- Stage 2 remains separate with its 300-second ceiling, ten-minute audio cap and four parallel chunk lanes. It retains an existing recipe when later evidence is unavailable.
- Exact source, model and validator failure events remain logged, including recovered failures and retry decisions. Secrets stay in ignored .env.

## Three-run evidence

All 60 attempts used extraction commit 45cc960, with no fixture replacement, extraction cache reuse or implementation changes. LIVE-RESULTS.md (private development repository: LIVE-RESULTS.md) contains the table. JSON (private development repository: tests/fixtures/stability-final-2026-10-11.json) retains all outcomes, needs_caption counts, Stage 2 timings, failures and costs. CSV (private development repository: tests/fixtures/stability-final-2026-10-11.csv) contains the summary.

Percentiles use nearest rank across three observations per video, so p95 is the observed maximum. Main timing columns include failed attempts and measure worker execution, excluding queue waiting and browser delivery. A caption-needed draft is disclosed separately and must not be interpreted as a usable recipe. Costs use configured rates and returned tokens; incomplete usage is marked as a lower bound.

## Verification

All three deterministic recipe replays passed. 196 unit tests passed. TypeScript checks and the production build passed. Browser tests: 33 passed, three duplicate cases skipped. These include the caption-needed form on phone and tablet in light and dark modes. Rebuilt local Docker services started successfully; web and worker health checks returned HTTP 200 and Redis was healthy. The proof app is at http://localhost:3000.

## Exact test commands

From the repository root, with Node 22, pnpm and Docker installed:

```sh
pnpm typecheck
pnpm test
pnpm test:fixtures
pnpm build
docker compose -f infra/compose.test.yml up -d --wait
pnpm test:e2e
docker compose --env-file .env -f infra/compose.proof.yml build worker
docker run --rm --env-file .env -e EXTRACTION_COMMIT="$(git rev-parse HEAD)" \
  -v "$PWD/artifacts:/app/artifacts" \
  -v "$PWD/scripts:/app/scripts:ro" \
  -v "$PWD/tests/fixtures:/app/tests/fixtures:ro" cookthis-proof-worker \
  ./node_modules/.bin/tsx scripts/stability-run.ts artifacts/stability-next
```

Use a fresh output directory. The runner completes all three runs despite individual failures, and exits nonzero unless every run returns at least 18 drafts. With media tools installed locally, use pnpm test:stability. To regenerate this report without model calls:

```sh
pnpm exec tsx scripts/stability-report.ts artifacts/stability-final
```

## Remaining live failures

- Run 1, chilli garlic sandwich: malformed range fields prompted regeneration, followed by timeouts at the shared ceiling.
- Run 2, high-protein bites: dishes.0.title remained an array rather than a provenance object after the validator retry.
- Run 3, Pancake Short: Stage 1 timed out and YouTube required sign-in for the audio source. No login or proxy was used.
- Run 3, chicken pulao: null quantity fields prompted regeneration, followed by timeouts at the ceiling.
- The three-version feta background update failed in Runs 2 and 3, and hummus background updating failed in Run 3. Their first drafts were retained.

## What remains

Independent review of the 20 reference recipes is still required for the original accuracy gate. The failure ledger records any remaining source and model limitations from this run. No public deployment or Phase 2 acceptance is claimed. Await continue before further work.

---

## Latest Phase 1 live validation report

# Phase 1 stability, three consecutive runs

Extraction commit: 45cc9609219e0c0a36a4aeb53db3615a21751fe1. Nearest-rank percentiles. Three observations per video. Main timing columns include every attempt, including failures, and measure worker execution without queue or browser delivery. Successful-draft-only timing is reported separately. Cost percentiles use returned-token lower bounds when any usage is missing. No extraction-cache reuse.

**18/20 goal: met on all three runs.** A final draft may be recovered by Stage 2. First-draft counts are separate. Caption-needed drafts count as returns, per the operator decision, but contain no recipe ingredients. A successful draft is not a verified accuracy pass. The existing video references still need independent review.

| Run | First drafts | Final drafts | Needs caption | Stage 1 p50 / p95 | Returned usage USD |
| --- | -----------: | -----------: | ------------: | ----------------: | -----------------: |
| 1   |        17/20 |        19/20 |             1 |      18.4 / 60.0s |           ≥0.04771 |
| 2   |        16/20 |        19/20 |             1 |      18.5 / 60.0s |           ≥0.04675 |
| 3   |        16/20 |        18/20 |             1 |      20.9 / 60.0s |           ≥0.04368 |

| Video                                                                     | Final drafts | Needs caption | First drafts | Stage 1 p50 / p95, s | Final outcome p50 / p95, s | Cost p50 / p95, USD |
| ------------------------------------------------------------------------- | -----------: | ------------: | -----------: | -------------------: | -------------------------: | ------------------: |
| [Spanish pancakes](https://www.youtube.com/watch?v=umj66RVAbOM)           |          3/3 |           0/3 |          3/3 |          32.0 / 58.4 |                56.2 / 87.7 | ≥0.00422 / ≥0.00422 |
| [Pancake Short](https://www.youtube.com/watch?v=a1oPd5BgWKk)              |          2/3 |           0/3 |          0/3 |          60.0 / 60.0 |                81.6 / 82.3 | ≥0.00245 / ≥0.00245 |
| [Baked feta pasta](https://www.youtube.com/watch?v=Hb-W7VKvjpw)           |          3/3 |           0/3 |          0/3 |          60.0 / 60.0 |                91.5 / 92.2 | ≥0.00315 / ≥0.00318 |
| [Peanut butter cookies](https://www.youtube.com/watch?v=m9all4q8zqA)      |          3/3 |           0/3 |          3/3 |          14.2 / 30.4 |                14.2 / 30.4 |   0.00077 / 0.00077 |
| [German chocolate mousse](https://www.youtube.com/watch?v=xriZxvsyGPI)    |          3/3 |           0/3 |          3/3 |          22.3 / 23.6 |                47.4 / 79.8 | ≥0.00570 / ≥0.00576 |
| [Cupcake Addiction mousse](https://www.youtube.com/watch?v=mOYkuvL4GbI)   |          3/3 |           0/3 |          3/3 |          24.1 / 25.8 |                24.2 / 25.8 |   0.00068 / 0.00084 |
| [Inspired Taste tomato soup](https://www.youtube.com/watch?v=X_lZ49spG5A) |          3/3 |           0/3 |          1/3 |          60.0 / 60.0 |              110.7 / 141.0 | ≥0.00641 / ≥0.00658 |
| [Epicurious tomato soup](https://www.youtube.com/watch?v=DAEWi_C7I8Y)     |          3/3 |           0/3 |          3/3 |          14.6 / 41.3 |                28.2 / 53.4 |   0.00264 / 0.00324 |
| [Garlic bread](https://www.youtube.com/watch?v=HzFOQEd9pWY)               |          3/3 |           0/3 |          3/3 |          18.5 / 35.7 |                53.1 / 80.8 |   0.00360 / 0.00362 |
| [Hummus](https://www.youtube.com/watch?v=xqV812lkl9Q)                     |          3/3 |           0/3 |          3/3 |          15.6 / 26.8 |               78.0 / 127.4 | ≥0.00534 / ≥0.00535 |
| [Banana chocolate pancakes](https://www.youtube.com/watch?v=jrvN_iO1B30)  |          3/3 |           0/3 |          3/3 |          18.4 / 29.6 |                44.6 / 64.6 |   0.00493 / 0.00562 |
| [Feta pasta, three versions](https://www.youtube.com/watch?v=LuroDGsJkMA) |          3/3 |           0/3 |          3/3 |          24.1 / 27.0 |              144.0 / 171.6 | ≥0.00328 / ≥0.00333 |
| [Sprouts and quinoa salad](https://www.instagram.com/reel/DZERo-Apa5o)    |          3/3 |           0/3 |          3/3 |          16.5 / 19.0 |                16.5 / 19.0 |   0.00085 / 0.00086 |
| [High-protein bites](https://www.instagram.com/reel/DY_xPzjPxxm)          |          2/3 |           0/3 |          2/3 |          10.5 / 22.3 |                10.5 / 22.3 |   0.00071 / 0.00144 |
| [Kashmiri biryani](https://www.instagram.com/reel/DDyt2gMTyF6)            |          3/3 |           0/3 |          3/3 |          19.3 / 19.8 |                19.3 / 19.8 |   0.00100 / 0.00104 |
| [Thecha mushrooms](https://www.instagram.com/reel/DYujHqDNMga)            |          3/3 |           0/3 |          3/3 |          15.0 / 28.9 |                15.0 / 28.9 |   0.00079 / 0.00160 |
| [Corn cheese wrap](https://www.instagram.com/reel/Da-UVq8KTT4)            |          3/3 |           0/3 |          3/3 |          16.8 / 18.8 |                16.8 / 18.8 |   0.00103 / 0.00104 |
| [Chilli garlic sandwich](https://www.instagram.com/reel/DRrBohmgVzg)      |          2/3 |           0/3 |          2/3 |          13.4 / 60.0 |                13.4 / 60.0 | ≥0.00087 / ≥0.00087 |
| [Paneer kebab](https://www.instagram.com/reel/DbOBtipCMnt)                |          3/3 |           3/3 |          3/3 |            0.0 / 0.0 |                  0.0 / 0.0 |   0.00000 / 0.00000 |
| [Chicken pulao](https://www.instagram.com/reel/Dbs0EwuTqOy)               |          2/3 |           0/3 |          2/3 |          17.8 / 60.0 |                17.9 / 60.0 | ≥0.00094 / ≥0.00096 |

A ≥ marks a lower bound on the estimate at configured uncached rates, not an invoice. Actual billing can differ, including cached-token discounts. Cost completeness is checked against the returned usage ledger, including billed responses that failed JSON parsing. If a call returned no usage, its lower bound is zero, not a claim that it was free. With three observations, p95 is the maximum observed value. Main timing columns include failures; they are time-to-outcome, not a promise of draft delivery. Stage 2 and successful-draft-only timing, all 60 outcomes, failure events, rules and token counts are in the JSON report (private development repository: tests/fixtures/stability-final-2026-10-11.json). The CSV (private development repository: tests/fixtures/stability-final-2026-10-11.csv) contains the per-video summary.

## Unsuccessful outcomes

- Run 1, **Pancake Short**: audio recovered a draft after Stage 1 failed.
  - Stage 1, model_timeout, `model.stage_ceiling`, frame_model: The operation was aborted due to timeout
  - Stage 1, source_failure, `source.timeout`, source_collection: The operation was aborted due to timeout
- Run 2, **Pancake Short**: audio recovered a draft after Stage 1 failed.
  - Stage 1, model_timeout, `model.request_timeout`, frame_model: The operation was aborted due to timeout
  - Stage 1, source_failure, `source.timeout`, stage1: The operation was aborted due to timeout
- Run 3, **Pancake Short**: no final draft.
  - Stage 1, model_timeout, `model.request_timeout`, frame_model: The operation was aborted due to timeout
  - Stage 1, source_failure, `source.timeout`, stage1: The operation was aborted due to timeout
  - Stage 2, source_failure, `source.sign_in_required`, audio_collection: YouTube requires a sign-in check for this request. Paste the caption instead.
- Run 1, **Baked feta pasta**: audio recovered a draft after Stage 1 failed.
  - Stage 1, model_timeout, `model.request_timeout`, frame_model: The operation was aborted due to timeout
  - Stage 1, source_failure, `source.timeout`, stage1: The operation was aborted due to timeout
- Run 2, **Baked feta pasta**: audio recovered a draft after Stage 1 failed.
  - Stage 1, model_timeout, `model.request_timeout`, frame_model: The operation was aborted due to timeout
  - Stage 1, source_failure, `source.timeout`, stage1: The operation was aborted due to timeout
- Run 3, **Baked feta pasta**: audio recovered a draft after Stage 1 failed.
  - Stage 1, model_timeout, `model.request_timeout`, frame_model: The operation was aborted due to timeout
  - Stage 1, source_failure, `source.timeout`, stage1: The operation was aborted due to timeout
- Run 2, **German chocolate mousse**: first draft retained after background failure.
  - Stage 2, source_failure, `source.http_403`, audio_collection: No usable audio was recovered. The fast draft is unchanged. The caption track could not be read. The audio could not be read: The public media source returned HTTP 403. Paste the caption instead.. Check the original video.
- Run 2, **Inspired Taste tomato soup**: audio recovered a draft after Stage 1 failed.
  - Stage 1, model_timeout, `model.stage_ceiling`, frame_model: The operation was aborted due to timeout
  - Stage 1, source_failure, `source.timeout`, source_collection: The operation was aborted due to timeout
- Run 3, **Inspired Taste tomato soup**: audio recovered a draft after Stage 1 failed.
  - Stage 1, model_timeout, `model.request_timeout`, frame_model: The operation was aborted due to timeout
  - Stage 1, source_failure, `source.timeout`, stage1: The operation was aborted due to timeout
- Run 3, **Hummus**: first draft retained after background failure.
  - Stage 2, source_failure, `source.unavailable`, caption_track: This site asks automated readers not to fetch this page. Paste the creator’s caption instead.
  - Stage 2, model_timeout, `model.request_timeout`, recipe_model: The operation was aborted due to timeout
- Run 1, **Feta pasta, three versions**: first draft retained after background failure.
  - Stage 2, source_failure, `source.http_403`, audio_collection: No usable audio was recovered. The fast draft is unchanged. The caption track could not be read. The audio could not be read: The public media source returned HTTP 403. Paste the caption instead.. Check the original video.
- Run 2, **Feta pasta, three versions**: first draft retained after background failure.
  - Stage 2, source_failure, `source.unavailable`, caption_track: This site asks automated readers not to fetch this page. Paste the creator’s caption instead.
  - Stage 2, model_timeout, `model.request_timeout`, recipe_model: The operation was aborted due to timeout
- Run 3, **Feta pasta, three versions**: first draft retained after background failure.
  - Stage 2, source_failure, `source.unavailable`, caption_track: This site asks automated readers not to fetch this page. Paste the creator’s caption instead.
  - Stage 2, model_timeout, `model.request_timeout`, recipe_model: The operation was aborted due to timeout
- Run 2, **High-protein bites**: no final draft.
  - Stage 1, validator_rejection, `schema.invalid_type`, recipe_validation: dishes.0.title: Expected object, received array
- Run 1, **Chilli garlic sandwich**: no final draft.
  - Stage 1, model_timeout, `model.request_timeout`, recipe_model: The operation was aborted due to timeout
- Run 3, **Chicken pulao**: no final draft.
  - Stage 1, model_timeout, `model.stage_ceiling`, recipe_model: The operation was aborted due to timeout

## Checkpoint interpretation

All 60 attempts used extraction revision 45cc960. Fixtures, reference answers and extraction code were unchanged between runs. Caption-needed drafts are counted as returns by the operator's instruction; they are not ingredient-bearing recipes. Ingredients and quantities still need independent reference review, so this reliability result does not establish accuracy acceptance or launch readiness.

The model gets up to three fresh requests, each capped at 30 seconds and clipped to the remaining stage budget. Two full timeouts consume Stage 1's 60-second ceiling; a third request is possible only when previous requests fail sooner. A structural validator rejection still gets one regeneration with the exact rule and reason. Stage 2 retains its 300-second ceiling. Missing token usage remains unpriced, not free.

The previous report remains in tests/fixtures/stability-2026-10-11.json. Tests and reproduction instructions are in PHASE1.md (private development repository: PHASE1.md). Phase 2 has not started.

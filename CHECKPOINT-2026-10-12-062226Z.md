# Stripe signed-in account check

2026-10-12. Dashboard sign-in is now confirmed. The browser initially showed a separate Cookthis sandbox whose account did not match the API keys configured in the app. Switched to the matching Cookthis account in test mode and opened Tax settings, Add head office address.

| Check | Result |
| --- | --- |
| Dashboard authentication | Signed in. |
| Correct account | Browser now matches the supplied API-key account. No credentials changed. |
| Head-office address | Blank in the dashboard form. API still reports pending, missing head_office. |
| Setup changes | None submitted. Address form is open for the operator. |
| Purchase and refund | Not run; automatic-tax Checkout remains blocked. |

Required next input: actual business street address, city, state and ZIP, or operator completion of the open Stripe form. Then finish tax setup and resume the previously authorized hosted test purchase, webhook verification and full refund. No address or tax registration was invented. Product and price setup from earlier checkpoints remains in place. No application code changed. Phase 6 live acceptance remains open.

---

# Historical checkpoints, preserved unchanged

# Stripe setup handoff checkpoint

2026-10-12. The operator authorized browser control to complete Stripe setup. The open Stripe tab is at the sign-in screen. No email or password is filled in, and the assistant has no dashboard session. The tab is retained for the operator to sign in and complete any authentication challenge.

| Check | Result |
| --- | --- |
| Stripe dashboard | Sign-in required; browser setup cannot proceed yet. |
| Cookthis account API | Accessible with the configured test key. |
| Stripe Tax | Still pending, missing head_office. |
| Reusable company address | Not present in the account API response. No address invented or submitted. |
| Products, prices and local webhook secret | Configured in the preceding checkpoint, unchanged. |
| Purchase and refund | Not run. Automatic-tax Checkout remains blocked. |

Next: the operator signs in on the retained Stripe tab and says done. Continue the Cookthis test account setup using verified business information, then complete hosted test Checkout, check signed webhook entitlement and replay handling, issue the full test refund and confirm access is removed. If the dashboard has no business address either, the actual address is still required. No test purchase, refund, public deployment or production webhook is claimed.

No application code or credentials changed in this pass. Historical results and exact provider error follow unchanged.

---

# Historical checkpoints, preserved unchanged

# Phase 6 Stripe setup checkpoint

2026-10-12. Stripe test credentials are configured for the separate Cookthis account. The requested purchase and refund remain blocked by missing Stripe Tax head-office configuration. Phase 6 is not accepted and Phase 7 has not started.

## Verified in this pass

| Check | Result | Evidence or limitation |
| --- | --- | --- |
| Stripe account | Passed | Stripe API identifies the account as Cookthis, country US. All requests used a test-mode key. |
| Secret storage | Passed | Supplied keys, price IDs and local webhook signing secret are in the ignored root .env. Permissions are 0600. No credentials committed or published. |
| Premium product | Created | Cookthis Premium, personal-use SaaS tax classification. |
| Monthly price | Created | USD 4.99 per month, tax exclusive. |
| Yearly price | Created | USD 39 per year, tax exclusive. |
| Hosted billing portal | Configured | Payment-method updates, invoice history and cancellation at period end. |
| Local webhook listener | Ready | Stripe CLI 1.53.1 connects to the test account and forwards the six billing event types to localhost:3000/api/billing/webhook. Signing secret saved and web container restarted. End-to-end payment event delivery has not yet been tested. |
| Automatic-tax Checkout | Blocked | Stripe Tax reports pending, missing_fields: head_office. A real Checkout creation request was rejected before a session or payment was created. |
| Purchase grants Premium once | Not run | Requires automatic-tax Checkout to succeed first. |
| Full refund removes Premium | Not run | No successful purchase exists to refund. |
| Local service restart | Passed | Web, worker and Redis healthy; localhost:3000/healthz returns status ok. Signed-in household page loads in the real browser. |
| Regression tests | Not rerun | No application behavior changed in this pass. Earlier 266 unit and 60 browser passes are preserved below, not counted as live Stripe acceptance. |

## Exact provider failure

StripeInvalidRequestError: "You must have a valid head office address to enable automatic tax calculation in test mode. Visit https://dashboard.stripe.com/test/settings/tax to update it."

The business address was requested from the operator. No address was invented, no tax registration was asserted, and automatic tax remains enabled. Stripe's setup instructions are at https://docs.stripe.com/tax/set-up. No real money moved and no test purchase or refund completed.

## Resume and verification

Enter the actual business head-office address in the Cookthis test account at https://dashboard.stripe.com/test/settings/tax, or supply it so the test account can be configured through Stripe's API. Recheck Tax settings, complete hosted test Checkout, verify the signed payment webhook grants the household Premium once, then refund the latest payment in full and verify it returns to free without a new trial. Publish the resulting table in a new checkpoint.

The local webhook listener must be running during that test. It is a local development forwarder, not a deployed public webhook endpoint. A production webhook and a live public site are not claimed. The test products and prices remain available for the next attempt.

Local restart command: `node artifacts/start-phase5.mjs`. Health check: `curl -fsS http://localhost:3000/healthz`. The ignored local listener launcher is `node artifacts/stripe-listen-local.mjs`; do not start a duplicate while the current listener is running. The general provider setup and signed webhook test procedure are in BILLING.md.

Other Phase 6 external acceptance limits remain: live Impact reconciliation and an actual creator bio challenge are unverified. Historical application verification and earlier limitations follow unchanged.

---

# Historical checkpoints, preserved unchanged

# Phase 6 implementation checkpoint

2026-10-12. Local app: http://localhost:3000. Source implementation commits: 1f3342f and dd196b5.

Operator reporting, affiliate reconciliation, creator claims and links, media kits and the outreach helper are implemented. Billing was implemented at the previous checkpoint and has been regression-tested again. Phase 6's live external acceptance remains open: Stripe and Impact credentials are empty, and no real creator has supplied a profile-bio challenge. No production launch is claimed.

## What works

- `/ops` is restricted to verified sign-ins on the explicit `OPS_EMAILS` allowlist. It shows household activity, extraction results and recorded spend, correction counts, carts and clicks, reported orders, trials, current paid households, cooking feedback, creator traffic, currency-separated daily revenue and CSV export. Unconfigured integrations and unpriced model calls are labelled. Counters begin with this deployment, not invented historical measurements.
- Impact reconciliation runs at 02:15 UTC and can be queued from `/ops`. It reads changed action IDs, fetches their current totals, matches `SubId1` to a known cart and upserts the action. Replays do not add commission twice; reversals zero the commission. Failures remain visible. No grocer basket contents or customer location from reports are stored.
- `/creator` provides a 48-hour profile-bio challenge. Only the matching profile's structured biography can prove ownership. A code in a comment does not count. Blocked public access leaves the claim unverified. No login or proxy is used.
- Verified creators receive `/c/{platform}-{handle}` and per-video links. Hourly discovery queues up to 20 public videos; creators can also submit a video link. The extracted owner URL must match the claimed profile before publication. Failed or unverifiable sources are labelled.
- Creators review and pin corrections through the normal recipe editor. The database rejects a pin with low-confidence fields. Later extraction and audio results cannot replace a pinned recipe. Later pastes of the same video use the pinned version. Household copies stay private.
- Public creator recipes support anonymous shopping and signed-in saving/cooking. Creator statistics show views, carts and cook starts without commission amounts. Raw captions/transcripts and provider usage records are stripped from public creator recipe payloads.
- Media kits download as transparent 1080 by 1920 PNGs in the creator's chosen accent, with a copyable two-line caption. Text colour adapts to the accent for contrast.
- `/ops/creators` accepts up to 20 handles, pre-extracts their latest public video and prepares a claim link and message. A message says a recipe is ready only when a draft exists. The operator sends outreach by hand. Optional payout share defaults to zero; saving a percentage does not send money.
- Anonymous shopping links expire after seven days. Their list contents and outbound cart links are erased. Opaque references remain for another 180 days to match delayed affiliate reports; references with recorded orders remain for reporting. Household deletion still cascades immediately.

## Verification

| Check | Result | Scope |
| --- | --- | --- |
| Type checking | Passed | Shared, grocery, worker, web and test code |
| Unit suite | 266 passed | Includes profile identity, anti-impersonation, Impact pagination/totals and JSON-order regression |
| Measured core line coverage | 98.55% | Configured core modules, not a whole-application coverage claim |
| Real Supabase household checks | Passed | RLS, corrections, immutable household shopping snapshots, per-device access and deletion |
| Creator database integration | Passed | Injected public bio proof, missing-code rejection, owner conflict, private claims, pin guard, version conflict, public payload filtering, RLS on eight new tables and deletion |
| Affiliate database integration | Passed | Decimal precision, replay, household attribution, approval/reversal, unknown cart rejection, expired anonymous content erasure with retained attribution and deletion cascade |
| Billing database integration | Passed | Injected Stripe transport, one trial, concurrent replay, invoice ledger, cancellation grace, full refund and old-event protection |
| Final full browser suite | 60 passed, 36 skipped, 2.9 minutes | Intentional duplicate-device skips; real local magic-link sign-in, both conductor scenarios, helpers, billing, persistence, creator and operator flows |
| Creator browser acceptance | Passed locally | Claim UI, correction and pin, refresh, pinned recipe on a later paste, public link, anonymous shopping, 1080 by 1920 PNG and denied operator access |
| Operator browser acceptance | Passed locally | Explicit test-only operator email, dashboard, CSV, anonymous denial and outreach screen |
| Live public YouTube profile read | Passed, 717 ms | foodwishes biography present and 20 public videos discovered; no ownership claim or video extraction asserted |
| Production builds | Passed | Next.js, worker, web Docker image and worker Docker image |
| Local deployment | Healthy | Web, worker and Redis; migrations 0012 and 0013 applied |
| Deployed browser inspection | Passed | Creator console loaded at localhost:3000 in the existing signed-in browser, no browser errors |
| Secrets handling | Passed | .env remains ignored with mode 0600; no secrets copied into Git or the public report |

The creator database test injects profile evidence, the billing test injects Stripe responses, and the browser tests use real local Supabase magic-link sign-in with a labelled disposable creator profile. None is presented as a live creator claim or Stripe purchase.

### Conductor regression at 60x

| Scenario | Completed steps | Finish spread | Maximum distance from serve time | Serve-time shift | Read p50 | Read p95 | Oven conflicts |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| on time | 7 | 0.375 min | 0.375 min | 0.647 min | 40 ms | 53 ms | 0 |
| 12-minute delay | 7 | 0.428 min | 0.428 min | 12.826 min | 33 ms | 69 ms | 0 |

Both three-dish scenarios, with dessert baked earlier, passed the five-minute serving window. These are local measurements, not a Lighthouse or mobile-network certification.

## Live integration blockers

| Check | Result | Exact reason |
| --- | --- | --- |
| Stripe hosted test-mode purchase and refund | Not run | STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET, STRIPE_PRICE_MONTHLY and STRIPE_PRICE_YEARLY are empty. Preflight stopped before any Stripe API call or purchase. |
| Live Impact reconciliation | Not run | IMPACT_ACCOUNT_SID and IMPACT_AUTH_TOKEN are empty. |
| Real creator ownership claim | Not run | A real creator must put the generated challenge in their own public biography. No third-party handle was claimed during testing. |
| Default operator access on localhost | Denied until configured | OPS_EMAILS is empty. The operator browser test uses an explicit test-only allowlist. |
| Public production deployment | Not performed | This checkpoint is a local implementation and verification checkpoint. |

## Failures found and resolved

- Postgres JSONB key ordering made identical review fields look changed. Comparisons now ignore object-key order, with a regression test.
- The first media-kit browser test exposed an invalid multi-child text block in the PNG renderer. It was corrected; dimensions are checked by the browser test.
- The first Docker web build lacked worker source files needed by the new database-check scripts during type checking. The build context now includes those files.
- One full browser run passed 59 checks and failed one security case because the new creator-paste test had consumed the shared test IP's anonymous allowance. The app correctly returned the human-check response. That test now starts with a fresh counter in the isolated test Redis; production limits were not relaxed.
- An initial retention test correctly hit the immutable shopping-snapshot guard. A separate migration permits only privileged erasure of expired anonymous contents. Household snapshots remain immutable, and delayed attribution is tested after erasure.
- An overlapping intermediate local build was superseded. Only the final successful build and completed test run count as verification.

## How to test

With local Supabase and test Redis running:

```sh
pnpm typecheck
pnpm test:coverage
pnpm test:database
pnpm exec tsx apps/web/scripts/check-creators.ts
pnpm exec tsx apps/web/scripts/check-affiliates.ts
pnpm exec tsx apps/web/scripts/check-billing.ts
pnpm build
pnpm exec playwright test
pnpm exec tsx apps/worker/scripts/profile-probe.ts
node scripts/phase6-preflight.mjs
```

Read CREATORS.md for the operator/creator setup and BILLING.md for hosted Stripe purchase and full-refund acceptance. Migrations 0012 and 0013 must run before deploying the new images. Set OPS_EMAILS to the actual operator's verified sign-in email. Add provider secrets only to .env, never Git.

## Known limits and next checkpoint

Public profile and video metadata depend on platform availability; login walls and robots restrictions are not bypassed. A platform may expose fewer than 20 recent videos. The ownership check intentionally refuses to publish when extracted creator attribution cannot be verified. Initial Impact import covers 30 days; later imports catch up from the last successful run. Commission reports retain currency and pending/approved/reversed status, not invented order completion. Optional creator payout share has no automatic payout execution.

The next Phase 6 acceptance step is a real Stripe test-mode hosted purchase and full refund after the four Stripe settings are supplied, then a configured Impact report and a real creator bio challenge. Phase 7 has not started. Historical extraction, grocery, helper and voice limitations remain in the preserved checkpoints below.


---

# Preserved historical checkpoints

The earlier checkpoint content and result tables below are historical. The current result is above.

# Phase 6 continuation checkpoint

2026-10-12. Published immediately at the operator’s request before continuing. The verified timeline repair and billing increment remain deployed locally. The operator dashboard, affiliate reconciliation, creator claim and links, media kit and outreach helper are the active work. Live Stripe purchase and refund acceptance remains unverified; configuration will be checked during this continuation. No new test result is claimed in this immediate checkpoint.

## Latest full checkpoint and historical results, unchanged

# Second-screen repair and Phase 6 billing checkpoint

2026-10-12. The requested second-screen repair is complete and deployed at http://localhost:3000. Household trial and billing code is also deployed locally. Phase 6 is in progress, not complete. There is no public production deployment and no verified live Stripe purchase.

Source commits: 32aacca for the timeline repair, a3879c1 for household trials and billing.

## Second-screen repair

The timeline uses two separate columns: step text on the left, bars on the right. The time axis is above the bars and shares their column boundaries. Bars and the current-time marker are contained within the bar column. Long instructions wrap within the text column. Narrow screens can scroll the schedule horizontally.

Equipment identifiers are shown as Oven, Hob 2 and Air fryer. The duplicated default serve-time sentence is gone; meaningful delay messages remain. The deployed real saved meal was reopened in the browser and inspected after the update.

The visual fixture includes a long instruction and multiple equipment names. It checks both screenshots and geometry: no bar may enter the text column, and the axis must align above the tracks. Phone/tablet and light/dark baselines are committed for macOS and Linux.

## Billing continuation

- Saving the second dish starts one 14-day trial for the household. Planning the seeded multi-dish example also starts it. No card or Stripe subscription is created for the free trial.
- Protected Postgres fields and a database function enforce trial expiry and prevent restarting it. Multi-dish planning, new helper invitations and voice require active access. Shopping and single-dish cooking remain free. Existing cooking sessions can finish.
- The meal screen offers one Premium plan with monthly and yearly choices. Checkout and the customer portal are restricted to the household owner. Stripe Checkout requests automatic tax and provider-managed currency presentation.
- Raw-body signed webhooks retrieve current Stripe objects. Event IDs and entitlement changes are stored in the same transaction. Replay does not duplicate the grant or invoice. Cancellation preserves the paid period; a full refund of the latest invoice cancels that subscription and revokes access. An old event cannot restore it.
- Household deletion cancels billing and expires an unfinished checkout first. Payment records are included in the household export and cascade on deletion. A Stripe outage prevents deletion from falsely succeeding while a subscription remains active.
- Plan and helper checks reuse their existing database transaction. Six simultaneous planning requests pass without exhausting the connection pool.
- Migrations 0009 through 0011 are applied locally. The web container has been rebuilt and is healthy; the existing worker and Redis are healthy.

The billing setup and exact behavior are documented in BILLING.md. A pending checkout retains its selected interval until its 30 to 60 minute expiry. This is a known usability limitation, not a second plan.

## Verification

| Check | Result | Scope |
| --- | --- | --- |
| TypeScript | Passed | Web, worker, packages and tests |
| Unit suite | 252 passed | Includes four new signed-webhook route tests |
| Measured line coverage | 98.48% | Existing configured core modules; not a whole-application or billing coverage claim |
| Signed webhook rejection | Passed | Missing, tampered and stale signatures, wrong mode, oversized body; provider/database error returns a retryable response |
| Real Supabase household checks | Passed | Isolation, confirmation, correction audit, versioning, shopping snapshots, helper/push ownership and cascading deletion |
| Billing database integration | Passed | Owner isolation, protected fields, one trial, expiry, concurrent replay, invoice ledger, cancellation grace, refund and old-event safety |
| Browser trial/persistence test | Passed | Real local email sign-in, save second dish, refresh, six simultaneous plans, expire trial, retain shopping |
| Final full browser suite | 58 passed, 30 skipped, 2.9 minutes | Skips are intentional duplicate device scenarios |
| Two-browser helper scenario | Passed | Anonymous helper, two-minute Ack fallback, voice Next through the real API, Done and read-only timeline |
| Linux visual regression | 12 passed, 6.4 seconds | 20 snapshots: Meal, Plan, Cook, Helper and second screen in four device/theme combinations |
| Production builds | Passed | Next.js, worker and latest web Docker image |
| Deployed browser check | Passed | Real saved meal at localhost:3000, separate Gantt columns, plain equipment names, one serve-time sentence |
| Live Stripe purchase and refund | Not run | STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET, STRIPE_PRICE_MONTHLY and STRIPE_PRICE_YEARLY are empty |
| New live extraction or grocery order | Not run | Historical results below are unchanged |

The billing database integration injects a Stripe transport. It does not contact Stripe or prove that the operator's account is configured. The browser tests use real local Supabase email links and the local Mailpit inbox; recipe extraction contracts use the isolated test model. No new paid model call is claimed.

## Three-dish conductor regression at 60x

| Scenario | Completed steps | Dish finish spread | Maximum distance from serve time | Serve-time shift | Read p50 | Read p95 | Oven conflicts |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| On time | 7 | 0.347 min | 0.347 min | 0.636 min | 40 ms | 68 ms | 0 |
| Injected 12-minute delay | 7 | 0.355 min | 0.355 min | 12.877 min | 39 ms | 56 ms | 0 |

Both scenarios passed the five-minute serving window. They include three hot dishes and a dessert baked earlier. These are local measurements, not a mobile-network or Lighthouse certification.

## Failures resolved during this work

The first billing browser setup sent extra fields to the strict meal API and was correctly rejected. The test now sends the supported fields. An initial full run passed 54 checks and failed four expected meal screenshots because the trial panel had been added. Those layouts were inspected, the baselines updated on macOS and Linux, and the final full suite passed. An intermediate suite was stopped when its build was superseded; it is not counted as a completed run.

Code review also found a nested-transaction connection risk in the new premium checks. The checks now reuse the caller's transaction, and the six-request browser regression passes.

## How to test

With local Supabase and the test Redis running:

```sh
pnpm db:local
pnpm typecheck
pnpm test:coverage
pnpm test:database
pnpm exec tsx apps/web/scripts/check-billing.ts
pnpm build
docker compose -f infra/compose.test.yml up -d --wait
pnpm test:e2e
```

For the targeted visual repair:

```sh
E2E_EXTERNAL=1 E2E_BASE_URL=http://localhost:3000 pnpm exec playwright test tests/e2e/helper-visual.spec.ts tests/e2e/conductor-visual.spec.ts
```

Open a saved meal, Plan the night, then the live second screen. Steps stay on the left and bars stay on the right. On a narrow screen, swipe within the schedule to see its full width.

## Incomplete work and next steps

This is an intermediate checkpoint, not Phase 6 acceptance. Live payment verification cannot run until the four Stripe settings are configured. The operator dashboard, affiliate reconciliation, creator claiming and public links, media kit and outreach helper remain unimplemented in Phase 6. They are not represented as working or as blocked solely by Stripe. No Phase 7 work has started.

Physical-phone push and microphone acceptance, the independent extraction-reference review, Kroger customer OAuth/cart write, affiliate approvals, public deployment and fresh-VM verification remain open as previously reported. No attributed grocery order or live paid subscription is claimed.

Next: finish the remaining Phase 6 creator/reporting features and run the real Stripe test-mode purchase/refund acceptance once configured. The status channel contains only this report and historical checkpoint Markdown, never application code or secrets.

## Historical checkpoints, unchanged

# Second-screen repair checkpoint

2026-10-12. The repair is deployed at http://localhost:3000. This is a local site, not a public production deployment. Phase 6 is now in progress and is not declared complete.

## What changed

The Gantt has separate text and bar columns. The time axis sits above the bar column, aligned with its boundaries. Bars and the current-time marker are clipped inside that column, so they cannot cover words. Narrow screens can scroll the schedule horizontally. Equipment names read Oven, Hob 2 and Air fryer instead of internal identifiers. The duplicated default serve-time sentence is removed; meaningful delay messages remain.

## Verification

| Check | Result | Scope |
| --- | --- | --- |
| Production web build | Passed | Next.js and Docker image |
| macOS visual regression | 8 passed | Plan, Cook, Helper and second screen, phone/tablet, light/dark |
| Linux baseline run | 8 passed | Same screens against the deployed container |
| Final macOS regression without baseline updates | 8 passed in 3.7 seconds | Saved screenshots match |
| Geometry assertions | Passed | Bars cannot cross into labels; time axis aligns above tracks |
| Live browser review | Passed | Refreshed the real saved meal's second screen, friendly labels and one serve-time sentence |

Long step text and multiple equipment names are included in the regression fixture. The new screenshot baselines are committed with the fix. No extraction, payment or grocery-order result is inferred from these visual checks.

## How to test

```sh
pnpm --filter @cookthis/web build
pnpm exec playwright test tests/e2e/helper-visual.spec.ts tests/e2e/conductor-visual.spec.ts
```

Open a saved meal, Plan the night, then the live second screen. Read the steps on the left and the bars on the right. On a narrow phone, swipe horizontally within the schedule.

## Continuing

Phase 6 implementation is in progress: household premium, Stripe billing, operator reporting and creators. Stripe and Impact credentials are empty, so live purchase/refund and affiliate reporting acceptance remain blocked by configuration. No Phase 6 completion or live payment is claimed. All earlier limitations remain as recorded below.

## Historical checkpoint, unchanged

# Phase 5 checkpoint: helpers and voice

2026-10-11. Phase 5 is implemented and running at http://localhost:3000. Automated checks and real MiMo calls passed. Physical-phone push delivery and microphone recognition are still unverified. The public production deployment is not complete. Stop here before Phase 6.

## What works

- The cook creates a private invitation for each helper. A helper joins with a display name only, with no email or account. Invitations expire after seven days. Only the access hash is stored; the browser exchanges the URL fragment for an HttpOnly, same-site cookie scoped to that helper's API.
- The cook assigns a saved recipe step. The server makes it available when its scheduled start and prerequisites are reached. Ack moves that task to the helper; Done updates the same stored cooking session, replans through the engine, and appears on the cook's screen and second screen.
- Unanswered tasks return to the cook after two minutes. Browser reads and background workers use the same transition rule. With every tab closed, the worker reconciles every five seconds and queues notifications in BullMQ.
- Per-device helper subscriptions support more than one phone or browser. Only supported push hosts are accepted. Expired endpoints are removed. Household deletion removes invitations and subscriptions; exports include helper records without capability hashes.
- Reopening a plan with invitations preserves that plan, preventing ordinary navigation from invalidating links. The plan is a saved recipe snapshot once invitations exist.
- Hands-free mode recognizes Next, Repeat and Behind. Next and Behind use versioned server actions. Behind adds twelve minutes. Questions cannot mutate timers or tasks.
- Spoken questions use the configured MiMo model with the saved plan, recipes and explicit server-computed minutes remaining. Numeric food safety information comes from a fixed FoodSafety.gov note. Recipe oven settings remain separate from internal doneness temperatures.
- Tap to talk records at most ten seconds and transcribes with the configured MiMo audio model. Recordings stay in memory only. Browser speech synthesis is primary, with verified mimo-v2.5-tts synthesis as fallback. All controls remain available by tap and keyboard.
- The second screen is a signed-in, read-only Gantt timeline. It polls the server every second and displays the server time, assignments, equipment and completion state.
- The updated web and worker images were built and deployed locally. Migration 0008 is applied. The in-app browser displays the deployed timeline for a saved three-dish Cookthis test meal.

## Verification

| Check                             | Result                | Scope                                                                                                                             |
| --------------------------------- | --------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| TypeScript                        | Passed                | Web, worker, shared code, grocery adapters and tests                                                                              |
| Unit tests                        | 248 passed            | Includes helper transitions, pinned assignments, exact voice commands and MiMo request contracts                                  |
| Measured line coverage            | 98.48%                | Configured core modules, not a whole-application coverage claim                                                                   |
| Helper state/commands coverage    | 100%                  | Lines and branches in the pure shared team module                                                                                 |
| Real Supabase checks              | Passed                | Household isolation, direct-write denial, derived helper ownership and cascading deletion                                         |
| Full browser suite                | 57 passed, 27 skipped | Skips are intentional duplicates across device projects                                                                           |
| Two-browser helper scenario       | Passed                | Real local Supabase email sign-in for the cook; separate anonymous phone-sized context for the helper                             |
| Invitation persistence regression | Passed                | Reopen Plan after creating an invitation, retain the same session and working link                                                |
| Helper Ack fallback               | Passed                | Two-minute server deadline, stale Ack rejected, task returned to cook                                                             |
| Voice Next                        | Passed                | Controlled browser recognition event through the real authenticated endpoint and database                                         |
| Helper Done and second screen     | Passed                | Completion survives refresh and appears on the read-only timeline                                                                 |
| Push worker integration           | Passed                | Real Postgres and BullMQ, two device subscriptions, injected push transport, stale suppression and no duplicate on reconciliation |
| No-tab fallback                   | Passed                | Worker transitions an overdue unacknowledged task with no browser polling                                                         |
| Production builds                 | Passed                | Next.js, worker and both Docker images                                                                                            |
| macOS visual checks               | Passed                | Plan, Cook, Helper and second screen, phone/tablet, light/dark                                                                    |
| Linux visual checks               | Passed                | Same four screens against deployed containers, 8 checks covering 16 snapshots                                                     |
| Physical-phone push               | Not run               | No claim that a notification reached a real phone                                                                                 |
| Physical microphone recognition   | Not run               | Browser event contract and live MiMo audio were tested separately                                                                 |

The full browser suite ran in 2.9 minutes. A final focused two-browser rerun after the invitation-persistence fix passed in 10.1 seconds. Tests use the isolated localhost:3100 server; production has no client-controlled clock override.

## Three-dish conductor regression at 60x

| Scenario                 | Completed steps | Dish finish spread | Maximum distance from serve time | Serve time shift | Read p50 | Read p95 | Oven conflicts |
| ------------------------ | --------------: | -----------------: | -------------------------------: | ---------------: | -------: | -------: | -------------: |
| On time                  |               7 |          0.374 min |                        0.374 min |        0.792 min |    36 ms |    78 ms |              0 |
| Injected 12-minute delay |               7 |          0.389 min |                        0.389 min |       12.750 min |    36 ms |    66 ms |              0 |

Both runs include three hot dishes and a dessert baked earlier. They passed the five-minute serving window. These are local server measurements; they do not certify mobile-network latency or Lighthouse scores.

## Live MiMo verification

Measured at 2026-10-11 18:52 UTC. `/v1/models` returned HTTP 200 and confirmed this account supports mimo-v2.6-pro and mimo-v2.5-tts. Text and audio use the configured mimo-v2.6-pro. The audio test transcribed synthetic speech made for this check, not a person's microphone recording.

| Call                | Outcome                                     |    Time | Input tokens | Output tokens | Estimated USD |
| ------------------- | ------------------------------------------- | ------: | -----------: | ------------: | ------------: |
| Cooking question    | Correctly answered 6 minutes remaining      | 3.206 s |          246 |            42 |    0.00014355 |
| Speech synthesis    | Returned 99,884 bytes of WAV audio          | 3.084 s |          158 |            15 |      Unpriced |
| Audio transcription | Returned “Repeat the current cooking step.” | 3.785 s |           48 |            14 |    0.00003306 |

Estimates use the configured rates, without cache discounts, and are not billing receipts. TTS rates are blank, so no zero-cost claim is made. TTS token counts are retained. No new twenty-video extraction benchmark or grocery order was run in this phase.

## Fixes made during verification

- The app's existing Permissions-Policy disabled microphones. It now permits microphone access on the same origin, with explicit browser permission.
- The first live model answer treated an epoch value as unclear timing. The route now supplies authoritative minutes remaining and an ISO server time. The subsequent live answer correctly reported six minutes.
- An unstarted plan previously regenerated when reopened. Plans with invitations now preserve the session so links survive; a browser regression covers it.
- The second screen initially repeated the Gantt as a full list. It now shows one timeline with a server-time marker and readable step labels.
- Voice recordings have a server-checked size and PCM format, same-origin protection and a per-household request limit. Priced voice calls share the daily model budget; missing usage or rates are counted explicitly.

## How to test

With dependencies installed, local Supabase running and test Redis available on port 6382:

```sh
pnpm db:local
pnpm typecheck
pnpm test:coverage
pnpm test:database
pnpm build
docker compose -f infra/compose.test.yml up -d --wait
pnpm exec tsx apps/worker/scripts/check-cook-timers.ts
pnpm exec tsx apps/worker/scripts/check-helper-tasks.ts
pnpm test:e2e
pnpm exec playwright test tests/e2e/helpers-voice.spec.ts --project=phone-light
pnpm exec tsx --env-file=.env scripts/check-voice-live.ts
```

Run the worker integration checks with no active local cook. They inject the push sender and do not contact a phone. The final command makes paid API calls and writes an ignored local report.

For the UI, sign in through the normal email link, open a saved meal and Plan the night. Create a helper invitation and open it in a separate browser profile. Enter a display name, then assign a step from the cook's screen. Start the cook when ready. Ack and Done work from the helper page. Skip Ack on another task to check the two-minute fallback. Open live second screen on a tablet signed into the same household. Enable hands-free voice on the cook's screen, or use Tap to talk or the typed command field.

## Known limitations and remaining launch checks

- Two browser contexts are not two physical phones. Real OS push delivery, background restrictions and noisy-kitchen speech recognition still need a device acceptance run on HTTPS. iOS push needs an installed home-screen web app, completed in Phase 7.
- Push is best effort. An ambiguous retry may duplicate a notification; stable tags replace previous cards where supported. The plan and fallback never depend on successful push delivery.
- Helpers are session-scoped guests. They are not permanent email household accounts. Keep each invitation private; it remains valid for seven days.
- Delegation is explicit. Cookthis does not silently assign a hot-oven task to a household member. Future unassigned tasks retain their conservative cook reservation; acknowledged tasks can free the cook's hands when the engine replans.
- Recipe edits made after inviting helpers do not rewrite that plan. The current saved plan is authoritative for the cooking session.
- Browser speech recognition may use the browser vendor's service. Tap to talk uses MiMo. The interface discloses the distinction and retains typing and buttons.
- TTS pricing is unverified. Fill the configuration from the operator's MiMo console; unpriced calls are counted and are never reported as free.
- Phase 1 independent extraction-reference review remains open. The historical reliability results are unchanged.
- Kroger catalog matching previously reached 12/12, including four package guesses. Customer OAuth/cart write remains blocked by the public callback versus localhost origin. Instacart approval and affiliate configuration are still required. No attributed order is claimed.
- Public deployment, fresh-VM verification, PWA/share-target completion and physical mobile acceptance remain Phase 7.

## Next

Stop at this checkpoint. On “continue”, Phase 6 adds household premium and the card-free trial, Stripe Checkout and signed webhooks, operator reporting, affiliate reconciliation, creator claiming, media kits and the outreach helper. No Phase 6 implementation was started.


---

# Historical checkpoints and result tables

The following reports are retained unchanged as historical evidence. They are not new extraction, grocery or deployment measurements. The Phase 5 report above is the current status.

# Cookthis status

The current checkpoint is followed by earlier reports, preserved as history. This public repository contains only checkpoint Markdown, never application code, credentials or private household data.

# Phase 4 checkpoint, 2026-10-11

The Kroger matcher update is deployed locally and the conductor acceptance scenarios pass. The app is running at http://localhost:3000. No public production deployment or live shopper checkout is claimed. This checkpoint stops before Phase 5.

## Kroger package matching

The latest certification catalog run matched 12 of 12 ingredients, up from 8 of 12. Eight use compatible package conversions. Four are explicit package guesses: apples, cinnamon, paprika and salt. Apples use a 250 g per-fruit estimate to select a labelled bag; small spice amounts use one standard package. The shopper sees the proposed product, package size, package count and reason under Check these package guesses.

Exact conversions take priority. Loose produce sold by weight, unavailable stock, unknown package sizes, unrelated food products, oversized amounts and inappropriate spice forms remain rejected. Only UPC and package count go to Kroger; guess metadata stays in the Cookthis result.

| Ingredient  | Amount | Lookup   | Time    | Match               | Packages |
| ----------- | ------ | -------- | ------- | ------------------- | -------- |
| apples      | 4 each | HTTP 200 | 1034 ms | Guessed package     | 1        |
| green beans | 400 g  | HTTP 200 | 1201 ms | Convertible package | 1        |
| lemon       | 1 each | HTTP 200 | 1705 ms | Convertible package | 1        |
| potato      | 800 g  | HTTP 200 | 2370 ms | Convertible package | 2        |
| butter      | 15 g   | HTTP 200 | 1608 ms | Convertible package | 1        |
| brown sugar | 14 g   | HTTP 200 | 1020 ms | Convertible package | 1        |
| oats        | 12 g   | HTTP 200 | 1358 ms | Convertible package | 1        |
| olive oil   | 41 g   | HTTP 200 | 1755 ms | Convertible package | 1        |
| water       | 100 g  | HTTP 200 | 2076 ms | Convertible package | 1        |
| cinnamon    | 5 ml   | HTTP 200 | 1005 ms | Guessed package     | 1        |
| paprika     | 5 ml   | HTTP 200 | 1100 ms | Guessed package     | 1        |
| salt        | 5 ml   | HTTP 200 | 1097 ms | Guessed package     | 1        |

All product requests returned HTTP 200. Client token: HTTP 200, 372 ms. Store lookup: HTTP 200, 249 ms. The empty-items cart authorization probe returned HTTP 403, 257 ms. No cart items were added. These are catalog matches, not customer-confirmed cart contents. Model and speech cost: $0.

The registered callback is still on cookthis.app while this app runs on localhost. Customer OAuth and a reachable matching callback remain necessary for a real cart write. Instacart and other affiliate acceptance gates remain unchanged.

## What works in the conductor

Saved meals have Plan the night. The server takes a snapshot of recipes, scales the mise en place amounts, checks household equipment and stores a timeline in Postgres. Before cooking, opening Plan the night regenerates the unstarted plan from the saved meal. Once cooking starts, the snapshot stays stable.

The scheduler respects sequential dependencies, active hands, passive appliance use, hob pools, air fryers and slow cookers. Same-temperature dishes can share an oven. Different oven temperatures get a ten-minute changeover. Recipe resting steps stay in the dependency chain. The cook checks that trays physically fit.

The Plan screen has an ingredient/tool checklist, start countdown and whole-meal schedule. Starting early shifts the remaining plan. The Cook screen has one large current step, a timer, 72 px Done and Behind buttons, next-step context, other ready dishes and wake lock support. Behind adds twelve minutes. The server replans unfinished work and explains a serve-time change.

Sessions survive refresh, closed tabs and a new signed-in device. Version checks reject stale actions from another screen. The browser renders the server plan and does not generate a schedule. Once loaded, its shell and last snapshot can reopen offline. Offline actions are disabled, the clock snapshot is labelled stale, and the complete saved timeline is visible until reconnection.

Per-device Web Push subscriptions and BullMQ timer jobs are implemented. Timer jobs recheck the stored step end before sending. The worker recovers reminders up to five minutes overdue, suppresses obsolete reminders, and removes expired device endpoints. A notification opens the saved cooking session. Local VAPID keys were generated into ignored .env only. Live phone delivery has not been verified.

End-of-cook feedback is stored with the session and linked back to recipe corrections. Sessions, device subscriptions and notification records have household ownership and row-level protection and are removed by household deletion. Direct authenticated database writes cannot change conductor plans.

## Acceptance measurements

The browser signs in with real local Supabase magic-link email. The scheduling fixtures are explicitly labelled Cookthis tests. They contain roast chicken, potatoes and greens, plus a dessert baked earlier at a different temperature. The browser uses the real web API and database with a server-only 60x clock on the isolated test origin. The test completes seven steps, injects the delay in the late scenario, checks every pair of blocks for conflicts, reloads offline and reconnects, then records feedback.

| Scenario        | Speed | Steps completed | Dish finish spread | Maximum distance from serve time | Serve shift | Read p50 / p95 | Oven conflicts |
| --------------- | ----- | --------------- | ------------------ | -------------------------------- | ----------- | -------------- | -------------- |
| on time         | 60x   | 7               | 0.415 min          | 0.415 min                        | 0.750 min   | 44 / 59 ms     | 0              |
| 12-minute delay | 60x   | 7               | 0.402 min          | 0.402 min                        | 12.801 min  | 44 / 59 ms     | 0              |

Each hot dish finished within five minutes of the stored serve time. Small serve shifts in the on-time run include real request and polling latency magnified by the 60x clock. The delayed run preserves the recipe's cooking and resting durations. Read latency is local API latency, not a Lighthouse result or an end-to-end phone rendering benchmark. No model calls are needed to run or replan this fixture.

| Check                        | Result                                                                                                                    |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Unit suite                   | 241 passed                                                                                                                |
| Line coverage                | 98.45% overall, 98.20% timeline, 100% shopping merge, 99.22% grocery adapters                                             |
| Type checks                  | Passed across packages and test code                                                                                      |
| Production builds            | Web and worker passed, including Docker images                                                                            |
| Database integration         | RLS, conductor write protection, device isolation and cascading deletion passed                                           |
| Full browser suite           | 52 passed, 24 intentional duplicate-scenario skips                                                                        |
| Conductor acceptance         | Both 60x scenarios passed, including the 12-minute delay and offline reload                                               |
| Timer integration            | Real Postgres and BullMQ, injected push transport; overdue recovery, stale-job suppression and duplicate avoidance passed |
| Visual baselines             | Plan and Cook on phone/tablet in light/dark, macOS and Linux; meal baselines updated for Plan the night                   |
| Local deployment             | New web and worker images running; web, worker and Redis healthy                                                          |
| Live Kroger catalog matching | 12/12 accepted, including four labelled package guesses; customer cart write still blocked                                |

The deployed localhost:3000 meal-save regression passed separately with actual local email sign-in, refresh and reopening (1 test, 2.5 seconds). Linux visual verification against the deployed containers passed all eight checks (4.0 seconds). The real signed-in in-app browser also opened the saved test dinner and generated its plan.

An earlier browser run failed on an ambiguous test selector that also matched Next.js's route announcer. The selector was corrected. A subsequent test-report write failed because simultaneous visual and full runs shared an output directory. The final full suite was rerun separately and passed; future separate checks use distinct output folders. These were not hidden or counted as successful full runs.

## Reproduce

```sh
pnpm typecheck
pnpm test:coverage
pnpm test:database
pnpm build
pnpm exec tsx apps/worker/scripts/check-cook-timers.ts
pnpm test:e2e
pnpm exec tsx --env-file=.env scripts/kroger-live.ts
```

Local Supabase and the test Redis on port 6382 must be running for integration checks. Run the timer integration with no active local cook session; it injects the notification sender and sends no external message. The full browser configuration enables 60x only on localhost:3100. Normal localhost:3000 and production use real time.

To try the app, open a saved meal at http://localhost:3000, choose Plan the night, check the mise en place list and start cooking. Household equipment is editable in Account. The existing saved-meal race regression is retained.

## Limits and next phase

- Kroger customer OAuth, callback reachability and a real shopper cart write remain blocked. A 12/12 catalog result is not a verified order. Other grocers still need their approved credentials or affiliate links.
- Real phone push delivery is unverified. Web Push is best effort, and a closed tab does not guarantee the operating system will show a reminder immediately.
- Oven temperature sharing assumes the cook checks tray capacity. There is no automatic tray-size model.
- The engine exposes prep-ahead output for explicitly marked steps. Existing recipes without this metadata show guidance to consult the recipe; automatic generation of day-before cooking/storage instructions is not implemented. Unverified warm-holding advice is not invented.
- Offline mode displays the last saved plan and clock snapshot. It does not accept offline completions or invent a revised schedule.
- Helpers, task acknowledgement, voice and the read-only Gantt second screen are Phase 5. Billing and creators are Phase 6. Full PWA sharing, lifecycle email, legal pages and fresh-VM launch acceptance are Phase 7.
- Phase 1's independent ingredient/quantity accuracy acceptance and prior live partner limitations remain as recorded in earlier checkpoints. No new 20-video extraction run was performed here.

Next: Phase 5, helper invitations, per-person tasks, acknowledgement fallback, voice controls and the live second screen. Wait for the operator's continue instruction.


---

# Historical checkpoint, published 2026-10-11 at 18:10 UTC

The material below is historical. Its references to Phase 4 not having started and to eight Kroger matches have been superseded by the current checkpoint above. Earlier extraction tables are preserved; they were not rerun for this update.

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

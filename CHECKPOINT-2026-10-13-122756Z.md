# Cookthis checkpoint: clearer navigation, appearance choice and display-only Kroger logo

13 October 2026. Application source commit c787054.

## What changed

Your kitchen is now a clearly labelled, larger button beside the Cookthis logo on larger screens. On phones it has its own full-width second header row, with at least a 48-pixel touch target. It no longer sits as a tiny uppercase label at the far edge.

The header now offers System, Light and Dark appearance choices. The selection is saved on the current device, applied before the page paints and retained across navigation and reloads. System follows the device preference. No account is required, and blocked local storage does not prevent changing the current tab's appearance.

The Kroger logo is display-only. It has no link, click action or keyboard stop. Try a recipe remains a separate link below the shopping explanation.

## Verification

| Check | Result |
| --- | --- |
| Local optimized web build and type validation | Passed |
| macOS browser checks | 36 passed |
| Linux browser checks | 36 passed |
| Appearance persistence | Light and Dark retained after reload and navigation |
| System override | Explicit Light overrides a dark device; System follows subsequent device changes |
| Kitchen navigation | Correct /meals destination; touch target at least 48 pixels |
| Kroger logo | Image remains visible; wrapper is a non-interactive div with no link |
| Visual baselines | Refreshed for affected home, recipe, meal, shop, plan, cook, second-screen and install surfaces, phone/tablet and light/dark, on macOS and Linux |
| Live browser verification | Larger kitchen button, Dark retained after reload, Light rendered correctly, Kroger logo non-clickable; preference restored to System after checking |
| Production deployment | Completed; web, worker, Caddy, Redis and backup healthy |
| Public HTTPS health | /healthz returned status ok, modelConfigured true |
| Source commit | c787054 |

## How to use it

Open https://cookthis.app. Use Your kitchen beside the logo, or the wide button beneath the logo on a phone. Choose Light, Dark or System from the appearance menu in the header. The change is remembered on that browser.

## Scope and limitations

This is a navigation and appearance update. Production credentials and grocery/billing configuration were not changed. The existing grocery OAuth/cart acceptance and customer billing-portal limitations remain recorded below. The Kroger logo is no longer a link, superseding the prior checkpoint's link description.

Rollback image: cookthis-web:before-header-theme. Verification used the Next.js production build and Playwright's home, recipe, meal, shopping, conductor, helper/second-screen, launch and appearance cases. No purchases or cart mutations were made.

## Historical checkpoints and measured results

The following is preserved history. Current navigation, appearance and logo behavior above supersede earlier descriptions.

# Cookthis checkpoint: Kroger production and home-page partner logo

13 October 2026. Application source commit aa3e157.

## Completed

The supplied Kroger production client credentials were written only to the private production environment file, with permissions 0600. KROGER_ENVIRONMENT is production. The callback is https://cookthis.app/api/grocery/kroger/callback. Both web and worker were recreated to load the settings. Local development environment files were not changed, and credentials were not committed.

The home page now displays the Kroger logo below the paste introduction, labelled Grocery partner · US. It says to choose Kroger when shopping a recipe and explains that prices and checkout are Kroger's. The logo links to Kroger; Try a recipe opens a Cookthis test recipe so a visitor can try shopping without first creating a meal. The logo is self-hosted and source attribution is recorded in BRAND.md.

## Verification

| Check | Result |
| --- | --- |
| Web Kroger environment | Production; credential and callback values match the private production file |
| Worker Kroger environment | Production; credential and callback values match the private production file |
| Production API authentication | Passed, product.compact client-credentials token received, 1,800-second expiry |
| Nearby stores | Passed, 3 stores returned for public test postal code 45202 |
| Product search | Passed, 3 apple products returned at the selected test store |
| Live home page | Verified in a real browser: logo, US partner label, checkout copy and recipe link visible |
| Production deployment | Web image rebuilt; both services recreated with production Kroger configuration |
| Container health after final deployment | Web and worker healthy |
| Public HTTPS health | /healthz returned status ok, modelConfigured true |
| Local web TypeScript | Passed |
| Local production build | Passed |
| macOS home browser checks and visual baselines | 4 passed: phone/tablet, light/dark |
| Linux home browser checks and visual baselines | 4 passed: phone/tablet, light/dark |
| Credential disclosure | No credentials printed, committed or copied into this status repository |

## How to use it

Open https://cookthis.app. The Kroger partner section is directly below the opening recipe input. Choose Try a recipe, review the recipe, then Shop. Select Kroger, choose a store and sign in to Kroger when asked. Checkout and prices belong to Kroger.

## Limits and next check

This verifies production authentication, locations and products. It does not prove an end-to-end shopper OAuth/cart addition. A shopper must authorize the new production Kroger connection before adding items to their cart; certification connections are separate. No grocery purchase or cart mutation was made during this task. Earlier Stripe customer-portal and extraction limitations remain in the historical record below.

Rollback image for the home-page change: cookthis-web:before-kroger-home. A code rollback must preserve the current private production environment. The source repository is committed locally; only checkpoint Markdown is pushed to the public status repository.

Tests: pnpm --filter @cookthis/web typecheck; pnpm --filter @cookthis/web build; pnpm exec playwright test tests/e2e/recipes.spec.ts -g 'home is usable' --update-snapshots; the same four home tests ran in the Playwright Linux container against the local built web app.

## Historical checkpoints and measured results

The following is preserved history. Current Kroger production results above supersede earlier certification settings.

# Cookthis checkpoint: Premium discoverability

13 October 2026. Source commit b94c992 (UI change beacbf5).

## What changed

Your kitchen now has separate Account and Premium entries. Premium links directly to the visibly labelled Household Premium panel at https://cookthis.app/account#premium. Unpaid households can choose the monthly or yearly subscription without creating a meal or starting a trial first. The optional purchase does not change the existing card-free 14-day trial trigger on saving a second dish.

The production browser reproduced the reported problem: an existing saved three-dish meal showed the Premium feature panel, but no payment button. That household had no trial start recorded, and the old UI required a trial start before rendering the button. This condition has been removed.

A second live issue was found by pressing Choose Premium: Stripe rejected the product because it had no tax category, required by this account's existing Managed Payments default. The product now uses txcd_10103000, SaaS for personal use. The account default was preserved. [Stripe category reference](https://docs.stripe.com/payments/managed-payments/eligibility#product-tax-code-requirements).

Stripe replayed the original rejection for the existing idempotency key even after the product fix. Checkout request identity now includes the selected price, preserving duplicate protection and allowing a fresh request. One standalone diagnostic live Checkout session was created and immediately expired, and the household checkout was successfully created and saved by the application billing function. No payment was submitted.

## Checks

| Check | Result |
| --- | --- |
| Running production web key | Live restricted Stripe key; exact equality to private production environment confirmed without printing values |
| Running monthly/yearly prices and webhook secret | Exact equality to production environment confirmed |
| Live monthly and yearly prices | Active, USD 4.99/month and USD 39/year before tax |
| Stripe account | Live charges and payouts enabled; card payments active, superseding the historical disabled state below |
| Stripe Tax | Active |
| Webhook | Live, enabled, all six billing events configured |
| Local browser regression | Passed: Premium link before trial, second-dish trial, persisted panel after refresh, expired trial leaves shopping usable |
| Billing webhook unit tests | 6 passed |
| Web TypeScript and build | Passed locally; final production image built and deployed successfully |
| Production health after final deploy | Web healthy; public HTTPS /healthz returned status ok, modelConfigured true |
| Production saved three-dish meal | Household Premium panel and Choose Premium visible after reload |
| Production Your kitchen and Account | Premium link and payment controls visible in the signed-in household |
| Live browser Checkout | Choose Premium opened Stripe, showing Subscribe to Cookthis Premium, monthly amount and local/USD currency options |
| Live payment | Not submitted; no new subscription or charge |

## Exact taps

1. Open https://cookthis.app and tap Your kitchen.
2. Tap Premium. Alternatively tap Account and use the Household Premium panel near the top.
3. Choose Monthly or Yearly in Premium payment interval.
4. Tap Choose Premium to open Stripe Checkout. Review the currency and tax total there before paying.

Direct entry: https://cookthis.app/account#premium. Payment management requires the household owner. An already-open checkout for one interval must expire before another interval can be opened, as explained by the existing app message.

The live hosted Checkout was opened from the signed-in production browser and left available to the operator. No payment fields were entered or changed, and Subscribe was not pressed. The anonymous diagnostic session was expired immediately; the household session remains open until its normal expiry. Existing household meals and recipes were unchanged.

Rollback image retained: cookthis-web:before-premium-discovery. Application source is committed locally. Only this checkpoint and its timestamped archive are published to the public status repository.

## Known limitations

No live purchase, payment details submission, subscription or refund was performed. The live customer portal still has no active configuration and must be configured before Manage payment can work for paid customers. Historical extraction and other launch limitations remain recorded below.

Verification commands: pnpm --filter @cookthis/web typecheck; pnpm --filter @cookthis/web build; pnpm exec playwright test tests/e2e/premium.spec.ts --project=phone-light; pnpm exec vitest run tests/billing-webhook.test.ts.

## Historical checkpoint and measured results

The following is preserved history. Current results above supersede earlier billing configuration status.

# Cookthis checkpoint: live Stripe configuration

13 October 2026. Source commit 704647a. The supplied live restricted key and publishable key were installed only in the production environment. Cookthis Premium, both live prices and the public live webhook were created through Stripe's API with idempotency keys. Its signing secret was stored only in production. The web application was rebuilt and recreated successfully. No purchase, Checkout session, charge, subscription or refund was created.

## Configuration verified through Stripe's API

| Setting | Result |
| --- | --- |
| Product | Cookthis Premium, live |
| Monthly price | USD 4.99 per month, active, tax exclusive |
| Yearly price | USD 39 per year, active, tax exclusive |
| Webhook URL | https://cookthis.app/api/billing/webhook |
| Webhook mode and status | Live, enabled |
| Webhook API version | 2026-09-30.endive, matching the installed Stripe SDK |
| Webhook events | Exactly the six listed below |
| Signing secret | Saved in private production environment only |
| Environment file permissions | 0600 |
| Stripe Tax | Active, no pending configuration fields returned |
| Stripe live charges | Disabled; card payments inactive |
| Account details | Submitted; API returned no currently-due fields or disabled reason |
| Live customer portal | No active configuration returned; Manage payment is not ready |

The six enabled events are checkout.session.completed, customer.subscription.created, customer.subscription.updated, customer.subscription.deleted, invoice.paid and charge.refunded.

## Fix and verification

The existing webhook guard recognised sk_live keys only. It now recognises rk_live restricted keys as well, while retaining raw-body signature verification and rejecting test-mode events in live configuration.

| Check | Result |
| --- | --- |
| Signed webhook unit tests | Six passed, including restricted-key live acceptance and wrong-mode rejection |
| Web application TypeScript | Passed |
| Production image build and recreation | Passed |
| Web and worker containers | Healthy |
| Public HTTPS /healthz | status ok, modelConfigured true |
| Public webhook, unsigned request | HTTP 400, Webhook signature required |
| Public webhook, signed test-mode request | HTTP 400, Wrong payment mode |
| Public webhook, signed live configuration probe | HTTP 200, received true |
| Live purchase/refund acceptance | Not performed, explicitly prohibited by the operator |

The configuration probe was an explicitly synthetic, non-financial event signed with the configured production secret. It records a diagnostic billing-event receipt only and contains no customer, invoice, payment or subscription. It does not prove delivery of an actual Stripe purchase event or live payment processing. Initial probes using Python's default user agent were rejected by Cloudflare with HTTP 403/1010; the documented results above use an identifying Cookthis deployment user agent. No WAF protections were changed.

Commands used for validation: `pnpm exec vitest run tests/billing-webhook.test.ts`, `pnpm --filter @cookthis/web typecheck`, `git diff --check`, and production `docker compose --env-file .env -f infra/docker-compose.yml build web` followed by `docker compose --env-file .env -f infra/docker-compose.yml up -d --no-deps --wait web`.

## Remaining before accepting live payments

Stripe reports charges_enabled=false and inactive card payments, despite submitted account details. Review payment activation in the Cookthis Stripe Dashboard; the API did not provide a specific missing field or reason, so no cause is assumed. Live Tax is already active. The live customer portal still needs configuration for payment-method management, invoice history and cancellation at period end. Neither account capabilities nor portal configuration was changed by this product/prices/webhook setup.

The requested live configuration and redeploy are complete. No claim is made that customers can pay while Stripe payment capabilities remain inactive. Test-mode purchase and refund results in earlier checkpoints remain test-mode evidence only. Other extraction, grocery and launch limitations are preserved below.

## Historical checkpoint, preserved verbatim

# Cookthis checkpoint: Supadata activated, hourly monitoring

13 October 2026. The supplied Supadata key was installed only in the production environment file on the server, with file permissions 0600. The worker was recreated. The key was not copied into local environment files, source code or this public repository.

## Live results

| Check | Result | First draft | Background audio | Notes |
| --- | --- | --- | --- | --- |
| Production synthetic, Allrecipes peanut butter cookies | Passed | 32.404 seconds | Initial attempt failed at 18.348 seconds | End-to-end check 33.599 seconds, four ingredients, MiMo spend delta $0.00060494 excluding Supadata credits |
| German pancake Short a1oPd5BgWKk | Draft returned | 56.428 seconds | Not needed | Supadata found native captions, so this does not qualify as the no-caption test; seven ingredients, one validator retry |
| Spanish pancakes umj66RVAbOM | Draft returned | 28.518 seconds | Not needed | Description and native captions available, six ingredients; also does not qualify as no-caption |
| No-caption cookie audio retest | Failed, Supadata HTTP 206 | Timed out at 17.285 seconds | Failed at 46.081 seconds | Genuine no-caption source from the earlier collection; fresh run returned no recipe after 64.115 seconds. Caption paste remains available. |
| Public /healthz | Passed | n/a | n/a | status ok, modelConfigured true |
| Better Stack heartbeat | Up | n/a | n/a | One-hour expectation saved; five-minute grace retained |

The first successful synthetic run occurred at 2026-10-13T04:18:51.751Z. Its pass checks the first draft and expected ingredient presence, not full extraction accuracy or background audio success. No twenty-video accuracy claim is made from these spot checks. Individual drafts still carry guesses for review.

## Timeout found and fixed

The no-caption cookie fixture produced its first draft from creator text. Its initial background audio request hit the generic 15-second HTTP timeout before Supadata returned. The adapter now allows up to 90 seconds for the audio-generation HTTP request, while retaining the overall 300-second background ceiling. Metadata and first-draft requests retain their existing limits. No login or proxy was added. A first attempt to reuse the original queue seed found that the completed job had already been removed. The audio retest therefore runs the same public cookie video through a fresh extraction and waits for background completion.

Runtime source commit efded8a; final documentation commit 25dc551. Worker deployment: passed; recreated worker healthy; private environment permissions verified as 0600. Targeted adapter, transport, source, stage and security checks: 94 passed across five files. Worker TypeScript check passed. Shell syntax and whitespace checks passed.

The direct provider diagnostic also returned HTTP 206 after 41.722 seconds, error transcript-unavailable, message “Transcript Unavailable”, details “No transcript is available for this video”, with neither content nor an asynchronous job ID. The no-caption audio path therefore remains a failed acceptance check. No further provider retries were made.

## Hourly schedule

The live cron entry and infra/deploy.sh now use `0 * * * *`. This runs at the start of each hour, 24 times per day instead of 96. Better Stack was updated and visibly confirmed “Expected every 1 hour”. The home and health monitors retain their existing intervals. Monitoring still reports actual failures; no fake success heartbeat was sent.

This reduces scheduled extraction volume by 75 percent. Provider credits remain separate from MiMo token spend. The current cookie fixture has no native captions, so its background generation costs are additional to the caption-only monthly baseline in earlier documentation. No subscription upgrade or automatic recharge was enabled.

## Reproduce

- Server: `cd /opt/cookthis && bash infra/synthetic.sh`.
- Verify schedule: `cat /etc/cron.d/cookthis-synthetic`.
- Local: `pnpm exec vitest run tests/transcript-transport.test.ts tests/transcript-provider.test.ts tests/transcript-source.test.ts tests/security.test.ts tests/stages.test.ts`.
- Local: `pnpm --filter @cookthis/worker typecheck` and `bash -n infra/deploy.sh infra/synthetic.sh`.

## Known limitations

The 25-second draft target was missed by these runs, although all three returned drafts within the 60-second ceiling. Supadata is now authenticated, but its availability and quota remain external dependencies. Caption paste is retained. Historical phase, grocery, billing and launch limitations below remain unless explicitly superseded here.

## Historical checkpoint, preserved verbatim

# Cookthis checkpoint: YouTube transcript fallback

Source commit: 5112c7e. Direct public YouTube fetching is still first. A new TranscriptProvider interface now supports Supadata after eligible direct-source failures, including the production server's sign-in challenge. No YouTube login, cookies or proxy configuration was added. Caption paste remains available.

## Provider comparison, checked 12 October 2026

| Provider | Existing captions | Audio when captions do not exist | Decision |
| --- | --- | --- | --- |
| Supadata | Native transcript mode | Generate mode, synchronous result or asynchronous job polling | Selected |
| TranscriptAPI | Creator and YouTube-generated caption tracks | Its API documents no_captions and tells the caller not to retry; no audio generation fallback is documented | Not selected |

References: https://docs.supadata.ai/api-reference/endpoint/transcript/transcript , https://docs.supadata.ai/api-reference/endpoint/transcript/transcript-get , https://transcriptapi.com/docs/api/ , https://supadata.ai/pricing . This was a documentation comparison, not a live comparison of paid accounts.

## Behaviour

- Stage 1 requests native captions only. Stage 2 requests generated audio, with distinct evidence labels. The 60-second first-stage and 300-second background ceilings remain in place. When configured, direct metadata gets a 15-second attempt so there is time to use the fallback.
- Provider metadata must report a video duration greater than zero and no longer than ten minutes. Unknown duration fails safely to caption paste.
- Robots denials, private/restricted video messages and network safety rejections are not routed through the provider. Authenticated provider calls use pinned public DNS, one fixed API origin, bounded response bodies, no redirects and no ambient proxies.
- Provider-generated audio is one asynchronous vendor job; direct audio retains MiMo's existing 60-second chunks. Neither path stores media after extraction.
- Source errors retain their direct-fetch reason and append the provider failure. Missing credentials explicitly identify SUPADATA_API_KEY.
- The privacy notice now identifies Supadata's receipt of public video links when configured. The worker alone receives its API key.

## Verification

| Check | Result |
| --- | --- |
| Source, adapter, two-stage and SSRF tests | 102 passed across 6 files, including the process error regression |
| Workspace and test TypeScript checks | Passed |
| Worker production build, local | Passed |
| Production deployment | Passed, web and worker rebuilt and recreated; all five containers healthy |
| Public /healthz after deployment | Passed, status ok and modelConfigured true |
| Production synthetic extraction | Failed in 2.494 seconds at 2026-10-12T13:26:47.303Z; YouTube sign-in challenge, then SUPADATA_API_KEY missing; Better Stack accepted the failure report |
| Live Supadata captions/audio calls | Not run, API key not supplied |

Commands: `pnpm typecheck`; `pnpm exec vitest run tests/transcript-provider.test.ts tests/transcript-source.test.ts tests/sources.test.ts tests/stages.test.ts tests/security.test.ts`; `pnpm exec vitest run tests/process.test.ts`; `pnpm --filter @cookthis/worker build`; production `bash infra/synthetic.sh`.

Exact synthetic failure: “YouTube requires a sign-in check for this request. Paste the caption instead. Supadata fallback needs SUPADATA_API_KEY. Paste the creator’s caption instead.” No provider request was made without a key.

## What is needed

Sign up at https://dash.supadata.ai/auth/sign-up?plan=free and supply the API key shown in the Supadata dashboard. The required private environment variable is SUPADATA_API_KEY. No YouTube credentials are needed. TRANSCRIPT_PROVIDER defaults to supadata; none disables it. After the key is installed in the private production environment, recreate the worker and repeat the real synthetic check, followed by a no-caption video to verify generated audio.

Pricing checked today: the free plan has 100 credits per month. Native transcripts use one credit; audio generation uses two credits per minute; metadata adds a credit. A 15-minute synthetic check deliberately bypasses the recipe cache, approximately 2,880 checks and 5,760 credits per 30-day month for the native-caption path alone. The free plan is suitable for initial validation only. No plan was purchased and no automatic recharge was enabled. Provider charges are separate from MiMo token-cost metrics. Review the dashboard quota before sustained monitoring; per-process pacing does not coordinate rate limits across multiple workers.

## Known limitations and next step

No claim is made that blocked YouTube extraction has been restored before the provider is authenticated and tested live. Caption paste remains the working fallback. Existing grocery, payment and launch limitations below remain historical unless explicitly superseded above.

## Historical checkpoint, preserved verbatim

# Cookthis checkpoint: current live status

The Home redesign is live at https://cookthis.app: the three-step guide comes first, followed by recipes beneath their own heading in a ruled menu layout. AKA Foods typography, colours and original artwork are retained.

The production health endpoint was checked again for this checkpoint and returned status ok with modelConfigured true. No application changes were made since source commit 671f5ab. The previous eight phone/tablet browser checks passed in light and dark mode; those tests were not rerun for this status-only checkpoint.

The requested design work is complete. Existing launch limitations and all verification details remain in the previous checkpoint preserved below. No new extraction, payment or grocery acceptance result is claimed.

## Previous checkpoint, preserved verbatim

# Cookthis checkpoint: steps first, recipes as a menu

The operator asked for the steps above the recipes, a distinct heading for the recipes, and a more deliberately designed layout.

## What changed

Home now opens its lower section with a blue three-step guide: bring the recipe, check the details, make it your dinner. An oversized heading, open columns, numbered rules and one asymmetric corner give this section a clear identity. On phones the steps form a vertical sequence.

The recipes follow beneath their own heading, “A few recipes to try.” The previous three coloured cards are replaced by a ruled recipe menu with course labels, larger dish names, ingredient counts and arrow links. All three remain clearly labelled original Cookthis test recipes. Outfit, AKA Foods colours, the original artwork and footer logo are retained.

Source commit: 671f5ab. The change is limited to Home and its visual checks. No household data, recipe model, extractor, shopping or cooking logic changed.

## Verification

| Check | Result |
| --- | --- |
| Next.js production build and bundled type checks | Passed |
| macOS browser checks, phone/tablet in light/dark | 4 passed |
| Linux browser checks, phone/tablet in light/dark | 4 passed |
| Home screenshot baselines | 8 updated and then verified without updating snapshots |
| Steps before recipes | Explicit browser assertion passed on all eight combinations |
| Dedicated recipe heading and three working example links | Existing/expanded browser assertions passed |
| Visual inspection | Phone dark and tablet light reviewed for wrapping, spacing and hierarchy |

Exact verification commands:

```sh
pnpm --filter @cookthis/web build
pnpm exec playwright test tests/e2e/recipes.spec.ts --grep 'home is usable'
```

Linux used the official Playwright v1.64.0 container against the same local production build, with E2E_EXTERNAL=1 and E2E_BASE_URL pointing to the host test server. The final runs above did not update screenshots. The wider application suite and live extraction/payment tests were not repeated for this Home-only change. Earlier results are historical below.

## Live deployment

The revised Home is live at https://cookthis.app. Public Home returned HTTP 200 with successful certificate verification. The health endpoint returned status ok and modelConfigured true. Caddy, web, worker, Redis and backup are all running and healthy.

The real browser confirms the three-step guide appears before “A few recipes to try.” and the three recipe links use the new menu layout. A live screenshot was saved locally. Only the web container was rebuilt and replaced. No credentials or database migrations were changed.

The previous web image is retained as cookthis-web:before-home-editorial-20261012. Runtime rollback from /opt/cookthis on the server:

```sh
docker image tag cookthis-web:before-home-editorial-20261012 cookthis-web:latest
docker compose --env-file .env -f infra/docker-compose.yml up -d --no-deps --force-recreate --wait web
```

Restore the intended source revision before a later rebuild. The server still uses an uploaded source archive without a private Git remote.

## Known limitations and next work

The requested Home redesign is complete and live. No new blocker was found. The existing launch limitations remain as recorded below, including the production YouTube source sign-in block, incomplete production payment/grocery attribution setup, R2 upload integration, Sentry, independent live video accuracy validation, creator verification, real-device acceptance and legal review. This visual change does not resolve or retest those integrations.

The previous mobile Lighthouse result of 92 performance and 100 accessibility is historical from the preceding AKA update. Lighthouse was not rerun for this layout revision. Next work remains the documented launch limitations, or further visual direction from the operator.

## Historical checkpoint, preserved verbatim

# Cookthis checkpoint: AKA brand update

The operator requested AKA Foods brand-book styling and a live update. This checkpoint records that visual change independently from the existing launch limitations below.

## What changed

Cookthis now uses self-hosted Outfit throughout the website, the AKA Foods source blue, pale blue/mint/peach surfaces, and accessible dark adaptations. Home uses the original ingredient-bowl artwork. The footer credits AKA Foods with its original SVG. The installed-app icon, manifest colours, shared plan image and media-kit typography follow the same identity. Creator-selected media-kit accents remain unchanged. Recipe guesses and errors keep a separate warning tone, dotted underlines and written explanations.

Core AKA Foods is the chosen route. Connected Drive metadata and the BrandBook inventory were checked on 12 October 2026. The original archive remains dated 30 June 2026, and no replacement visual guide was found in targeted discovery. Bundled assets are from the verified 28 September snapshot. This was a targeted freshness check, not an exhaustive branding approval audit.

Source commit: abaf479. Application source stays local and on the private server. The public status repository contains checkpoint Markdown only.

## Verification

| Check | Result |
| --- | --- |
| Production build, local | Passed |
| Workspace type checking | Passed |
| Final web type checking | Passed |
| macOS visual/browser checks | 32 passed |
| Linux visual/browser checks | 32 passed |
| Screenshot baselines | 80 updated: ten screens, four light/dark phone/tablet combinations, two platforms |
| Main screens | Home, Recipe, Meal, Shop, Plan, Cook, Helper, second screen, Install, iOS guide |
| Second-screen text/bar separation | Existing geometry assertions still pass |
| White text on primary blue | 4.61:1 contrast |
| Muted text on pale blue/mint/peach | 5.32:1 / 4.92:1 / 5.03:1 |
| Dark primary button text/background | 8.90:1 |
| Original logo/illustration | Byte-identical to verified archive assets |
| Local generated icon | HTTP 200, visually inspected |

Exact verification commands:

```sh
pnpm build
pnpm typecheck
pnpm --filter @cookthis/web typecheck
pnpm exec playwright test tests/e2e/recipes.spec.ts tests/e2e/household.spec.ts tests/e2e/shopping.spec.ts tests/e2e/conductor-visual.spec.ts tests/e2e/helper-visual.spec.ts tests/e2e/launch.spec.ts --grep 'home is usable|original recipe|meal layout|shopping layout|plan and cook screens|helper and second screen|phone sharing|launch pages'
```

Linux ran the same 32 selected checks in the official Playwright v1.64.0 container against the local production build with E2E_EXTERNAL=1 and the host browser-test server. Baselines were first regenerated, representative home/recipe/cook/timeline screenshots inspected, then the final checks ran without updating baselines. No model calls, live fixture accuracy run or payment transaction was required or claimed for this visual change.

## Live deployment

https://cookthis.app now serves the updated design. The production web container was rebuilt using the production environment and restarted without rebuilding the worker, Redis, Caddy or backup service. No database migration or credential change was needed. The prior web image is retained on the server as cookthis-web:before-aka-20261012.

| Production check | Result |
| --- | --- |
| Public home | HTTP 200, valid certificate |
| Public health endpoint | HTTP 200, status ok, model configured |
| Direct-origin health endpoint | HTTP 200, TLS verification successful |
| Generated icon | HTTP 200 |
| Caddy, web, worker, Redis, backup | All five running and healthy |
| Real browser on live Home and Recipe | Outfit loaded, original artwork/logo loaded, guesses remain marked |
| Mobile Lighthouse, live Home | Performance 92, accessibility 100, best practices 100, SEO 100 |

Lighthouse 13.5.0 measured the live public site with the default mobile simulation. This is one measured run, not a guarantee for every network or page. First contentful paint was 1.6 seconds and largest contentful paint was 3.1 seconds. The new look clears the 90 mobile performance target on this run.

Verification commands:

```sh
curl -fsS https://cookthis.app/healthz
curl --resolve cookthis.app:443:2.28.41.10 https://cookthis.app/healthz
pnpm dlx lighthouse https://cookthis.app --chrome-flags='--headless --no-sandbox' --only-categories=performance,accessibility,best-practices,seo --output=json --output-path=artifacts/aka-live-lighthouse.json --quiet
```

Runtime rollback, from /opt/cookthis on the production server:

```sh
docker image tag cookthis-web:before-aka-20261012 cookthis-web:latest
docker compose --env-file .env -f infra/docker-compose.yml up -d --no-deps --force-recreate --wait web
```

The saved image rolls the running web app back. Before rebuilding after a rollback, also restore the intended source revision. The server still uses an uploaded source archive, with no private Git remote configured.

## Known limitations and next work

The requested branding update is complete and deployed. Existing production limitations remain: the YouTube synthetic extraction is blocked by a source sign-in check on the server IP; the caption/text fallback works. Production Stripe and Kroger credentials, affiliate attribution, R2 application upload integration, production Sentry, real Android/iOS device acceptance, independent live 20-video accuracy acceptance, creator bio verification and legal review remain unverified or incomplete as described in the preserved checkpoint. No source bypass, payment test, grocery order or private-data publication was performed for this visual update.

Next work should address those launch limitations. The previous extraction failure is preserved rather than reported as fixed by a visual deployment.

## Historical checkpoint, preserved verbatim

# Production deployment checkpoint, 12 October 2026

Cookthis is live at https://cookthis.app and https://www.cookthis.app. The requested email setup, production deployment and monitoring configuration have been performed. The website and email path are verified. A real YouTube source-access failure remains active and is visible in monitoring. This is not a declaration that every original Phase 7 acceptance gate is complete.

## Deployment and verification

| Check | Result | Evidence and scope |
| --- | --- | --- |
| Fresh server | Passed | Ubuntu 24.04.4 LTS, no Docker initially; source uploaded from the local Git archive |
| infra/deploy.sh | Passed | Installed Docker, built standard web/worker/migration/backup images, ran migrations and started services |
| Hosted migrations | Passed | 15 entries in drizzle migration journal |
| RLS observation | Verified | 29 public tables have RLS enabled, a count rather than a new full permission audit |
| Containers | Healthy | Caddy, web, worker, Redis and backup |
| https://cookthis.app | HTTP 200 | TLS verification result 0 |
| https://www.cookthis.app | HTTP 200 | TLS verification result 0 |
| Public /healthz | HTTP 200 | status ok, modelConfigured true |
| Direct origin certificate | Validated | HTTPS request addressed directly to the origin with cookthis.app as hostname, TLS verification result 0 |
| Cloudflare TLS | Full (strict) | Read from live dashboard |
| Owner sign-in | Passed | Resend delivered sign-in mail; a fresh link opened the production household |
| Older sign-in link | Expired | Initial link failed as expired; a fresh request succeeded, no failure concealed |
| Owner /ops | Passed | Production authenticated owner opened the operator dashboard |
| Plain-text recipe extraction | Draft produced | Real public API and MiMo, original Cookthis test recipe, 4 ingredients, 25.876 seconds; the 25-second target was exceeded by 0.876 seconds |
| Inbound recipe email | Passed | Original Cookthis test recipe sent to private recipes-subdomain address, real signed webhook queued processing, database status complete with Paprika potatoes saved |
| Worker type check | Passed | Current worker source |
| Unit suite | 274 passed | 28 test files; no new full browser, coverage or twenty-video live suite run in this turn |
| Operational script checks | Passed | Node syntax, shell syntax and git diff whitespace validation |
| Synthetic metrics exclusion | Passed | Failure count remained 2 before/after a fresh synthetic probe on the updated worker |

Local source commit: 5cac75c. The web deployment uses the prior application source plus the current production environment. The worker was rebuilt and restarted with the synthetic-metrics exclusion. The new operational scripts and deploy script are on the server. No credentials were committed or published.

## Email setup

Resend sending for cookthis.app and receiving for recipes.cookthis.app are both verified. Tracking is disabled. The root-domain Cloudflare mail routing and its existing SPF/DKIM records remain intact. Six additional records were added through the Cloudflare DNS interface and their public resolution was verified.

| Type | Name within cookthis.app | Content or purpose | Status |
| --- | --- | --- | --- |
| TXT | resend._domainkey | Resend-generated public DKIM key | Verified |
| MX | send | feedback-smtp.eu-west-1.amazonses.com, priority 10 | Verified |
| TXT | send | v=spf1 include:amazonses.com ~all | Verified |
| CNAME | rsend | send.forge.rmta.net, DNS only | Verified |
| TXT | resend._domainkey.recipes | Resend-generated public DKIM key | Verified |
| MX | recipes | inbound-smtp.eu-west-1.amazonaws.com, priority 10 | Verified |

Cloudflare file import was unavailable because the browser extension lacked file-URL upload permission. All six records were instead added individually; no extension permissions were changed.

The receiving capability routes mail on recipes.cookthis.app. The webhook subscribes to email.received at https://cookthis.app/api/email/inbound. Its signing secret is stored only in the private production environment on the Mac and server. No additional credential was requested. The real receiving test confirms webhook authentication and processing, not merely a mocked event.

Supabase Auth custom SMTP is saved: smtp.resend.com, port 465, username resend, sender hello@cookthis.app, name Cookthis, using the supplied Resend key. The Site URL is https://cookthis.app with callback and confirmation redirect patterns allowed. A fresh email sign-in succeeded in the same browser. The household's recipe-reminder preference remained off during testing.

## Monitoring and real failure

The current Better Stack Free allowance supports the two HTTP monitors and one heartbeat used here. No paid plan upgrade was purchased. Reference: https://betterstack.com/uptime .

| Monitor | Configuration | Observed result |
| --- | --- | --- |
| Home | https://cookthis.app, every 3 minutes, certificate verification on, email to existing primary responder | Up |
| Dependencies | https://cookthis.app/healthz, requires status ok, every 3 minutes | Up |
| Live video fixture | Expected every 15 minutes, 5 minutes of grace; uncached source/model attempt | Down, real source failure reported |
| Cron | /etc/cron.d/cookthis-synthetic, every 15 minutes; cron service active | Installed and manual equivalent runs verified |
| Failure notification | Better Stack failure endpoint plus support email throttled to one/hour | Failure accepted by Better Stack; support email delivered by Resend |

The fixture is the public Allrecipes three-ingredient peanut-butter-cookie YouTube video already in the fixture set. A dedicated cache key forces source collection; a missing recipe, empty ingredient evidence or missing expected ingredient is a failure. The first observed check failed in 2.577 seconds. The final metrics check failed in 2.503 seconds with the exact reason: "YouTube requires a sign-in check for this request. Paste the caption instead." Better Stack showed an active Down incident. No platform login or proxy was used and no success heartbeat was sent for the failure.

The check and scripts are shipped in apps/worker/scripts/synthetic.mjs and infra/synthetic.sh. Run on the server with `cd /opt/cookthis && bash infra/synthetic.sh`. Exit 1 and the Down monitor are intentional evidence of the current source failure. Logs are in /var/log/cookthis-synthetic.log. The heartbeat URL is private and is not published here. Future synthetic successes/failures do not inflate household extraction totals. The first two probes happened before that exclusion and remain in the initial day's historical counters.

Better Stack native notifications target the existing primary responder. Synthetic failures additionally go to support@cookthis.app; the real failure email showed delivered. A separate Sentry deployment/alert test was not part of this completed setup.

## What remains and what is needed

No further credential is needed for the verified email and HTTPS deployment. Automatic YouTube video retrieval from this VPS is blocked by the platform's sign-in check. Users can paste creator captions or recipe text; this path produced a real draft. The video access issue needs further source investigation within the existing no-login/no-proxy rules. Do not count the deployed site or healthy dependencies as proof of working video retrieval.

Production Stripe and Kroger are still unconfigured because supplied keys belong to test and certification environments. Impact attribution, live grocery ordering, real creator verification and R2 application integration remain unverified/incomplete as previously recorded. The R2 credentials are stored, which is not proof of integrated uploads. Actual Android/iPhone sharing, legal review, independent twenty-video accuracy acceptance and a production backup restore remain outstanding. No grocery order, Stripe charge, outreach message or launch post was sent in this turn.

The server received a Git archive, not a Git clone. A private Git remote and server checkout are still needed before infra/update.sh can pull future source updates. This limitation is documented in DEPLOY.md. The successful clean server deployment does not erase the rest of the original definition of done.

## Owner access and next steps

Open https://cookthis.app, choose Your kitchen and request a sign-in email using the configured operator address. Open the newest link in the same browser. /ops is available to that verified owner.

The next work is to resolve the production video source issue, arrange private source updates, finish the remaining live partner acceptance and run physical-device sharing and backup-restore checks. Stop at this checkpoint per the operator's request. Historical results below are preserved; the current deployment and email evidence supersede only the older statements that those were not yet deployed or verified.

## Historical checkpoints and result tables, preserved verbatim

# Production R2 credential checkpoint

The operator supplied R2 S3 credentials. R2_ACCESS_KEY_ID and R2_SECRET_ACCESS_KEY were updated only in the local private production environment and server /opt/cookthis/.env. The remote update changed only these two fields. The additional Cloudflare API token was not stored because it is not needed for the configured S3 credentials.

| Check | Result |
| --- | --- |
| R2 access key ID and secret | Stored in both production files |
| Local and server environment | SHA256 matches |
| Permissions | Owner-only 0600 |
| Local production file | Ignored by Git |
| R2 authentication, bucket access or uploads | Not tested in this credential-only step |
| Application deployment | Not started |

No application code was changed. R2 bucket access and application integration, Resend domain/webhook/SMTP setup, production billing and grocery credentials, and the remaining launch acceptance checks still require verification as recorded below. No credential values are published.

## Prior full checkpoints and result tables, preserved verbatim

# Production Resend credential checkpoint

The operator supplied the Resend API key. RESEND_API_KEY was updated in the local private production environment and server /opt/cookthis/.env. The remote update changed only this field and retained the other server settings.

| Check | Result |
| --- | --- |
| RESEND_API_KEY | Stored in both production files |
| Local and server environment | SHA256 matches |
| Permissions | Owner-only 0600 |
| Provider authentication or email delivery | Not tested in this credential-only step |
| Application deployment | Not started |

No email was sent, no webhook was created and no application code was changed. Resend sender verification, receiving setup, webhook signing secret and Supabase SMTP still require setup and verification. R2 and other launch acceptance gaps remain below. No credential values are published.

## Prior full checkpoints and result tables, preserved verbatim

# Production database credential checkpoint

The operator supplied the existing Supabase database password and authorized storing it only in production DATABASE_URL. The local production environment and server /opt/cookthis/.env now have the completed session-pooler URI with sslmode=require. The password was not added to any separate password variable, local development configuration, tracked file or public report.

| Check | Result |
| --- | --- |
| Production DATABASE_URL | Updated on Mac and server |
| Local and server files | SHA256 matches |
| Local production file | Ignored by Git |
| Environment access | Owner-only permissions retained |
| Database authentication or migrations | Not run in this step |
| Application deployment | Not started |

Resend and R2 configuration, live billing/grocery credentials and the remaining production acceptance checks remain as listed below. No application changes or tests were run. The requested credential update is complete.

## Previous full checkpoints and tables, preserved verbatim

# Production environment preparation checkpoint

The operator reported the server is running and both domain names point to it. SSH access verified the hostname cookthis-1 and Ubuntu 24.04.4 LTS. No application deployment, database migration, paid purchase or production email was performed in this step.

A private production environment was created from the checked-in template. A local .env.production and server /opt/cookthis/.env contain identical bytes; both are permission 0600 and the local file is ignored by Git. The existing local development .env was preserved. No credential values, private addresses, project identifiers or environment contents are published here.

| Configuration | Result |
| --- | --- |
| DOMAIN and APP_URL | cookthis.app and its HTTPS origin configured |
| Operator access | Requested verified operator email configured |
| MiMo | Existing authorized key and model configuration carried over |
| Supabase URL and anon key | Retrieved from the existing Cookthis production project; anon role and project binding checked |
| Database connection | Session-pooler host, user and port identified; password unavailable from dashboard, DATABASE_URL remains empty |
| Turnstile | Created Cookthis production widget for cookthis.app, Managed mode, no pre-clearance; both keys stored privately |
| Push | Fresh production VAPID pair generated and stored privately |
| Internal services | Redis container address, proxy mode, worker settings and cost controls configured |
| Email addresses | Sender and support addresses configured; separate recipes subdomain selected for inbound to preserve existing support mail routing |
| Resend | No API keys exist. Named full-access key form prepared but not submitted; credential creation awaits operator action |
| R2 | Account ID and intended cookthis-images bucket name configured; Cloudflare account email verification blocks R2 setup |
| Stripe | Existing credentials are test-only; not copied into production |
| Kroger | Existing credentials are certification-only; not copied into production |
| Deployment | Not started, as requested |

## Values or actions still needed

1. Supabase database password for the existing Cookthis project, or a completed Session pooler connection URI. Dashboard: Connect, Direct connection string, Session pooler, URI. The password is not displayed by Supabase. A forgotten password must be reset by the operator. The URL and anonymous key are already retrieved; the unused service-role key is not requested.
2. Resend API key: https://resend.com/api-keys, Create API key, Name Cookthis production, Permission Full access, Add. The prepared dialog is left open. Copy the generated key into RESEND_API_KEY in the private production file. Full access supports receiving retrieval as well as outbound email and grants broader account access. Existing keys cannot be retrieved because none exist.
3. R2: complete Cloudflare account email verification first. Then R2 Object Storage, create cookthis-images if absent, API Tokens Manage, Create API token, Object Read & Write, Apply to specific buckets only, cookthis-images. Required returned fields are Access Key ID and Secret Access Key, not the general Cloudflare API token. Store as R2_ACCESS_KEY_ID and R2_SECRET_ACCESS_KEY. Bucket/account information is already selected. Reference: https://developers.cloudflare.com/r2/get-started/s3/ .

New Resend and R2 access tokens grant persistent account access. Their final creation is left for operator confirmation or the operator's own click under browser permission rules. Turnstile is already configured; no Turnstile values are requested.

After the email key and domain setup are available, configure the Resend receiving webhook and its signing secret, sender verification and Supabase SMTP. No need to request a signing secret before that endpoint exists. Existing support-mail routing must remain intact. Before paid public billing or real grocery carts, live Stripe credentials and prices/webhook and Kroger production credentials are still required; the test/certification keys are not substitutes. Affiliate approval, R2 application integration and monitoring verification remain as recorded below.

## Verification and stopping point

Environment parse and key format checks passed; local/server SHA256 hashes matched; server file ownership is root with mode 0600. The Turnstile dashboard lists the created widget. No new unit/browser/application tests were run. The production file is intentionally incomplete and must not be deployed yet. Stop here per the operator's request, awaiting the missing database and service credentials/actions.

## Historical checkpoints and test tables, preserved verbatim

# Hetzner server preparation checkpoint

Account verification no longer blocks project creation. The cookthis project was created successfully in the operator's logged-in Hetzner account.

| Item | Verified status |
| --- | --- |
| Project | cookthis created |
| Prepared server | cookthis-1, CPX32, shared x86 AMD, 4 vCPU, 8 GB RAM, 160 GB SSD |
| Location and image | Falkenstein and Ubuntu 24.04 selected |
| Networking | Public IPv4 and IPv6 selected |
| Optional services | No additional volumes, firewall or backups selected, as requested |
| Console price | USD 41.99/month server plus USD 0.60/month IPv4, total USD 42.59/month excluding VAT |
| SSH | Dedicated Ed25519 key generated locally; public key filled in the Add SSH key dialog as mac, not yet submitted |
| Purchase | Not submitted; no server or IPv4 allocated |
| App changes and test runs | None |

The browser is left at the SSH-key confirmation step. The operator must add the prepared key, review the configured server and price, then complete Create & Buy now. Adding the key authorizes SSH access for its holder; purchasing starts recurring charges. The private key remains outside the application repository and is not published. No credentials, public key, account screenshots or account identifiers are included in this public report.

Next: after the operator completes creation, verify server status and IPv4 and proceed with deployment. Phase 7 and prior provider acceptance gaps remain incomplete. All earlier measurements are historical, not rerun during this setup.

## Earlier checkpoint and full result tables, preserved verbatim

# Hetzner setup checkpoint

The operator authorized preparing the Cookthis project and server in the logged-in browser. Hetzner rejected the New project action because account verification is required. The browser is left on the official account verification page for the operator to complete.

| Item | Status |
| --- | --- |
| Existing Hetzner sign-in | Verified in the requested browser profile |
| Project cookthis | Not created, blocked by account verification |
| Server cookthis-1 | Not created or purchased |
| SSH key | No key generated or uploaded in this attempt |
| Public IPv4 | Not assigned |
| Application or test changes | None |

Next: the operator completes Hetzner account verification. Then resume project and server preparation with Falkenstein, Ubuntu 24.04, CPX32 and the requested options. Review the actual quoted price before the paid creation step. Phase 7 remains incomplete; all previous test results, limitations and deployment gates below are unchanged. No account documents, card information, secrets or screenshots are published here.

## Prior full checkpoint and tables, preserved verbatim

# Checkpoint publication, 2026-10-12 07:40:19 UTC

Published again at the operator's request. No application changes or new test runs since the Phase 7 checkpoint below. Application HEAD remains 6b475a2 and the working tree is clean. All measurements below retain their original verification scope and time; local service health has not been rechecked for this publication.

Phase 7 remains incomplete. Next work is the outstanding local behavior, production configuration, fresh-VM deployment and real-device, email, monitoring and provider acceptance checks listed below. This publication does not announce a public launch.

## Full prior checkpoint and tables, preserved verbatim

# Phase 7 checkpoint, 12 October 2026

Status: local release candidate. Phase 7 is not complete and no public launch is claimed.

## What works locally

- Installable manifest and share entry point accept a shared link or caption for review before extraction. The existing cooking service worker is registered across the app.
- The iOS Shortcut is signed by Apple's signing command, downloadable at `/ios`, and accompanied by an install guide and animation. It targets the public cookthis.app domain.
- Six seasonal pages contain copyable scheduling practice using the three original Cookthis test recipes. They are explicitly not traditional holiday menus. Passover excludes the oat dessert.
- An authenticated plan image download creates a private PNG. It shows up to twelve blocks with UTC times, not an unlimited full-plan export.
- Signed Resend inbound events map a private household address to a queued extraction. Address tokens are hashed; duplicate deliveries are idempotent; household deletion cascades through mail records. Only plain-text email bodies are supported, not attachments or HTML-only messages.
- Owner-controlled email preferences are opt-in. Daily jobs prepare recipe links, cook invitations, trial notices, monthly recaps and operator summaries. Weekly reports count corrections by field; they are not an independent accuracy measurement.
- Legal pages name AKA Foods Co., USA and are marked for lawyer review. Creator removal instructions and unpublished launch drafts are included.
- Deployment, operations and secret setup documentation now covers these flows. A preflight script reports missing setting names without printing secrets.

## Verification

| Check                                          | Result                                                         | Scope                                                                                                       |
| ---------------------------------------------- | -------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Type checking                                  | Pass                                                           | Web, worker, packages and tests                                                                             |
| Unit suite                                     | 274 passed                                                     | Local deterministic tests                                                                                   |
| Configured core line coverage                  | 98.55%                                                         | Extraction, recipe, scaling, shopping, timeline, security, push and grocery modules; not the whole codebase |
| Email route and share checks                   | 8 passed                                                       | Signature, recipient token, payload and mocked queue behavior                                               |
| Email database checks                          | Pass                                                           | Real local Supabase and Redis, cached original recipe fixture; duplicate handling, RLS and cascade deletion |
| Browser regression suite                       | 68 passed, 36 intentional duplicate skips                      | Local Supabase sign-in; deterministic extraction model; four viewport/theme projects                        |
| Additional seasonal persistence check          | 1 passed                                                       | Sign in, copy three dishes, reload and retain dishes                                                        |
| Final existing-screen visual checks            | 24 passed                                                      | Home, Recipe, Meal, Shop, Plan, Cook, Helper and timeline coverage across configured projects               |
| Production source build                        | Pass                                                           | Next.js and worker                                                                                          |
| Updated local containers                       | Healthy                                                        | New web and worker images reused existing dependency image                                                  |
| Lighthouse mobile, local home page             | Performance 94, accessibility 100, best practices 100, SEO 100 | Local simulated mobile run, not a public deployment benchmark                                               |
| iOS Shortcut signing                           | Pass                                                           | Apple signing tool and download bytes only; no iPhone import claim                                          |
| Deployment shell syntax and Compose validation | Pass                                                           | Static validation, not a fresh-server deployment                                                            |

The final launch-page browser suite passed 9 tests with 3 intentional persistence duplicates skipped. It includes eight install/iOS screenshot baselines across phone/tablet and light/dark, shared-input review, legal/example labels and the seasonal save/refresh check.

The full browser suite ran before the final footer touch-target polish; the affected visual checks were rerun afterward. The seasonal save check was added and run separately. Final installation-page visual checks are recorded in the published checkpoint.

The 60x conductor run still passed both scenarios: on-time maximum serve deviation 0.365 minutes, delayed maximum serve deviation 0.380 minutes, zero oven conflicts in each. The injected delay moved serve time by 12.779 minutes. Read latency p50/p95 was 42/60 ms on-time and 38/65 ms delayed.

The standard clean Docker builds were stopped after prolonged dependency downloads. Successful replacement local builds copied current source onto an existing dependency image. They do not prove the required clean Ubuntu deployment.

## Exact local checks

```sh
pnpm install --frozen-lockfile
pnpm db:local
pnpm typecheck
pnpm test:coverage
pnpm build
docker compose -f infra/compose.test.yml up -d --wait
pnpm exec tsx apps/worker/scripts/check-email.ts
pnpm test:e2e
node scripts/launch-preflight.mjs
```

The preflight currently exits 2, correctly reporting missing production settings. Browser tests use a deterministic extraction model and local Supabase. Live MiMo, Resend, affiliate and physical-device tests must not be inferred from those results.

## Remaining acceptance and limitations

- Public DNS, Hetzner access and production Supabase configuration have not been supplied. No fresh Ubuntu 24.04 VM was deployed. localhost:3000 is a local preview only.
- Production settings missing from `.env`: DOMAIN, SUPABASE_URL, SUPABASE_ANON_KEY, DATABASE_URL, NEXT_PUBLIC_TURNSTILE_SITE_KEY, TURNSTILE_SECRET_KEY, RESEND_API_KEY, RESEND_WEBHOOK_SECRET and OPS_EMAILS. Local preview overrides do not substitute for production credentials.
- Android sharing from TikTok and actual iPhone Shortcut import still need physical-device checks on the public HTTPS site.
- Real Resend receipt, sending, DNS verification and mail routing are unverified. Keep existing support mailbox MX routing intact. Saved email recipes do not yet receive later background audio updates. The monthly recap selects recent saved recipes rather than proving that each was never cooked.
- Monitoring alert delivery, the required recurring synthetic extraction, R2 media storage and a fresh-server backup restore remain unverified or incomplete. No uptime promise is made.
- Impact credentials and real attribution reporting are absent. Actual creator handle control has not been verified. These remain external Phase 6 gaps.
- The earlier real Stripe test-mode purchase, signed webhook handling, duplicate replay, refund and premium revocation passed. No real funds moved. Production webhook delivery and a yearly purchase are not verified.
- Earlier Phase 1 independent video accuracy, full grocery checkout/attribution and other historical limitations remain as recorded in prior checkpoints. They have not been replaced by these local test results.
- The starter recipes and meal are seeded; a cook session begins when the user plans a cook rather than being automatically seeded on first login.
- Legal review, production prices and current monthly infrastructure quotes are still required. Launch drafts have not been published and no outreach has been sent.

## Next

Finish missing local behavior and provider setup, run the standard clean container build and the documented fresh-VM deployment, then verify real phone sharing, Resend delivery, monitoring and an attributed grocery checkout. Do not mark Phase 7 complete until those acceptance checks pass. Stop at this checkpoint for the operator's next instruction.

## Source and publication

Local application commits: 2ab449d, 6e229d0 and 6b475a2. Application code and secrets are not published to this status repository. The local preview is http://localhost:3000 and is not a public live URL.

## Historical checkpoints, preserved verbatim

The following text records earlier verification at the time it was performed. Later results above supersede only explicitly updated findings.

# Phase 6 live Stripe purchase and refund checkpoint

2026-10-12. The operator completed the head-office address in the Cookthis test account. Stripe Tax is active. The real Stripe hosted test-mode purchase and full-refund acceptance now passes against the local Cookthis application. No real money moved. Phase 7 has not started.

## Acceptance results

| Check | Result | Measured evidence |
| --- | --- | --- |
| Correct account | Passed | Dashboard and supplied API key use the same Cookthis test account, distinct from the separate Cookthis sandbox. |
| Stripe Tax readiness | Passed | Tax settings API reports active; missing head-office blocker resolved by the operator. |
| App-to-hosted Checkout | Passed | Signed-in household opened monthly Premium from the app after starting its multi-dish trial. Checkout visibly labelled Sandbox. |
| Test purchase | Passed | USD 4.99 paid using Stripe's 4242 test card and a clearly named test customer. Session complete, payment paid. |
| Automatic tax | Passed | Enabled; calculation status complete. Checkout showed USD 0.00 tax, USD 4.99 total. This does not prove tax collection in a taxable registered jurisdiction. |
| Payment webhooks | Passed | invoice.paid, customer.subscription.created and checkout.session.completed delivered by the connected Stripe CLI, each HTTP 200 from the real signed webhook route. |
| Household Premium | Passed | Database plan premium, subscription active, one USD 4.99 invoice ledger row. Real browser displayed Premium through November 12, 2026. |
| Duplicate handling | Passed | Fetched the actual Checkout event from Stripe and replayed it with a fresh SDK-generated test signature. HTTP 200, duplicate true, billing snapshot unchanged. This replay was locally signed, not a second Stripe-origin delivery. |
| Full refund | Passed | Stripe refund succeeded for the complete USD 4.99 test payment. |
| Refund webhooks | Passed | charge.refunded and customer.subscription.deleted delivered by Stripe CLI, both HTTP 200. |
| Access removed | Passed | Database plan free, premium_until null, subscription canceled, refunded true, trial ended. Browser after refresh showed the trial-ended/free-plan state. |
| Ledger | Passed | One 499-cent payment row and one 499-cent refund row, net zero USD. No duplicate invoice row. |
| Credentials | Protected | Remain only in ignored .env with 0600 permissions; no keys or webhook secrets committed or published. |
| Application regression | Historical | No application behavior changed in this pass. Earlier 266 unit and 60 browser passes are preserved below; not rerun for this configuration-only verification. |

## What works and how to verify

The hosted checkout, real signed payment events, Premium grant, idempotent replay, full refund, subscription cancellation and removal of household access have all been exercised against the real Stripe test API and local database. The existing signed-in browser household was used, with a Cookthis test dinner. Its test trial is now ended as required by the full-refund policy; its recipes and meals remain intact.

Evidence screenshots remain local at artifacts/stripe-premium-purchase.png and artifacts/stripe-premium-refund.png. The local verification script is artifacts/stripe-live-verify.mjs, contains no keys, and reads the ignored environment. `node artifacts/stripe-live-verify.mjs` inspects this completed acceptance session; `node artifacts/stripe-live-verify.mjs replay` repeats the locally signed duplicate check. Secrets and private identifiers are not copied into this public report.

General repeat procedure: start the local app, keep the Stripe CLI listener forwarding the six billing event types to localhost:3000/api/billing/webhook, open a household with a multi-dish trial, choose monthly Premium and complete hosted Checkout with Stripe's test card. Check Premium in the browser and database, refund the latest test payment in full, then refresh and confirm Free. See BILLING.md for configuration and existing automated checks.

## Limits and next work

This is a real-provider test-mode acceptance on localhost, not a production deployment. The webhook currently uses the local CLI forwarder; a public production endpoint, live-mode payments and live tax registrations remain deployment tasks. Only monthly purchase/refund was exercised; the yearly price exists but was not purchased. No additional registrations or business identity facts were invented.

Stripe acceptance is complete. Phase 6's other external checks remain separate: configured live Impact affiliate reporting and a real creator's bio challenge are unverified. Their injected and database tests, the implemented operator dashboard and creator workflows, and all earlier results remain in the historical checkpoints below. Stop here for the phase checkpoint, with no Phase 7 work started.

---

# Historical checkpoints, preserved unchanged

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

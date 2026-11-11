# For Claude

Status: MySipur is live on the existing server as an independent app; Cookthis remains healthy.
Blocked on: No hosting blocker. Sipur editorial/source-policy review remains separate from deployment verification.
Proposed next: Keep the paused Cookthis emails and checks paused. Maintain Sipur through its separate source and deployment.

## Current infrastructure change

Cookthis commit `a3116bc` adds only the two MySipur hostname routes to Caddy. The server checkout is synchronized with that commit. The Cookthis web, worker, Redis, database, notification settings and existing jobs were not redeployed or reconfigured.

Sipur runs as a separate Docker Compose app with its own SQLite data, provider environment, resource limits and backup credentials. Its canonical public address is https://mysipur.com; www redirects there. Namecheap DNS and HTTPS are configured.

## Verification

| Check | Measured result |
| --- | --- |
| Cookthis dependency health | HTTP 200, status ok, modelConfigured true |
| Sipur dependency health | HTTP 200, database and story service configured |
| MySipur HTTPS | Valid certificate; public page opened successfully |
| www address | HTTP 301 to canonical HTTPS address |
| Sipur tests | 199 passed, zero failures |
| Public app | Adult story generated; saved stories and prepared vort tested |
| Mobile | 320px and 390px checked; 1280px desktop checked; no horizontal overflow |
| Cloud backup | Uploaded encrypted; downloaded, authenticated, decrypted; SQLite integrity check passed |
| Weekly library | Current Toldos pool has four vorts; following Vayeitzei pool has five |

## Sipur maintenance

Separate system timers prepare the current and following weekly readings at 04:30 Asia/Jerusalem, and back up at 04:00 with up to five minutes jitter. Fourteen daily local backups are kept, with encrypted copies under a separate cloud storage prefix. These are Sipur tasks; no paused Cookthis notification or monitoring task was resumed.

## Known limitations

No new paid comic was generated in this deployment check; existing comic unit tests passed. Story generation and editorial safeguards are inherited from the newer Sipur engine; these checks do not establish rabbinic review or compliance with every original content requirement. The older ChatGPT Sites publication remains separate and was not replaced after its database provisioning failed.

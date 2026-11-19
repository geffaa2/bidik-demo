# Bidik

A membership site for a fictional photography school, built on WordPress and Paid Memberships Pro. It is the working demo behind a freelance service listing
("a WordPress membership site with recurring payments"), so everything in it is real code that does what the listing says, on made-up data.

**What it does:** plans (Free, Member, Pro) with monthly and yearly prices and a 7-day trial; coupons; a checkout with test cards; protected articles, lessons and downloads
(a teaser for visitors, nothing leaking through the page source, RSS, REST or file links); courses that open one module a week; a member account (plan, card, invoices, profile, email choices);
upgrades that show the exact prorated bill first; downgrades at the end of the period; pause and cancel with a short survey and a retention offer; failed payments with a 7-day grace period,
three reminders and recovery; an owner dashboard (MRR, churn, trial conversion, members, CSV export, email log); a CSV importer with a dry run; email-list tags.

**Payments:** the public demo runs on a built-in test gateway that behaves like Stripe's test mode (test cards, events shaped like Stripe's, one code path for checkout, renewals and failures).
The same code also runs against a real Stripe account in test mode: Stripe Checkout (hosted) with the trial, a signed webhook, products/prices/coupons created from code, and Stripe's billing portal for card, plan and cancel.
Tested against real Stripe test clocks (trial, renewal, failing card, recovery, cancel, bad signature, out-of-order events). Not done in Stripe mode: the in-site proration preview and card form (members use the billing portal),
and live-mode keys are refused unless you opt in. Setup: put `BIDIK_STRIPE_SECRET` and `BIDIK_STRIPE_PUBLISHABLE` in `.wp/stripe.env` (never committed), then `node scripts/wp.mjs stripe-setup` and `node scripts/wp.mjs gateway stripe`.

## Run it

Needs Node 20+ and `zip`, `ffmpeg` only for the media scripts.

```
npm install
node scripts/wp.mjs setup      # once: WordPress on Playground (PHP in WebAssembly, SQLite) plus the pinned PMPro build
node scripts/wp.mjs seed       # the school: levels, 40 lessons, 24 articles, the people, 6 months of history
npm run dev                    # http://127.0.0.1:9410
```

Sign in with the buttons in the demo bar (View as), or as the owner at `/wp-admin` with `admin` / `password`. The made-up people all use the password `bidik-demo`:
Dewi (Free), Rani (Member, module 3 opens in 4 days), Bayu (Pro), Maya (a card that failed: seven days to fix it).
Test cards: `4242 4242 4242 4242` works; `4000 0000 0000 9995` is declined; `4000 0000 0000 0341` works now and fails on renewal.

The **Demo clock** in the bar moves time forward 1, 3 or 7 days and runs what would have run: renewals are charged, cards fail, grace periods end, modules open, emails are logged.
**Reset the whole demo** puts the clock, the people and the history back. Demo mode exists only when `BIDIK_DEMO` is set (the dev scripts set it). **Never set it on a real site.**

## Test

```
php tests/run.php                                      # pure logic: 164 checks, no WordPress needed
node scripts/wp.mjs php bidik-tests/integration.php    # inside the site: 279 checks (moves the demo clock; cleans up after itself)
node scripts/e2e.mjs                                   # in a browser against `npm run dev`: 114 checks
node scripts/wp.mjs php bidik-tests/e2e-verify.php     # what the browser run left in the database: 15 checks
node scripts/e2e-demo.mjs                              # demo bar, view as, Demo clock, reset: 12 checks (run it last; the reset removes the test accounts)
node scripts/wp.mjs php bidik-tests/stripe-live.php    # real Stripe test mode (needs keys): 20 checks, cleans up its clocks and users
node scripts/wp.mjs php bidik-tests/cleanup.php        # removes test debris, even from a run that died halfway
```

The tests were checked the other way too: for each rule that matters (a locked article never leaks, a download checks the plan, a renewal is never charged twice, a past-due member keeps access for exactly seven days...),
the rule was broken on purpose and the suite had to fail. Where it did not, a test was added.

## The live demo (WordPress Playground)

```
node scripts/demo.mjs --base-url=https://your-host/bidik
```

builds `dist/` (the pinned PMPro zip, the plugin, the theme and `blueprint.json`). Host those four files together on any static host with CORS enabled and open
`https://playground.wordpress.net/?blueprint-url=https://your-host/bidik/blueprint.json`. The blueprint installs everything, turns on demo mode, builds the school and signs in as Rani.
Tested here with the Playground CLI (the same runtime): the school is ready in about 30 seconds. In a browser the first load builds it in WebAssembly and takes longer; the browser run has not been timed.
Nothing is published by the script.

## Where things are

```
wp-content/plugins/bidik-membership/   the plugin (src/, seed/, assets/)
wp-content/themes/bidik/               the block theme
tests/                                 pure, integration, e2e, fixtures (a 500-row import file)
scripts/                               wp.mjs (site), e2e, demo (Playground build), gallery, video, clips
docs/                                  PRD, tech spec, milestones, screens, design (DESIGN.md), spikes, catalogs/
```

`docs/TECH-SPEC.md` ends with notes on how each milestone was actually built, including what differs from the plan.
`docs/spikes.md` explains why PMPro is bundled (it left WordPress.org) and the small SQLite shim it needs on Playground.

## Credits

Photos: Firmansyah Chandra, Matthew T Rader, Basile Morin and Tobias ToMar Maier, CC BY-SA 4.0, via Wikimedia Commons (`mockups/IMAGES.md`). Icons: Lucide (ISC).
Paid Memberships Pro is GPL; the pinned build is the last one published on WordPress.org.

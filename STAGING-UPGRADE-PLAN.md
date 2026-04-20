# Staging Upgrade Plan — dev.wefixyoursite.com

**Owner:** Kristopher
**Architect:** Claude
**Executor:** Claude Code
**Baseline:** s19 recon, 2026-04-20
**Goal:** Staging is a production mirror that's measurably better than prod, provably safe to upgrade to, and ready to be the template for prod's own upgrade.

---

## What "better than prod" means, concretely

Prod today: WP 5.0.4, WC 3.6.2, PHP 7.4.33, 100+ plugins from 2019-era, debug.log bleed dormant (latent), no WAF, unverified checksums, 65 theme overrides fighting modern WC.

Staging target: WP 6.9.1, WC 10.5.2, PHP 8.1.34, plugin stack on current versions with deprecations resolved, WAF wired, debug.log quiet, caching operational, checkout end-to-end tested, documented.

When this is done, prod's upgrade path is: "Clone staging onto prod's box, swap docroots." Not "figure out what breaks."

---

## Baseline findings worth naming

**Good:**
- WP + WC are both on current versions already
- 0 plugin updates pending
- PHP 8.1 running cleanly, no fatals in last hour
- Uploads dir is clean (only legit wpo_wcpdf files)
- Disk: 4.4 GB docroot on 20 TB, not even a rounding error
- FPM config is reasonable (8 GB uploads, 10k input vars — custom tuned for the pricing table)

**Bad:**
- `debug.log` is **148 MB / 334K lines** — not as bad as prod's 43 GB but actively bleeding. ~190K PHP Deprecated, ~120K Notice, ~15K Warning.
- **4 orphan Requests library files** fail core checksum (`wp-includes/Requests/Transport/fsockopen.php`, `cURL.php`, `IDNAEncoder.php`, `SSL.php`). These are WP 6.1-era files left behind when WP moved to the new Requests library. Harmless but they break verification.
- **`functions.php:6131` emits 130,690 deprecations** — 40% of all log volume from one line. This is in WP core but it's being *called* by something. Needs tracing.
- **amp plugin emits 30,492 deprecations** (9% of volume) — plugin is version 2.0.9 network-activated, current is 2.5+. Needs update or removal.
- **Quform, Yoast Premium, WC PDF Invoices, Ultimate_VC_Addons** each emit 10K+ deprecations — all legacy plugins on older versions.
- **WP_DEBUG is ON in staging** (`WP_DEBUG=true, WP_DEBUG_LOG=true, WP_DEBUG_DISPLAY=false`) — correct for staging, this is why we can see deprecations
- **No object-cache, no advanced-cache** — wp-optimize-premium is the active caching plugin but hasn't written drop-ins. Caching is effectively off.
- **88 child theme WC overrides** + a `BACKUP~` theme directory still present (byte-for-byte duplicate of current theme). Plus a `print-left-navigation-3-2-2021` (82 files, stale from 2021).
- **DB is 934 MB** with `postmeta` at 265 MB / 1M rows, `wcch_page_history` at 90 MB, `aioseo_links_suggestions` at 53 MB. A lot of that is bloat.
- **3,686 users** — 3,150 wholesale, 6 admins. Normal for a B2B store but worth noting.
- **Wordfence is installed but not configured** (WAF not wired, license conflict, stale 2019 rule cache).

**Open questions:**
- `wc-special-product-pricing` on staging is v1.1 with PHP 8 null-safety hardening from Feb 24 — NOT the Shape A patch we applied to prod yesterday. Staging still has the dead `$_SESSION` code.
- `wp wc status` command doesn't exist in this WP-CLI version — can't get WC health snapshot that way. Need a different path.

---

## Multi-session plan

Six sessions. Each is self-contained, reversible, and ends with staging in a better state than it started. If you stop after any session, you still win.

### Session 20 — Hygiene & stabilization (30–60 min)

**Goal:** Clean up the easy wins. Nothing structurally changes. Get staging quiet.

1. Remove the 4 orphan `Requests/` files (checksum fix, ~1 min, zero risk — they're unused by modern WP)
2. Delete `print-left-navigation-BACKUP~` and `print-left-navigation-3-2-2021` theme dirs (stale, not active)
3. Truncate `debug.log` (148 MB → 0)
4. Apply the BL-001 Shape A patch to staging's copy of `wc-special-product-pricing` (mirror what we did on prod yesterday)
5. Finish Wordfence config: Reset License → Free tier → run Optimize Firewall wizard → verify `.user.ini` points at staging paths, not prod
6. Post-session: confirm staging still 200s, debug.log growth rate dropped

**Exit criteria:** `wp core verify-checksums` clean, debug.log < 10 MB/day growth, Wordfence WAF active.

### Session 21 — Plugin deprecation cascade (1–2 hours)

**Goal:** Kill the top 80% of debug.log volume by updating or replacing the worst offenders.

Targets in order of deprecation volume:

1. **AMP plugin** (30K deprecations, 9% of volume) — v2.0.9 → current, or remove entirely if not used
2. **Quform** (43K deprecations across 4 lines) — v2.13.1, check for updates
3. **Yoast Premium** (~41K deprecations, Seo + Seo-Local + Seo-Woo) — v11.4, way behind current
4. **WC PDF Invoices** (32K across 3 lines) — v2.3.5, check for updates
5. **Ultimate_VC_Addons** (10K deprecations) — v3.18.0, page builder addon, likely abandoned
6. **Perfmatters, Redux Framework, the-grid, js_composer** — audit each

For each: pull current version from wp.org, check PHP 8.1 compatibility, update or flag-for-removal. Deactivate-test-reactivate pattern for anything risky.

**Exit criteria:** debug.log growth under 1 MB/day, top 5 deprecation sources eliminated.

### Session 22 — DB bloat cleanup (30–60 min)

**Goal:** Reclaim space without breaking anything.

1. `wcch_page_history` (90 MB) — WooCommerce Customer History plugin data. Purge records > 6 months.
2. `ualp_user_activity_detail` (86 MB) + `ualp_user_activity` (36 MB) — Ultimate Points plugin. Purge records > 12 months.
3. `aioseo_links_suggestions` (53 MB) — orphan table from a plugin not currently active (`aioseo` not in plugin list). Drop table or verify with AIO SEO docs.
4. `ewwwio_images` (50 MB) — image optimizer cache table. Safe to truncate, plugin will rebuild.
5. `postmeta` (265 MB) — run WC's own orphan-meta cleanup: `wp wc cleanup orphaned_products` etc.
6. Expired transients purge.

**Exit criteria:** DB under 500 MB, no orphan tables.

### Session 23 — Theme overrides audit (2–3 hours, potentially 2 sessions)

**Goal:** Identify which of the 88 WC template overrides are still needed and which can be deleted.

This is the real work. An override exists in the child theme for one of these reasons:
- (a) Intentional customization that still matters
- (b) Customization that mattered 5 years ago, doesn't now
- (c) Copy-pasted from a WC version 5+ years ago and is now a liability (breaks features added since)

Method: diff each override against the current WC version's template. If identical → delete (child theme inherits). If near-identical with tiny customization → patch the customization onto current template. If heavily customized → document and keep.

**Exit criteria:** Override count reduced from 88 to realistic baseline, every remaining override has a documented reason.

### Session 24 — Caching, performance, checkout testing (1–2 hours)

**Goal:** Make staging fast, then exercise the hard paths.

1. Configure wp-optimize-premium properly (or install WP Super Cache as alternative — matches YL's setup)
2. Measure TTFB before/after
3. Walk the full checkout as logged-out + wholesale + admin — Stripe, Shop as Customer, SMS, PDF invoice, order emails
4. Document any breakage

**Exit criteria:** TTFB under 200ms cached, checkout works end-to-end for all three user types.

### Session 25 — Final audit + prod upgrade readiness (1 hour)

**Goal:** Can we copy staging onto prod?

1. Re-run the recon script from s19, compare against baseline
2. Write prod upgrade runbook based on what we learned
3. Document delta: "staging has these things prod doesn't, prod has these things staging doesn't"
4. Decide: in-place upgrade on prod (risky, real data), or blue-green deploy (safer, more work)

**Exit criteria:** A prod upgrade runbook that you would actually run.

---

## Rules for every session

1. **Backup before any destructive change.** `cp -a` to `/root/session-N-backup/`.
2. **One session = one major topic.** No scope creep. If something urgent surfaces, write it down and move on.
3. **Read-only recon first, action second.** Always diff expected state against reality before writing.
4. **Changes documented at session end** — session file, backlog updates, gotchas, artifacts index.
5. **If prod is at risk during the session, STOP.** Staging work shouldn't touch prod. If it does, investigate.
6. **Don't over-gate.** Staging is reversible. Batch where it makes sense.

---

## What this plan deliberately doesn't do

- **Doesn't rebuild from scratch.** The Next.js/Postgres rebuild track (BL-022) is a separate, parallel decision. This plan is "make the current WP stack shippable."
- **Doesn't touch prod.** Every change here is on staging only, until Session 25.
- **Doesn't solve every backlog item.** BL-005 (prod admin password), BL-006 (prod wordfence-waf.php), BL-007 (FPM env audit), BL-010 (export token), BL-011 (uploads PHP block), BL-016 (PHP 7.4 hardening) all remain prod-side and get picked up after staging is done.
- **Doesn't fix yl.deadlinesigns.com.** That's BL-003 original scope and a different stack. Covered elsewhere.

---

## Estimated total: 6-10 hours across 6 sessions

Spread across however many days makes sense. No individual session > 3 hours.

Next action: start Session 20 when ready.

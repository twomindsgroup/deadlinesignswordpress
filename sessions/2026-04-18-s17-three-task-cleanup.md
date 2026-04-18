# Session 17 — 2026-04-17/18 — Three-task cleanup: orphan PHP + super-admin hygiene + staging-yl diagnosis

## Context

Following s16's 43 GB prod `debug.log` investigation, three standing items were queued:
1. Orphan PHP cleanup
2. Super-admin hygiene
3. Staging-yl's actively-growing 15 GB `debug.log`

Worked all three in one session with CC as executor.

## Tasks

1. Orphan PHP cleanup
2. Super-admin hygiene (11 → 4)
3. Staging-yl log diagnosis

## Task 1 — Orphan PHP cleanup: COMPLETE

Moved 7 files to `/root/orphan-cleanup-20260417/` (reversible for 30 days). Originals removed from webroot:

- `cloner.php` (0 bytes, orphan)
- `export-urls.php` (leaked public URL inventory)
- `Old_files/test/php/test.php`
- `twilio/test.php` (embarrassing Twilio test-magic creds)
- Two `toodles.php` copies at `/wp-content/uploads/2017/03/` and `/wp-content/uploads/sites/4/2017/03/` (byte-identical phpinfo stubs — 2017-era compromise recon artifact)
- `info.php` (19-byte phpinfo stub, added mid-session after S5 surfaced it)

### info.php leakix exposure (info disclosure, not compromise)

- File existed on current server since 2024-07-22 (mtime suggests older, possibly from prior server migration).
- Removed 2026-04-18 03:02 UTC.
- Minimum exposure duration: 8 months 27 days on current server.
- 20 confirmed leakix scrapes in last 30-day log window (earliest: 2026-03-23 12:23 UTC from 207.154.212.47).
- No credential leak — WP `define()`s aren't env vars, don't appear in phpinfo; FPM pool `env[]` clean on PHP 7.4.
- Did leak: PHP 7.4.33 version fingerprint, full paths, `open_basedir=none`, `disable_functions=opcache_get_status` (effectively no hardening), full extension list, Plesk install layout.
- Severity: Low-Medium info disclosure. No forced DB password rotation; defense-in-depth audit queued.

### Files kept at webroot (confirmed legitimate)

- `timeclock-proxy.php` — Kris's Google Apps Script proxy for time-clock kiosk
- `packing.php` — Kris's WP-CLI-only packing utility
- `export-2025-lineitems.php` — Kris's WC order CSV exporter, token-gated with weak `ds2025export` (rotate before next use — BL-010)

### Files left in place, out of scope

- `wordfence-waf.php` (430 B) — may be wired via `auto_prepend_file`; deletion without audit would 500 every PHP request. Separate task (BL-006).

## Task 2 — Super-admin hygiene: COMPLETE

Network super-admin count: 11 → 4. Final set is exactly staff: `kris, chantel, carlie, megan`.

### Per-user outcomes

- **tringo91, tuannguyen, farmclashofclan** — stale sitemeta entries from s09 deletion, no user rows in DB. Removed from `site_admins` serialized array via SQL (WP-CLI `site-meta` command hit the "table not installed" multisite quirk; direct SQL worked).
- **brookeurena (ID 4392)** — legitimate former team member (brooke@2minds.biz), 447 design attachments, active Oct 2020 → Aug 2025 doing sign artwork, dormant 8+ months. Account preserved, super-admin stripped only. The reported "55 active sessions" was a misread — all 55 tokens expired Aug 2025.
- **Tuan (ID 613)** — originally classified as "Wayne, legacy dev, delete outright." S3 product dump revealed Tuan authored **45 active catalog products with 4,588 lifetime orders**, all still selling daily. Plan revised: keep account, strip super-admin, strip blog-4 administrator role, preserve `tax_exempt` flag and blog-1 `wholesale` role. **Most important correction of this session** — name-pattern matching for classification is not sufficient on a store with 14 years of history.
- **dmytro (ID 628)** — `system-admin.work` freelance-marketplace email, admin on both blogs, 3 posts worth of content. Deleted with `--reassign=1 --network`, content moved to kris.
- **freelancer2 (ID 645)** — keyboard-mash email (`dfasf@home.net`), 13 posts of content. Deleted with `--reassign=1 --network`.

### Content impact

Total content reassignment: 16 rows moved to user ID 1 (kris) — 15 on blog 1, 1 revision on blog 4. Shop orders unchanged at 5,773 (no WC data disturbed).

Post-change storefront verification: Tuan's product 8122 (Adhesive Decal) still renders. `/?p=8122` → 301 → canonical pretty URL → 200.

## Task 3 — Staging-yl debug.log: DIAGNOSIS ONLY (fix queued)

Log was 15.26 GB, growing ~220 MB/day.

**Root cause is NOT the prod session_start bug class** — initial assumption was wrong. Actual causes:

- **woo-discount-rules-pro v2.4.5** — ~22% of log volume. PHP 8.2 dynamic-property deprecations on `WDRPro::$config` (218,100 hits in sampled 1M lines).
- **wc-paypal-payments v1.9.2** — "Translation loaded too early" warnings (14,544 hits).
- **Redis Object Cache plugin active, Redis daemon unreachable** — full stack trace on every pageload from `wp-content/object-cache.php:712`.
- **wp-smush-pro, payment-gateways-by-user-roles-for-woocommerce, woondershop-pt theme** — dynamic-property deprecations.
- **PHP 8.2 `${var}` interpolation deprecation** — 76,356 hits across multiple plugins.

Staging-yl was upgraded to PHP 8.2 in s08; plugin versions on staging-yl predate PHP 8.2 compatibility. This is a known-class problem, not an emergency. 220 MB/day on a 20 TB partition is not urgent.

Fix plan queued as BL-003 — NOT executed this session.

## Artifacts

On-server, 30-day retention:

- `/root/orphan-cleanup-20260417/` — 7 files
- `/root/superadmin-hygiene-20260418/` — user JSONs + sitemeta snapshots (before/after serialized)

## New gotchas

- G-05 (`wp super-admin remove` alone is not a hygiene action)
- G-14 (PHP 8.2 deprecation cascade signatures)
- G-25 (Uploads directory phpinfo stubs — the 2017 toodles.php pattern)
- G-27 (Name-pattern classification is insufficient for user deletion)
- G-28 ("Same bug class" ≠ "same plugin")

## Backlog delta

Added:
- BL-003 (Staging-yl plugin update batch)
- BL-006 (wordfence-waf.php audit before removal)
- BL-007 (FPM env[] directives audit across all PHP pools)
- BL-010 (export-2025-lineitems.php token rotation)
- BL-011 (Uploads-directory PHP execution block)
- BL-016 (PHP 7.4 hardening)

Closed:
- Orphan PHP cleanup
- Super-admin hygiene
- Staging-yl log diagnosis (fix queued as BL-003)

## Corrections and lessons

1. **Name-pattern classification is insufficient.** Tuan was nearly deleted based on "legacy dev" pattern match. 45 active products with 4,588 lifetime orders invalidated that. Product/content check before destructive action is non-negotiable going forward.

2. **CC's initial plan proposed using `wp super-admin remove` alone.** That only strips network-admin bit, leaves user active as per-site admin. Full hygiene requires the sitemeta remove → demote (or delete with `--reassign=1`) path.

3. **"Same bug class" doesn't mean same plugin.** Assumed staging-yl's log bleed was `wc-special-product-pricing`. Wrong — entirely different stack, different plugins, different DB. YL and main store share an origin but not a codebase.

4. **CC attempted to write to `docs/ARCHIVE-SESSIONS.md` and `docs/TRACKER.md`** — these don't exist in this project. CC's project context has drifted or is confusing this with cs-dashboard. Worth verifying next session (noted in BL-012).

## Open threads

- BL-001 (plugin patch) still blocked on BL-004 (main-store staging unblock).
- BL-002 (inode watch on prod wp-config) — continue passive monitoring.
- BL-003 (staging-yl plugin batch) — needs a proper maintenance-window session.
- This session also produced the documentation restructure (README, ENVIRONMENT, BACKLOG, GOTCHAS, PREFERENCES, sessions/) to replace the legacy single-file `CLAUDE.md`.

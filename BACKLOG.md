# BACKLOG

**Last updated:** 2026-04-18 (Session 17)

Living document. Edit in place as items move. Sessions close out against this file.

**Status keys:** `open` · `in-progress` · `blocked` · `done` · `wontfix`
**Priority:** `P0` (right now) · `P1` (next session) · `P2` (soon) · `P3` (someday)

---

## P0 — Active or next-session

### BL-001 · Plugin patch: `$_SESSION` → `WC()->session` in `wc-special-product-pricing`
- **Status:** blocked
- **Blocker:** needs main-store staging unblocked first (BL-004)
- **Why it matters:** Root cause of prod's 43 GB debug.log. If anything flips `WP_DEBUG=true`, the bleed resumes within minutes.
- **Scope:** 7 call sites across `inc/functions.php` and `inc/ajax.php`. Plugin already uses `WC()->session` in `inc/add-fee.php:18-25`, so migration aligns existing pattern. Eliminates `session_start()` entirely.
- **Validation plan:** Staging-first. Confirm session warnings stop AND guest pricing flow still works before prod deploy.
- **References:** sessions/s16, sessions/s17

### BL-002 · Inode watch on prod `wp-config.php`
- **Status:** open (standing)
- **Why:** Session 16 surfaced an unattributed wp-config replacement at 18:31 UTC on Apr 17. Actor not identified. If it recurs, that's the escalation signal.
- **Action:** Periodic `stat` on `/var/www/vhosts/deadlinesigns.com/httpdocs/wp-config.php`. If Birth/Modify timestamp changes without an authenticated action, re-open investigation.
- **References:** sessions/s16

---

## P1 — Soon

### BL-003 · Staging-yl plugin update batch
- **Status:** open
- **Why:** staging-yl's `debug.log` is 15.26 GB growing ~220 MB/day. Root cause is PHP 8.2 deprecation cascade from outdated plugins, NOT the prod session_start issue.
- **Plugins to update:** `woo-discount-rules-pro v2.4.5` (~22% of volume), `wc-paypal-payments v1.9.2`, `wp-smush-pro`, `payment-gateways-by-user-roles-for-woocommerce`, `woondershop-pt` theme.
- **Also:** resolve Redis — either provision the daemon or deactivate `redis-cache` plugin (currently failing stack trace on every pageload).
- **After fixes confirmed:** truncate the log.
- **Scope creep available:** full staging-yl plugin audit (67 active, some severely outdated — `jetpack v11.2`, `cf7 v5.6.2`).
- **Needs:** proper maintenance-window session with plugin-by-plugin testing.
- **References:** sessions/s17

### BL-004 · Main store PHP 8.1 staging — SSL vhost blocker
- **Status:** blocked (since Feb 25, 2026)
- **Blocker:** Plesk only generated HTTP vhost on port 7080, no SSL vhost on port 7081. `plesk repair web` didn't fix. Self-signed cert handshake works but vhost still 404s.
- **Unblocks:** BL-001, BL-005, and the whole WP 5.0.4 → current upgrade path.
- **Location:** `development.wefixyoursite.com`
- **References:** sessions/s02

### BL-005 · WordPress admin password + Wordfence install
- **Status:** open
- **Why:** Flagged in Session 9 (Mar 20) after malware discovery, never confirmed complete. Relates to the orphan `wordfence-waf.php` file at webroot.
- **References:** sessions/s09

### BL-006 · `wordfence-waf.php` audit before removal
- **Status:** open
- **Why:** 430-byte file at webroot root. May be wired via `auto_prepend_file` in `.user.ini` or php.ini. Deletion without audit would 500 every PHP request.
- **Action:** Check `.user.ini`, php.ini, and Plesk PHP additional directives for `auto_prepend_file` pointing to this file. If found, remove directive first, then file.
- **References:** sessions/s17

---

## P2 — When there's time

### BL-007 · FPM env[] directives audit across all PHP pools
- **Status:** open
- **Why:** info.php leaked phpinfo for 9 months; PHP 7.4 pool confirmed clean but 8.0/8.1/8.2/8.3 pools not yet audited. If any carry secrets in env[], they were in the public exposure window.
- **Also:** grep webroot for `.htaccess` / `.user.ini` `SetEnv` directives that might put secrets into PHP env.
- **References:** sessions/s17

### BL-008 · Google Business Profile merge
- **Status:** open
- **Why:** Consolidating Broome Sign Company + Two Minds Group → Deadline Signs. Guide delivered Feb 26, 2026. Awaiting Kris to gather Business Profile IDs and submit to Google Support.
- **References:** sessions/s03

### BL-009 · Post-GBP-merge directory cleanup
- **Status:** blocked (on BL-008)
- **Targets:** Yelp, Bing Places, Apple Maps, Facebook, industry directories.
- **References:** sessions/s03

### BL-010 · `export-2025-lineitems.php` token rotation
- **Status:** open
- **Why:** Kris's WC order CSV exporter uses weak token `ds2025export` in plaintext at webroot. Rotate to random string before next use.
- **Better long-term:** move into a plugin, gate with `current_user_can('manage_woocommerce')`, expose as admin-ajax or REST.
- **References:** sessions/s17

### BL-011 · Uploads-directory PHP execution block
- **Status:** open
- **Why:** Two `toodles.php` phpinfo stubs from 2017 were found in `/wp-content/uploads/`. `.htaccess` rule denying `.php` execution inside uploads would have prevented them from running.
- **Action:** add `.htaccess` to `/wp-content/uploads/` with `<FilesMatch "\.php$"> Require all denied </FilesMatch>`.
- **References:** sessions/s17

### BL-012 · cs-dashboard project documentation
- **Status:** open (lives in separate Claude project)
- **Why:** React/TypeScript frontend + PHP backend, SMS/MMS messaging, contact management, call logs, order search merging custom `cs_orders` + legacy WC posts, commitments system. Kept surfacing in this project's sessions but is architecturally separate.
- **Also:** In Session 17, CC tried to write to `docs/ARCHIVE-SESSIONS.md` and `docs/TRACKER.md` paths that don't exist in this project — possible CC context bleed from cs-dashboard project. Worth verifying.
- **References:** sessions/s10, sessions/s14, sessions/s17

---

## P3 — Someday / nice-to-have

### BL-013 · YL remaining plugin audit
- **Status:** open
- **Targets:** `woo-product-slider-pro`, `wp-featherlight`, `woocommerce-services`, `halfdata-green-popups`, `blackhole-pro`, plus 13 plugins with available updates.
- **References:** sessions/s08

### BL-014 · Quform PHP session issue on YL
- **Status:** open
- **Why:** Blocks WP Super Cache's PHP-layer cache generation on YL.
- **References:** sessions/s08

### BL-015 · Remove CF7 from YL
- **Status:** open
- **Why:** Quform is the active form builder. CF7 was accidentally restored during Session 8 cleanup; proper removal still pending.
- **References:** sessions/s08

### BL-016 · PHP 7.4 hardening
- **Status:** open (part of eventual PHP 8.x upgrade)
- **Why:** `open_basedir=none`, `disable_functions=opcache_get_status` (effectively empty). Combined with public exposure of this fingerprint via the info.php leak, standing risk.
- **Blocker:** BL-004 (staging unblock) → BL-001 (plugin patch) → full upgrade.
- **References:** sessions/s17

---

## Recently closed (last 5 sessions)

| ID | Title | Closed |
|---|---|---|
| — | Orphan PHP cleanup (7 files) | s17, 2026-04-18 |
| — | Super-admin hygiene (11 → 4) | s17, 2026-04-18 |
| — | Staging-yl log diagnosis | s17, 2026-04-18 |
| — | Prod debug.log emergency brake | s16, 2026-04-17 |
| — | Convenience fee plugin verification | s16, 2026-04-17 |
| — | SMS notifications (Flowroute fix) | s14, 2026-03-25 |
| — | YL upgrade (PHP 8.2, WC 10.6, WP Super Cache) | s07–s08, 2026-03-19/20 |
| — | Custom Stripe gateway (`deadline-stripe-gateway`) | s06, 2026-03-18 |

---

## Rules for this file

1. **P0/P1 items never have more than one sentence of prose per field.** Dump detail into the session file and link back.
2. **Every item has `Status`, `References`, and either `Why` or `Blocker`.** If you can't fill all three, the item isn't real yet.
3. **When an item is closed**, move it to "Recently closed" with a session reference. Don't delete.
4. **Re-prioritize at session start**, not end. Fresh eyes catch drift.

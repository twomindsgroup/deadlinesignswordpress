# BACKLOG

**Last updated:** 2026-04-19 (Session 19)

Living document. Edit in place as items move. Sessions close out against this file.

**Status keys:** `open` · `in-progress` · `blocked` · `done` · `wontfix`
**Priority:** `P0` (right now) · `P1` (next session) · `P2` (soon) · `P3` (someday)

---

## P0 — Active or next-session

### BL-022 · Roadmap decision session
- **Status:** open (new, s18)
- **Why:** Three realistic endpoints exist: (1) finish security track + accept running WP 5.0.4, (2) unblock BL-004 and do incremental WP/WC/PHP upgrade, (3) stand up a fresh site at a parallel subdomain and migrate products/orders/customers via export-import + SQL, then flip DNS. Each has very different work shape and timeline. Every session is currently decided case-by-case because this hasn't been picked.
- **Scope:** 20 min focused conversation. Pick an endpoint. Adjust BACKLOG priorities accordingly.
- **References:** sessions/s18 "Open threads"

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

### BL-017 · Ubuntu security updates (incl. kernel)
- **Status:** open (new, s18)
- **Why:** 13 pending security updates. Kernel 6.8.0-110 vs installed 6.8.0-79. libssl3t64, bind9, ruby among them.
- **Needs:** reboot maintenance window (kernel upgrade requires reboot).
- **References:** sessions/s18 I10

### BL-021 · Wordfence re-install (clean slate)
- **Status:** open (new, s18 — replaces old BL-005 Wordfence scope)
- **Why:** s18 confirmed prior Wordfence install was abandoned half-done; wiring stripped in s18 D2, site now has no WAF. Before reinstalling, need to confirm which Wordfence version supports WP 5.0.4 (current versions may refuse or install in degraded mode).
- **First step:** version compatibility research. Not an auto-execute task.
- **References:** sessions/s09 (original flag), sessions/s18 D2

---

## P2 — When there's time

### BL-007 · FPM env[] directives audit across all PHP pools
- **Status:** partial (s18 verified PHP 7.4 clean; other pools not yet audited)
- **Why:** info.php leaked phpinfo for 9 months; PHP 7.4 pool confirmed clean but 8.0/8.1/8.2/8.3 pools need verification. If any carry secrets in env[], they were in the public exposure window.
- **Also:** grep webroot for `.htaccess` / `.user.ini` `SetEnv` directives that might put secrets into PHP env.
- **References:** sessions/s17, sessions/s18 I1

### BL-009 · Post-GBP-merge directory cleanup
- **Status:** open
- **Targets:** Yelp, Bing Places, Apple Maps, Facebook, industry directories.
- **References:** sessions/s03

### BL-012 · cs-dashboard project documentation
- **Status:** open (lives in separate Claude project)
- **Why:** React/TypeScript frontend + PHP backend, SMS/MMS messaging, contact management, call logs, order search merging custom `cs_orders` + legacy WC posts, commitments system. Kept surfacing in this project's sessions but is architecturally separate.
- **Also:** In Session 17, CC tried to write to `docs/ARCHIVE-SESSIONS.md` and `docs/TRACKER.md` paths that don't exist in this project — possible CC context bleed from cs-dashboard project. Worth verifying.
- **References:** sessions/s10, sessions/s14, sessions/s17

### BL-018 · postgres on 0.0.0.0:5432 publicly exposed
- **Status:** open (new, s18)
- **Why:** Phase 1 I16 surfaced postgres listening on all interfaces. Not related to Deadline Signs (different vhost on same server), but a risk on the server as a whole. Needs investigation: what's using it, can it be bound to localhost.
- **References:** sessions/s18 I16

### BL-019 · SSH authorized_keys audit for root
- **Status:** open (new, s18)
- **Why:** 15 entries in /root/.ssh/authorized_keys, some with generic/ambiguous names ("Generated", "new", "all_key", "bitwarden_all_key"). Audit candidates for pruning.
- **References:** sessions/s18 I11

### BL-020 · MySQL %-host wildcards
- **Status:** open (new, s18)
- **Why:** 17 accounts including `deadlinesigns@%` have `%` host wildcards. MySQL isn't bound publicly so current risk is limited, but credential leak would allow external use.
- **References:** sessions/s18 I13

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
- **Blocker:** BL-001 (plugin patch) → full upgrade — or fresh-rebuild track (BL-022).
- **References:** sessions/s17

---

## Recently closed

| ID | Title | Closed |
|---|---|---|
| BL-001 | Plugin patch: $_SESSION removal in wc-special-product-pricing | s19, 2026-04-19 |
| BL-002 | Inode canary verified live | s19, 2026-04-19 |
| BL-004 | Main store staging SSL (self-resolved) | s19, 2026-04-19 |
| BL-008 | Google Business Profile merge | s19, 2026-04-19 |
| BL-002 | Inode watch on prod wp-config — cron installed, alerts wired | s18, 2026-04-18 |
| BL-005 (partial) | WP admin password rotation for kris | s18, 2026-04-18 |
| BL-006 | wordfence-waf.php audit — confirmed orphan, cleanly removed | s18, 2026-04-18 |
| BL-010 | export-2025-lineitems.php token rotation | s18, 2026-04-18 |
| BL-011 | Uploads PHP execution block | s18, 2026-04-18 |
| — | XMLRPC block (D7, new in s18) | s18, 2026-04-18 |
| — | Orphan PHP cleanup (7 files) | s17, 2026-04-18 |
| — | Super-admin hygiene (11 → 4) | s17, 2026-04-18 |
| — | Staging-yl log diagnosis | s17, 2026-04-18 |
| — | Prod debug.log emergency brake | s16, 2026-04-17 |
| — | Convenience fee plugin verification | s16, 2026-04-17 |
| — | SMS notifications (Flowroute fix) | s14, 2026-03-25 |

---

## Rules for this file

1. **P0/P1 items never have more than one sentence of prose per field.** Dump detail into the session file and link back.
2. **Every item has `Status`, `References`, and either `Why` or `Blocker`.** If you can't fill all three, the item isn't real yet.
3. **When an item is closed**, move it to "Recently closed" with a session reference. Don't delete.
4. **Re-prioritize at session start**, not end. Fresh eyes catch drift.

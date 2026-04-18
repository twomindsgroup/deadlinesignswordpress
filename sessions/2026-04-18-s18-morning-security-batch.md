# Session 18 — 2026-04-18 — Morning security close-out + hardening batch

## Context

Six+ security and hardening items were open across the BACKLOG after s17. Kris was at breakfast, wanted a long autonomous run. Structured the session as Phase 1 (24 read-only intel probes) → gate → Phase 2 (7 destructive steps).

## Tasks

**Phase 1 — intel (read-only):** FPM env[] audit, auto_prepend scan, wordfence-waf inspection, Wordfence plugin state, admin accounts + sessions, wp-config baseline, webroot + uploads PHP inventory, uploads .htaccess, token intel, system patch state, SSH hardening, fail2ban, MySQL users, TLS expiry, disk/inodes, listening sockets, Plesk action log 48h, recent webroot writes, backup state, XMLRPC + REST user-enum, AllowOverride check, mail delivery, cron audit, live traffic snapshot.

**Phase 2 — destructive (closed BL-002, BL-005 partial, BL-006, BL-010, BL-011, plus new D7):**
- D1 — WP admin password rotation for user `kris` (id=5)
- D2 — Wordfence "clean slate": strip auto_prepend_file from .user.ini, move orphan wordfence-waf.php to backup
- D3 — (skipped — no Wordfence plugin to activate, deferred to future install session)
- D4 — Uploads PHP execution block (.htaccess with `<FilesMatch>` deny rule) on both uploads roots
- D5 — Inode watch cron for wp-config.php, hourly, mail alerts to kris@2minds.biz
- D6 — export-2025-lineitems.php token rotated from `ds2025export` to `7310e7511d0abebfc65f839f4dd77f38`
- D7 (added mid-session) — XMLRPC block via webroot .htaccess `<Files xmlrpc.php>Require all denied</Files>`
- Also installed `mailutils` package (prereq for D5's mail alerts)

## Findings

### Phase 1 surprises worth noting

- **Wordfence state was "half-installed and broken."** The `.user.ini` at webroot had `auto_prepend_file` wiring to `wordfence-waf.php`. That file existed (430 B, mtime 2023-04-21) and tried to `require` `wp-content/plugins/wordfence/waf/bootstrap.php` — but the plugin directory no longer existed. So auto_prepend fired on every PHP request, did a `file_exists` check, failed silently, fell through. Functionally a no-op with microsecond overhead. Confirms that the "install Wordfence" task from s09 (March) was never completed — someone started it, got partway through, and abandoned.
- **kris account is user ID 5, not user ID 1** on this install. Worth noting because earlier session docs assumed ID 1.
- **kris had 3 active session tokens** from Cloudflare IPs dated Apr 7, Apr 10, Apr 17 — not all Kris's current-device logins. D1's session destroy killed all of them; Kris re-logs in fresh.
- **XMLRPC was accepting `system.listMethods` with HTTP 200.** Standard brute-force amplification vector on old WP. D7 blocks it entirely.
- **mail command not installed** despite postfix running — `mailutils` package was missing. Fixed mid-session.
- **13 pending Ubuntu security updates** including kernel 6.8.0-110 (installed: 6.8.0-79), libssl3t64, bind9, ruby. Kernel needs reboot window.
- **postgres listening on 0.0.0.0:5432.** Publicly exposed. Not related to Deadline Signs (different vhost on same box), but a risk on the server as a whole.
- **17 MySQL accounts with `%` host wildcards** including `deadlinesigns@%`. MySQL isn't bound publicly so current risk is limited, but credential leak would allow external use.
- **15 root SSH authorized_keys** with some generic-named keys ("Generated", "new", "all_key", "bitwarden_all_key"). Audit candidate.
- **TLS expiry clean** across all four domains.
- **Disk/inodes healthy.**
- **Live traffic snapshot: 801 hits in 2 minutes at 06:47 UTC** — heavy bot/scanner traffic even off-hours. Cloudflare is fronting but not aggressively filtering.
- **recidive fail2ban jail had 64 active bans** — fail2ban is doing its job.

### Incident during D2 (resolved in ~2 minutes, customer impact minimal)

After moving `wordfence-waf.php` to backup, PHP-FPM workers had the old `.user.ini` cached via `user_ini.cache_ttl` (300s default). Workers kept trying to `auto_prepend` the now-missing file, causing intermittent HTTP 500s on a fraction of requests for ~2 minutes. Recovered by `systemctl reload plesk-php74-fpm`.

Most customer traffic during the window was served from Cloudflare's 200 cache; only non-cached requests saw the error.

## Changes made

| Step | What | Path / target | Backup |
|---|---|---|---|
| D1 | kris password + session destroy | WP user id=5 | (passwords not backed up) |
| D2 | .user.ini stripped, wordfence-waf.php moved | webroot + /root/wordfence-clean-20260418/ | .user.ini.pre-d2, wordfence-waf.php |
| D7 | xmlrpc.php deny block appended | webroot .htaccess | /root/htaccess-webroot-backup-20260418/.htaccess.pre-d7 |
| mailutils | `apt install mailutils` | system | n/a |
| D4 | uploads PHP deny block | wp-content/uploads/.htaccess + wp-content/uploads/sites/4/.htaccess | /root/htaccess-uploads-backup-20260418/ |
| D5 | inode watch cron | /usr/local/bin/wp-config-inode-watch.sh, /etc/cron.d/wp-config-watch, /var/lib/wp-config-watch/ | n/a (new install) |
| D6 | export-2025-lineitems.php token | webroot | /root/export-lineitems-backup-20260418/ |

New export URL: `https://deadlinesigns.com/export-2025-lineitems.php?token=7310e7511d0abebfc65f839f4dd77f38`

Baseline recorded by D5: inode 206831785, size 3704, mtime 1776450682 — matches the 2026-04-17 18:31:22 UTC event documented in s16.

## Verification

All green after FPM reload:
- `/` = 200
- `/wp-admin/` = 302
- `/xmlrpc.php` = 403 (GET + POST both denied)
- uploads canary.php = 403
- inode watch baseline written
- mail test queued to kris@2minds.biz (delivery verification pending Kris's inbox check)

## Artifacts

On-server, 30-day retention:

- `/root/wordfence-clean-20260418/` — .user.ini.pre-d2, wordfence-waf.php (original + .moved copy)
- `/root/htaccess-webroot-backup-20260418/.htaccess.pre-d7`
- `/root/htaccess-uploads-backup-20260418/` — existing uploads .htaccess before overwrite
- `/root/export-lineitems-backup-20260418/export-2025-lineitems.php.pre-rotation`

## New gotchas

- **G-30** (new): Removing files referenced by .user.ini auto_prepend_file requires FPM reload FIRST — see GOTCHAS.md.

## Backlog delta

Closed:
- BL-002 (inode watch on prod wp-config) — cron installed, baseline recorded, alerts wired to kris@2minds.biz
- BL-005 partial (WP admin password rotation) — kris rotated; Wordfence install still deferred
- BL-006 (wordfence-waf.php audit before removal) — audited, confirmed orphan, cleanly removed
- BL-010 (export-2025-lineitems.php token rotation) — new token in place
- BL-011 (uploads PHP execution block) — .htaccess deny deployed to both uploads roots

Added:
- BL-017 — Ubuntu security updates (13 pending incl. kernel) — needs reboot maintenance window
- BL-018 — postgres on 0.0.0.0:5432 publicly exposed — investigate whether needed, bind to localhost if not
- BL-019 — 15 root SSH authorized_keys audit — prune generic-named keys
- BL-020 — MySQL %-host wildcards on 17 accounts — tighten hosts
- BL-021 — Wordfence re-install as standalone task — needs WP 5.0.4 compatibility research before execution (replaces old BL-005 remaining scope)

Re-prioritized:
- BL-005 — half-closed; remaining Wordfence install becomes BL-021

## Process notes

- Phase 1's one-big-heredoc approach hit a local shell parse error. CC adapted by splitting into 4 smaller SSH calls in parallel (all read-only, rate-limit safe). Worth remembering: for this server, 4×6 probes is better than 1×24 when there's any risk of quote-handling issues.
- D2 incident is documented as G-30 and as a process lesson: when a file is referenced in .user.ini via auto_prepend, the removal sequence is strip-directive → reload-FPM → move-file, NOT the other order.
- CC proposed sendmail-direct as a fallback if mailutils was unavailable. Went with mailutils install instead (simpler, standard package). If D5's mail alerts ever stop working, switching to sendmail-direct is the documented fallback.

## Open threads

- Mail delivery to kris@2minds.biz not yet user-confirmed. If the `[TEST2]` email didn't arrive within ~10 min of session close, D5 needs rework.
- `/wp-admin/` login with new password not yet user-confirmed. Kris intends to log in fresh.
- Roadmap decision (Track 1 "finish security" vs Track 2 "retire WP 5.0.4 via incremental upgrade" vs fresh-rebuild-and-migrate) still deferred. Worth a 20-min dedicated session to pick an endpoint.

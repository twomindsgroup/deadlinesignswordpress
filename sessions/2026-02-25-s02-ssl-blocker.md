# Session 02 — 2026-02-24/25 — HTTPS 404 blocker on staging (STALLED)

## Context

Continuation of s01. Staging responded over HTTP but browsers kept forcing HTTPS, which returned Apache "Not Found." This became the blocker that halted the entire main-store upgrade project.

## Tasks

1. Diagnose why HTTPS was 404ing on staging
2. Attempt fixes for Plesk SSL vhost
3. Kill HTTPS redirect as fallback

## Findings

- Plesk generated HTTP Apache vhost on port 7080 but NO SSL vhost on port 7081.
- `plesk repair web development.wefixyoursite.com -y` did not fix it.
- Adding a self-signed SSL cert via Plesk UI worked for the TLS handshake but Apache still 404'd because the SSL vhost wasn't routing to WordPress.
- `FORCE_SSL_ADMIN = false` in wp-config didn't help.
- HSTS cache clearing across multiple browsers didn't help.
- `.htaccess` was clean. `wp_options.siteurl` and `home` were correct. nginx was correctly redirecting 443 traffic.

## Changes made

None successful. Multiple attempts, all unresolved.

## Artifacts

- Handoff document: `/mnt/user-data/outputs/HANDOFF-staging-upgrade.md`

## New gotchas

- G-07 (Plesk SSL vhost generation is unreliable)

## Backlog delta

Added: BL-004 (Main store PHP 8.1 staging — SSL vhost blocker). Still blocked as of s17.

## Open threads

SSL vhost on port 7081 never successfully generated. Session hit context limits before resolution. **The entire main-store upgrade project has been stalled on this since Feb 25.**

Also deferred: third batch of PHP 8 compatibility patches covering 80+ WooCommerce template overrides in the child theme.

# Session 07 — 2026-03-19 — YL upgrade begins (full audit + staging setup)

## Context

Kris uploaded WoonderShop PT child theme and new parent theme 4.3.0, asking for help diagnosing slow load times on yl.deadlinesigns.com.

## Tasks

1. Full audit of YL stack
2. Set up staging-yl environment
3. Begin incremental WooCommerce upgrade

## Findings

Five compounding problems identified:

1. PHP 7.4.33 (EOL since 2022)
2. WooCommerce 6.9.4 (current was 10.6)
3. Zero page caching, 2.58s TTFB
4. 12 severely outdated WooCommerce template overrides in child theme
5. Blocking cURL call on the order thank-you page with a 60-second timeout to internal IP (decommissioned server `10.10.2.145`, built by original "Green Apps" devs)

Additional surprises:

- **257 GB debug.log** at `/var/www/vhosts/deadlinesigns.com/yl.deadlinesigns.com/wp-content/debug.log` — `WP_DEBUG` left on in production for years. Truncated with `truncate -s 0`. (First instance of the oversized-debug.log pattern that recurs.)
- **34 GB database** that was 91% expired WooCommerce sessions — cleaned up.

## Changes made

- Staging-yl provisioned at `staging-yl.deadlinesigns.com` on same OVH server.
- Parent theme upgraded to WoonderShop PT 4.3.0.
- PHP switched 7.4 → 8.2 on staging-yl. TTFB dropped 2.58s → 0.60s on that change alone.
- WooCommerce incrementally upgraded through three jumps: 6.9.4 → 7.9 → 8.9 → 9.9.
- 257 GB `debug.log` truncated.
- 34 GB session table cleaned.

## Artifacts

- Handover doc: `/mnt/user-data/outputs/YL-Upgrade-Handover.md`

## New gotchas

- G-01 (WP-CLI full-path invocation)
- G-06 (Plesk PHP version switching syntax)
- G-08 (Plesk site create without hosting)
- G-09 (Let's Encrypt CLI extension requires `-m` email flag)
- G-12 (`FS_METHOD = 'direct'` required in wp-config on Plesk)
- G-22 (DNS at Cloudflare, not Squarespace)

Also established: testing staging without DNS via `curl --resolve staging-yl.deadlinesigns.com:443:148.113.187.163`.

## Backlog delta

Added: YL upgrade workstream (completed in s08).

## Open threads

Session ended mid-upgrade. Jump 4 (WC 9.9 → 10.6) and Phases 4–7 (template updates, caching, dead code cleanup, production push) remained. Continues in s08.

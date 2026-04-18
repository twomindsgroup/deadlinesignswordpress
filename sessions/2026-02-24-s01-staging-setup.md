# Session 01 — 2026-02-24 — Staging site setup (PHP 8 upgrade project begins)

## Context

Main store (deadlinesigns.com) on WordPress 5.0.4 / WC 3.6.2 / PHP 7.4. Trigger for the upgrade project was a specific bug: `wc-special-product-pricing` plugin's pricing table admin page truncating because the form sends ~5,400 POST variables and PHP's `max_input_vars` default was 1000. Chose not to risk production — build full staging first, validate, then push.

## Tasks

1. Provision staging on the same OVH server
2. Clone production files + DB
3. Apply PHP 8 compatibility patches to custom plugins and theme
4. Validate staging is accessible

## Changes made

- Staging provisioned at `development.wefixyoursite.com`
- Production files and DB cloned over
- Initial PHP 8 compatibility patches applied across three batches covering `wc-special-product-pricing`, `dashboard-deadlinesigns`, and the child theme
- Staging `wp-config.php` updated: new DB creds, `WP_HOME`, `WP_SITEURL`, `WP_DEBUG=true`
- Full multisite DB URL replacement via comprehensive SQL script (hit network tables + blog-specific tables + plugin data tables)
- Disabled two plugins that crashed on domain change due to license checks: `ilab-media-tools-premium`, `ajax-login-and-registration-modal-popup-pro`

## Findings

- Multisite DB URL replacement requires hitting every table tier — partial replacement causes redirect-back-to-prod bugs.
- Plugin license systems are the most common source of staging crashes. Disable systematically rather than trying to debug their code.
- Kris's dynamic IP needed adding to Plesk firewall whitelist. First-known-check for SSH failures.

## Artifacts

- Handoff doc written for next session (continued into s02)

## New gotchas

- G-10 (Plesk firewall IP whitelist first on SSH fail)
- G-15 (Multisite URL replacement must cover all table tiers)

## Backlog delta

Added: initial staging upgrade workstream.

## Open threads

Staging site returning wp-admin error on first load. Needed to check debug.log for the crashing plugin before the pricing table bug could even be tested. Continues in s02.

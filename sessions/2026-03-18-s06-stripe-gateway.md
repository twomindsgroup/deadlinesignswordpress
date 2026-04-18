# Session 06 — 2026-03-18 — Pricing table save button + building custom Stripe gateway

## Context

Long technical session with two interleaved workstreams:
1. Pricing table save button had vanished on reseller.deadlinesigns.com admin
2. Official Stripe plugin dropped support for WordPress 5.0.4 — needed replacement payment gateway

## Tasks

### Pricing table

1. Diagnose the truncated admin page
2. Fix the PHP fatal and form-size issues

### Stripe gateway

1. Build from scratch without official Stripe plugin dependency
2. Integrate with custom 5-step multistep checkout

## Findings

### Pricing table

- Root cause: PHP fatal in `/wp-content/plugins/wc-special-product-pricing/inc/backend/backend-options.php` line 138 — `wc_get_product()` returned false for a deleted product and code called `get_name()` on it without checking.
- Secondary: `max_input_vars = 1000` (PHP default) was dropping form fields — raised to 10000 via Plesk additional directives.
- Tertiary: OPcache preventing PHP file changes from taking effect.

### Stripe gateway — implementation problems

- `payment_fields()` rendering twice in the DOM creating duplicate hidden inputs; empty review-step copy overwrote the tokenized one.
- Perfmatters deferring Stripe.js so it wasn't available when inline scripts ran.
- WooCommerce re-rendering payment methods on gateway switch, destroying mounted Stripe Elements.

## Changes made

### Pricing table

- Added `if (!$product) continue;` guard to skip deleted products in `backend-options.php`.
- Raised `max_input_vars` to 10000 via Plesk PHP directives (production).
- Temporarily disabled OPcache via `opcache.enable=0` during debugging.

### Stripe gateway

- Built `deadline-stripe-gateway` plugin from scratch, no SDK dependency, uses `wp_remote_post()` against Stripe Payment Intents API directly.
- `jQuery.ajaxPrefilter` injects payment method ID directly into WC's `wc-ajax=checkout` POST data (bypasses all DOM/hidden-input issues).
- Stripe.js loaded dynamically via `document.createElement('script')` with onload callback (bypasses Perfmatters defer).
- `data-ds-original="1"` attribute distinguishes original card mount from `append_info_review_order()` clones.
- Continuous poll with re-mount logic detects when WC empties the card container after gateway switch.
- CSS in `style.css` hides the payment method display on review step (panel 3).
- Gateway ID: `ds_stripe`.
- Packaged as `deadline-stripe-gateway-final.zip`, network-activated across all three sites.
- Per-site API key config required in each subsite.

## Artifacts

- `deadline-stripe-gateway-final.zip` (deployed)

## New gotchas

- G-11 (OPcache silently preventing PHP changes)
- G-17 (Custom multistep checkouts break standard Stripe integration)

## Backlog delta

No new backlog items. Stripe gateway and pricing table save are both closed.

## Open threads

Pricing table save verification on staging still deferred (blocked on s02 SSL issue).

## Process notes

- Kris pushed back when Claude was overly cautious about a Stripe API key shared in chat — "move forward quickly."
- Established preference: complete ready-to-deploy files, not copy-paste snippets.

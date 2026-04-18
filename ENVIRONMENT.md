# ENVIRONMENT

**Last full refresh:** 2026-04-18 (Session 17)
Each section carries its own `Verified:` date so drift is visible.

---

## Business context

**Verified:** 2026-03-21 (Session 10)

Deadline Signs (formerly Two Minds Group), Concord, NC. 14+ years, 5,000+ orders. Print and sign shop.

Four revenue lines:
- **deadlinesigns.com** — main WooCommerce print store
- **yl.deadlinesigns.com** — yard letters/signs (standalone WP, different DB)
- **twomindsgroup.com** — graphic design, rebranded "Now Deadline Signs"
- **reseller.deadlinesigns.com** — B2B reseller portal (blog 4 on main multisite)

Address: 5-L C Carpenter Ct NW, Concord NC 28027 (shared across Broome Sign Company + TMG + Deadline Signs for GBP merge).

---

## Server

**Verified:** 2026-04-18 (Session 17)

| Item | Value |
|---|---|
| Provider | OVH VPS |
| IP | `148.113.187.163` |
| Control panel | Plesk, Linux |
| Partition | 20 TB on `/var/www` |
| SSH alias | `ovh` (in `~/.ssh/config`) |
| Whitelisted IPs | work `67.62.101.122`, home `136.57.2.111` |

Workflow: KiTTY/PuTTY for interactive, PowerShell for SCP/SSH from Windows, Claude Code for agentic execution.

---

## DNS

**Verified:** 2026-04-17 (Session 16)

| Item | Value |
|---|---|
| Nameservers | `piers.ns.cloudflare.com`, `kami.ns.cloudflare.com` |
| Registered at | Squarespace |
| DNS managed at | **Cloudflare** |

⚠ Domain is registered at Squarespace but DNS lives in Cloudflare. DO NOT add DNS records at Squarespace.

---

## WordPress multisite (main network)

**Verified:** 2026-04-18 (Session 17)

| Item | Value |
|---|---|
| Network table prefix | `4k18Gue_` |
| Blog 1 | `deadlinesigns.com` (main store, prefix `4k18Gue_`) |
| Blog 4 | `reseller.deadlinesigns.com` (prefix `4k18Gue_4_`) |
| WordPress version | 5.0.4 (March 2019) |
| WooCommerce version | 3.6.2 (April 2019) |
| PHP version | 7.4.33 |
| Status | Seven years out of date, upgrade project STALLED (see BACKLOG) |

⚠ `yl.deadlinesigns.com` is NOT part of this multisite — it runs its own standalone WP install.

---

## YL stack (standalone WP install)

**Verified:** 2026-03-20 (Session 8, last major change)

| Item | Value |
|---|---|
| Database | `wp_dleig` |
| DB user | `wp_jmcov` |
| Table prefix | `IbopU_` |
| Staging DB | `wp_dleig_staging` |
| PHP version | 8.2 |
| WooCommerce | 10.6 |
| Parent theme | WoonderShop PT 4.3.0 |
| Caching | WP Super Cache |
| TTFB (cached) | ~0.050s (was 2.58s pre-upgrade) |

---

## Paths (critical)

**Verified:** 2026-04-18 (Session 17)

| What | Path |
|---|---|
| Main store docroot | `/var/www/vhosts/deadlinesigns.com/httpdocs` |
| YL production | `/var/www/vhosts/deadlinesigns.com/yl.deadlinesigns.com` |
| YL staging | `/var/www/vhosts/deadlinesigns.com/staging-yl` |
| Main store PHP 8.1 staging (stalled) | `/var/www/vhosts/wefixyoursite.com/development.wefixyoursite.com` |
| PHP 7.4 binary | `/opt/plesk/php/7.4/bin/php` |
| PHP 8.2 binary | `/opt/plesk/php/8.2/bin/php` |
| WP-CLI binary | `/usr/local/bin/wp` |

**WP-CLI always-use-full-path invocation:**
```bash
/opt/plesk/php/8.2/bin/php /usr/local/bin/wp [cmd] --allow-root --path=[docroot]
# or for PHP 7.4 sites:
/opt/plesk/php/7.4/bin/php /usr/local/bin/wp [cmd] --allow-root --path=[docroot]
```

Bare `wp` picks up the wrong system PHP. See GOTCHAS #wp-cli.

---

## Key custom plugins (main store)

**Verified:** 2026-04-18 (Session 17)

| Plugin | Role | Status |
|---|---|---|
| `wc-special-product-pricing` | Pricing engine + Flowroute SMS + embedded convenience-fee logic | Active, 8,832 lines, built by prior devs |
| `dashboard-deadlinesigns` | Custom admin dashboard | Active |
| `deadline-stripe-gateway` | Custom Stripe Payment Intents gateway (ID: `ds_stripe`) | Built Mar 2026, no SDK dependency |
| `deadline-signs-convenience-fee` | 3.1% card fee on `ds_stripe` orders | Network-active since Mar 24, 2026 |
| `woocommerce-shop-as-customer` | cxThemes v2.16.1, abandoned upstream | Patched locally for multisite super-admin check |

---

## Themes

**Verified:** 2026-04-17 (Session 16)

| Site | Theme |
|---|---|
| Main store | `print-left-navigation` child of `printshop` (Netbase) |
| YL | WoonderShop PT child (jungle skin, originally GreenApps) |
| Main store page builders | WPBakery + SiteOrigin |
| Main store nav | MegaMenu |

Main store custom feature: 5-step multistep checkout (defining feature, hostile to standard Stripe patterns — see `deadline-stripe-gateway` history in sessions s06, s11).

---

## Payment + SMS

**Verified:** 2026-03-25 (Session 14, last Flowroute work)

| Item | Value |
|---|---|
| Stripe gateway ID | `ds_stripe` |
| Stripe integration | Payment Intents API via `wp_remote_post` (no SDK) |
| Convenience fee | 3.1% on card (`ds_stripe`) orders only, network-active plugin |
| SMS provider | Flowroute (access key `23832001`) |
| SMS code location | `wc-special-product-pricing/sms.php` |

Dead code NOT to reintroduce: RingCentral (commented out), Twilio SMS plugin (superseded), `2mind.us` short links via Linkity (abandoned self-hosted shortener).

---

## Super admins (network)

**Verified:** 2026-04-18 (Session 17)

Exactly 4 accounts. Any drift is a signal.

- `kris`
- `chantel`
- `carlie`
- `megan`

Any other entry in `site_admins` sitemeta is suspicious and should trigger an audit.

---

## Related server artifacts

See `artifacts/INDEX.md` for the full index of backup dirs, handoff docs, zipped plugins, and rollback SQL files.

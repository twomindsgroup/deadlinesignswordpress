# GOTCHAS

Accumulated landmines. Tagged by subsystem so you can grep.

**Tags:** `#wp-cli` `#plesk` `#php` `#multisite` `#woocommerce` `#stripe` `#ssh` `#dns` `#caching` `#debugging` `#security` `#process`

---

## `#wp-cli`

### G-01 · WP-CLI full-path invocation required
Bare `wp` picks up the wrong system PHP and throws MySQL extension errors. Always use:
```bash
/opt/plesk/php/8.2/bin/php /usr/local/bin/wp [cmd] --allow-root --path=[docroot]
# or /opt/plesk/php/7.4/bin/php for PHP 7.4 sites
```
**Source:** s07

### G-02 · `wp site-meta get` multisite quirk
`wp site-meta get 1 site_admins` errors with "The table is not installed" in some multisite contexts. Fall back to direct SQL:
```bash
wp db query "SELECT meta_value FROM 4k18Gue_sitemeta WHERE meta_key='site_admins'"
```
**Source:** s17

### G-03 · `wp plugin list --status=active` misses network-activated plugins
Only shows per-site activations, not network-level. Use `--status=active-network` or query `wp site-meta get 1 active_sitewide_plugins` directly.
**Source:** s16

### G-04 · `wp user list --network --role=administrator` ≠ super-admin list
That command returns all users, not super admins. Use SQL on `sitemeta WHERE meta_key='site_admins'` for the authoritative list.
**Source:** s16

### G-05 · `wp super-admin remove` alone is not a hygiene action
It only strips the network-admin bit. Account still exists with whatever per-site roles it had. Full hygiene = content audit → sitemeta remove + per-blog role change (demote) OR delete with `--reassign=1 --network`.
**Source:** s17

### G-37 · `wp core verify-checksums | tail -N` masks orphan count
The command emits one warning per orphan file followed by an `Error: …` summary line. Tailing the last few lines catches the summary but hides the warnings, making it look like a small problem when it isn't. The s19 recon used `tail -5` and reported 4 orphans; full enumeration in s20 surfaced ~150 (a 37× under-count). For orphan enumeration, capture full output and `grep -c "^Warning"` for the count, or pipe to `wc -l`.
**Source:** s20

---

## `#plesk`

### G-06 · Plesk PHP version switching syntax
Only this command works: `plesk bin domain --update [domain] -php_handler_id [handler]`. Variants like `plesk bin site --update-hosting` or `plesk bin hosting --setup` return "unknown command."
**Source:** s07

### G-07 · Plesk SSL vhost generation is unreliable
Plesk sometimes only generates HTTP vhost on port 7080, no SSL vhost on port 7081. `plesk repair web [domain] -y` does not always fix it. This is the blocker on the main-store staging (see BL-004).
**Source:** s02

### G-08 · Plesk site create without hosting
`plesk bin site --create [domain] -webspace-name [parent] -www-root /[path]` creates a subdomain but without web hosting configured. Hosting must then be set up in the Plesk UI.
**Source:** s07

### G-09 · Let's Encrypt CLI extension requires `-m [email]` flag
Without it, throws a null email TypeError. Also requires hosting configured first.
**Source:** s07

### G-10 · Plesk firewall IP whitelist is the first check when SSH fails
Kris's dynamic IP needs to be on the whitelist. Home `136.57.2.111` and work `67.62.101.122` are standing whitelisted entries.
**Source:** s01

---

## `#php`

### G-11 · OPcache silently prevents PHP file changes from taking effect
During debugging, temporarily disable via `opcache.enable=0` in Plesk additional directives.
**Source:** s06

### G-12 · `FS_METHOD = 'direct'` required in wp-config on Plesk
Without it, WP-CLI plugin installs throw FTP filesystem errors.
**Source:** s07

### G-13 · `session_status()` check is NOT a substitute for `!headers_sent()`
They're orthogonal. Both are needed to guard `session_start()` safely. Missing this is the root cause of the prod debug.log spam (see BL-001).
**Source:** s16

### G-14 · PHP 8.2 deprecation cascade signatures
When a site upgrades PHP to 8.2 and plugins aren't updated, expect:
- Dynamic-property deprecations (classes without `#[AllowDynamicProperties]`)
- `${var}` string-interpolation deprecated in favor of `{$var}`
- `filter_input()` null parameter deprecations
- WooCommerce "Translation loaded too early" for plugins calling `load_plugin_textdomain` after init
**Source:** s17

### G-30 · Removing files referenced by `.user.ini` `auto_prepend_file` requires FPM reload FIRST
`.user.ini` directives are cached by PHP-FPM workers via `user_ini.cache_ttl` (default 300s). If you strip a directive AND move the file it referenced in the same breath, workers keep trying to `auto_prepend` the now-missing file for up to 5 minutes → intermittent HTTP 500s on a fraction of requests. Correct order:
1. Strip the directive from `.user.ini`
2. `systemctl reload plesk-php<N>-fpm` (flushes worker cache)
3. THEN move or delete the file

Fixed this in s18 by reloading FPM after the fact. Customer impact was limited because Cloudflare was serving most traffic from its 200 cache, but it's real and the order matters.
**Source:** s18

### G-33 · `wc -l` vs PHP `file()` disagree by 1 on files without trailing newline
`wc -l` counts `\n` characters; PHP's `file()` returns one array element per line even when the last line lacks a terminator. A legacy file ending `}` or `?>` with no trailing newline gives `wc -l = 145` but `count(file($path)) = 146`. Anchor-based patch scripts that verify line counts against `wc -l` will abort incorrectly on these files. Use the `file()` count as the source of truth inside PHP patch scripts.
**Source:** s19

---

## `#multisite`

### G-15 · DB URL replacement must cover all table tiers
Partial replacement = redirect-to-prod bug. Hit network tables + blog-specific tables + plugin data tables.
**Source:** s01

### G-16 · Super-admin capability checks need explicit multisite branches
Standard role checks fail for super admins. Pattern:
```php
if (is_multisite() && current_user_can('manage_network')) return true;
```
Shop as Customer plugin has this bug and is patched locally.
**Source:** s09

---

## `#woocommerce`

### G-17 · Custom multistep checkouts break standard Stripe integration
`payment_fields()` renders twice, duplicate hidden inputs collide, WC re-renders on gateway switch destroying mounted Elements. Clean fix: `jQuery.ajaxPrefilter` injects payment method ID directly into WC's `wc-ajax=checkout` POST data. See `deadline-stripe-gateway` history.
**Source:** s06

### G-18 · CSS Hero static file persists after plugin removal
`/wp-content/uploads/2021/11/csshero-static-style-woondershop-pt-child.css` (563 lines) contains critical visual overrides — header, nav, footer, fonts, logo. After removing CSS Hero, enqueue this file directly from child theme's `functions.php`.
**Source:** s08

### G-19 · WP Super Cache activation via WP-CLI is incomplete
WP-CLI activation doesn't run the admin UI setup routine. You must manually inject `.htaccess` mod_rewrite rules and generate `wp-cache-config.php`. Also check for leftover `advanced-cache.php` from prior caching plugins (WP Rocket, etc.).
**Source:** s08

### G-35 · `wp_ajax_nopriv_` registration ≠ working feature
A `wp_ajax_nopriv_<action>` hook is reachable by anonymous users, but the registration alone is not evidence the feature works. In `wc-special-product-pricing`, `list_media` and `modal_media` register `nopriv` endpoints that query a `_token_auto_delete` postmeta key no code anywhere writes — feature has been broken since circa 2020. Before refactoring an AJAX endpoint, grep for readers/writers of the data it depends on.
**Source:** s19

---

## `#ssh`

### G-20 · SSH rate limiting on OVH
Batch commands in single heredoc SSH calls. Keep per-step connection count ≤ 2 (e.g., 1 scp upload + 1 ssh exec).
**Source:** s08

### G-21 · Bash special characters break terminal commands
Especially `!` in passwords used inside commands. Avoid them in anything you'll paste into a shell. Safe: letters, digits, `@ # - _ + . , = /`. Dangerous: `! $ \` " ' * ? [ ] & | ; < > ( ) space`.
**Source:** s09

---

## `#dns`

### G-22 · DNS at Cloudflare, NOT Squarespace
Domain is registered at Squarespace but nameservers point to Cloudflare. Any DNS record changes go at Cloudflare. Changes made at Squarespace are ignored.
**Source:** s07

---

## `#debugging`

### G-23 · `is_user_logged_in()` in hooks silently blocks automation
Any hook guarded by this check will silently skip execution during cron, webhooks, gateway callbacks, or any context where no user is logged in. Recurring gotcha in legacy custom code from prior devs (TriNgo, Wayne, GreenApps).
**Source:** s14

### G-24 · `wp-config.php` inode change as canary
Stat `wp-config.php` periodically. If Birth/Modify timestamp changes without an authenticated action, it's a signal for investigation. Inode preservation under in-place edits is normal; inode change = full-file replacement = suspect. As of s18 this is automated: `/usr/local/bin/wp-config-inode-watch.sh` runs hourly, emails kris@2minds.biz on change.
**Source:** s16, automated in s18

---

## `#security`

### G-25 · Uploads directory phpinfo stubs
Identical-byte, identical-hash `toodles.php` copies in `/wp-content/uploads/2017/03/` and `/wp-content/uploads/sites/4/2017/03/` are fingerprints of a 2017-era compromise recon probe. As of s18 this is mitigated: `.htaccess` in both uploads roots denies PHP execution.
**Source:** s17, mitigated in s18

### G-26 · phpinfo() does NOT leak WordPress `define()` constants
`define('DB_PASSWORD', ...)` in wp-config is a PHP constant, not an env var. Does not appear in phpinfo output. phpinfo still leaks plenty (paths, PHP version, extensions, pool config, open_basedir status, disable_functions list) but credentials via `define()` are safe from that specific vector. FPM pool `env[]` directives are NOT safe — audit those separately.
**Source:** s17

### G-31 · `mailutils` package required for `mail` CLI on Plesk boxes
Postfix running ≠ `mail` command available. Plesk provides postfix and sendmail but not the `mail` / `mailx` wrapper by default. If cron alerts or any shell script needs to pipe through `mail`, install with `apt install mailutils` first. Alternative: pipe through `sendmail -t` directly with explicit headers.
**Source:** s18

---

## `#process`

### G-27 · Name-pattern classification is insufficient for user deletion
On a 14-year-old store, a "legacy dev name" can own active catalog, customer-facing attachments, or real orders. ALWAYS run a content/order audit before destructive action on any user. Tuan was nearly deleted in s17 based on name match; turned out to own 45 active products with 4,588 lifetime orders.
**Source:** s17

### G-28 · "Same bug class" ≠ "same plugin"
Prod and YL share origin but not codebase. Don't assume a symptom that looks similar (oversized debug.log) has the same root cause across stacks. In s17, prod's debug.log was a `session_start` issue in `wc-special-product-pricing`; YL's same-sized log was a PHP 8.2 deprecation cascade in entirely different plugins.
**Source:** s17

### G-29 · Stripe API key caution threshold
Kris explicitly deprioritizes caution on live-store Stripe keys shared in chat. Don't slow momentum over it.
**Source:** s06

### G-32 · One-big-heredoc SSH can hit local shell parse errors
When a single heredoc contains many nested quote styles (PHP `-r` inline scripts, bash `<<'SCRIPT'` nested heredocs, sed quotes, awk scripts), the local shell can mis-match quotes before it even ships to SSH. Symptom: "unexpected EOF" or parser errors on the client side. Workaround: split into 4–6 smaller SSH calls with logical grouping. For read-only probes, these can run in parallel safely; for writes, keep them serial. CC handled this autonomously in s18.
**Source:** s18

### G-34 · `grep -c` exits 1 when match count is zero
Under `set -euo pipefail`, a `grep -c pattern file` that finds zero matches returns 1 and kills the script — even when "zero matches" is the desired post-condition after a cleanup patch. Wrap post-condition greps with `|| true`, or bracket the verification block with `set +e` / `set -e`.
**Source:** s19

### G-36 · Backlog entries can describe the wrong shape of fix
A backlog entry written from initial triage ("migrate `$_SESSION` to `WC()->session`") can prescribe a mechanical refactor for code that is actually dead. Before executing the prescribed shape, verify the code being refactored is live: grep for readers of every write, callers of every function, writers of every meta key being queried. If a feature doesn't have a complete round-trip, deletion is cheaper than migration. BL-001 was reshaped from migration to deletion on this basis.
**Source:** s19

### G-38 · Recursive `find -delete` with `src/` preservation needs explicit predicate
`find . -type f ! -path "./src/*" -delete` only excludes files directly in the top-level `src/`. It does NOT exclude files in nested directories that happen to match the exclusion pattern at deeper levels — e.g. when a tree has both `Foo/src/` (canonical) AND `Foo/library/Foo/Bar.php` (also canonical per checksum manifest), the find sweeps the second. In s20, the SimplePie cleanup deleted `SimplePie/library/SimplePie/*` files that WP 6.9.1 expects to exist; site stayed up because SimplePie isn't on the shop hot path, but it required restore from backup. When preserving canonical files that don't live under `/src/`, enumerate orphans first (`grep "^Warning: File should not exist"`) and delete the explicit list — don't trust pattern exclusion.
**Source:** s20

### G-39 · Staging gate discipline
Confirmation gates exist for prod destructive writes. On staging, when a backup has already been taken at session start, gating every step turns a 15-minute install into a 3-hour flow. The s20 Wordfence install was budgeted at ~10 minutes and ran ~3 hours because every step (move stale file, edit `.user.ini`, install plugin, network-activate) was gated separately. Rule: **batch by risk profile** — one gate at the start of a risk class (e.g. "we are about to write to webroot"), one verify at the end. Treat staging like staging, not like prod-with-extra-steps.
**Source:** s20

---

## How to add to this file

When a session surfaces a new landmine:
1. Pick the most specific subsystem tag, or add a new one.
2. Assign the next `G-NN` number (global numbering).
3. One-paragraph statement of the landmine + its fix.
4. `**Source:** s<session-number>` at the bottom so the full context is traceable.

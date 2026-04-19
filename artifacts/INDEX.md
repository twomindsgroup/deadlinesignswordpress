# Artifacts index

Where things live on the server. On-server paths only; not files in this repo.

Cleaned up periodically. Check dates — some of these may be gone.

---

## Active backups

### `/root/wcspp-pre-bl001-20260419/`
**Created:** 2026-04-19 (s19)
**Retention:** 7 days (delete after ~Apr 26, 2026) if prod stable
**Contents:** Full pre-patch copy of `wc-special-product-pricing` plugin (6.8 MB, `cp -a`). Backed up before BL-001 Shape A patch.
**Rollback command:**
```bash
ssh ovh 'rm -rf /var/www/vhosts/deadlinesigns.com/httpdocs/wp-content/plugins/wc-special-product-pricing && cp -a /root/wcspp-pre-bl001-20260419 /var/www/vhosts/deadlinesigns.com/httpdocs/wp-content/plugins/wc-special-product-pricing && systemctl reload plesk-php74-fpm'
```

### `/root/orphan-cleanup-20260417/`
**Created:** 2026-04-17/18 (s17)
**Retention:** 30 days (delete after ~May 18, 2026)
**Contents:** 7 files moved from webroot during orphan cleanup — `cloner.php`, `export-urls.php`, `Old_files/test/php/test.php`, `twilio/test.php`, two `toodles.php` copies, `info.php`.
**Rollback command:**
```bash
ssh ovh 'bash -se' <<'REMOTE'
set -euo pipefail
cd /root/orphan-cleanup-20260417
find . -type f | while read -r f; do
  rel="${f#./}"
  mkdir -p "/var/www/vhosts/deadlinesigns.com/httpdocs/$(dirname "$rel")"
  mv -v "$f" "/var/www/vhosts/deadlinesigns.com/httpdocs/$(dirname "$rel")/"
done
REMOTE
```

### `/root/superadmin-hygiene-20260418/`
**Created:** 2026-04-18 (s17)
**Retention:** indefinite (small files, keep for audit trail)
**Contents:**
- `user-Tuan-613.json`, `user-brookeurena-4392.json`, `user-dmytro-628.json`, `user-freelancer2-645.json` (user record JSONs)
- `usermeta-<user>-<id>.json` (usermeta JSONs for same four)
- `site_admins-before.serialized` — pre-change `site_admins` sitemeta value (224 B)
- `site_admins-after.serialized` — post-change value (74 B)
- `site_admins-before-row.tsv` — full DB row for restore
**Rollback sitemeta only:**
```sql
UPDATE 4k18Gue_sitemeta
SET meta_value='<paste site_admins-before.serialized contents>'
WHERE meta_key='site_admins';
```
**Rollback deleted users:** not cleanly reversible without full DB restore. User JSONs preserved for reference but content reassignment to kris doesn't snap back.

### `/root/wordfence-clean-20260418/`
**Created:** 2026-04-18 (s18, D2)
**Retention:** 30 days (delete after ~May 18, 2026) unless Wordfence reinstall happens first
**Contents:**
- `.user.ini.pre-d2` — webroot .user.ini before auto_prepend_file wiring was stripped
- `wordfence-waf.php` — original file
- `wordfence-waf.php.moved` — post-move copy (belt and suspenders)
**Rollback:**
```bash
# Restore .user.ini
cp /root/wordfence-clean-20260418/.user.ini.pre-d2 /var/www/vhosts/deadlinesigns.com/httpdocs/.user.ini
# Restore wordfence-waf.php
cp /root/wordfence-clean-20260418/wordfence-waf.php /var/www/vhosts/deadlinesigns.com/httpdocs/
# REQUIRED: flush FPM worker cache
systemctl reload plesk-php74-fpm
```

### `/root/htaccess-webroot-backup-20260418/.htaccess.pre-d7`
**Created:** 2026-04-18 (s18, D7)
**Retention:** 30 days
**Contents:** webroot .htaccess before the xmlrpc.php `<Files>` deny block was appended.
**Rollback:** `cp /root/htaccess-webroot-backup-20260418/.htaccess.pre-d7 /var/www/vhosts/deadlinesigns.com/httpdocs/.htaccess`

### `/root/htaccess-uploads-backup-20260418/`
**Created:** 2026-04-18 (s18, D4)
**Retention:** 30 days
**Contents:** pre-existing uploads .htaccess files (if any existed) before overwrite with the PHP-deny rule.
**Rollback:** copy whichever `.htaccess.bak` file corresponds to the uploads dir back into place.

### `/root/export-lineitems-backup-20260418/export-2025-lineitems.php.pre-rotation`
**Created:** 2026-04-18 (s18, D6)
**Retention:** 30 days
**Contents:** export-2025-lineitems.php with original `ds2025export` token, before rotation.
**Rollback:** `cp /root/export-lineitems-backup-20260418/export-2025-lineitems.php.pre-rotation /var/www/vhosts/deadlinesigns.com/httpdocs/export-2025-lineitems.php`

### `/root/yl_prod_backup_parent_theme/`
**Created:** 2026-03-19/20 (s08)
**Retention:** indefinite (YL rollback)

### `/root/yl_prod_backup_child_theme/`
**Created:** 2026-03-19/20 (s08)

### `/root/yl_prod_backup_woocommerce/`
**Created:** 2026-03-19/20 (s08)

### `/root/yl_prod_backup_wp-config.php`
**Created:** 2026-03-19/20 (s08)

### `/root/yl_production_pre_phase7.sql`
**Created:** 2026-03-19/20 (s08)
**Purpose:** Full YL production DB snapshot before Phase 7 (production push).

### `/root/yl_staging_pre_wc79.sql`
**Created:** 2026-03-19/20 (s08)
**Purpose:** Staging DB snapshot before WC 7.9 jump.

---

## Active system installs from sessions

### `/usr/local/bin/wp-config-inode-watch.sh`
**Created:** 2026-04-18 (s18, D5)
**Purpose:** Hourly check of wp-config.php inode + mtime + size. Emails kris@2minds.biz if any change, includes diff + recent access log hits.
**Related:** `/etc/cron.d/wp-config-watch` (cron entry), `/var/lib/wp-config-watch/baseline.txt` (current baseline), `/var/lib/wp-config-watch/history.log` (change log).
**Removal (if ever needed):**
```bash
rm /etc/cron.d/wp-config-watch /usr/local/bin/wp-config-inode-watch.sh
rm -rf /var/lib/wp-config-watch
```

---

## Plugin zips / deployable bundles

### `deadline-stripe-gateway-final.zip`
**Created:** 2026-03-18 (s06)
**Purpose:** Custom Stripe Payment Intents gateway, no SDK dependency.
**Deployed:** network-activated across all three sites on main multisite.
**Location on server:** wherever Kris keeps plugin masters (not centralized in `/root`).

---

## Handoff docs (from session outputs)

These lived in `/mnt/user-data/outputs/` at time of writing — ephemeral path, not guaranteed to persist.

| File | From session | Purpose |
|---|---|---|
| `HANDOFF-staging-upgrade.md` | s02 | SSL vhost blocker details |
| `YL-Upgrade-Handover.md` | s07 | YL upgrade mid-session state |
| `kristopher-working-reference.md` | s15 | Original working-rules reference (superseded by PREFERENCES.md) |

---

## Cleaned-up (no longer on disk)

From s16 investigation:
- `wp-config.php.bak-20260417-211040` (removed)
- `/tmp/cc-wpconfig-ts` (removed)
- `/root/debug-log-tail-100k.txt` (removed)

---

## Audit documents

### `artifacts/FULL-SITE-AUDIT.md`
**Created:** 2026-04-18 (s19)
**Purpose:** Complete site audit and rebuild specification for deadlinesigns.com → Next.js + PostgreSQL migration. Covers all 9 product types, pricing engine, checkout flow, payment gateways, order management, reseller portal, integrations, data migration plan, and recommended architecture.

### `artifacts/THEME-WOOCOMMERCE-AUDIT.md`
**Created:** 2026-04-18 (s19)
**Purpose:** Detailed theme and WooCommerce template override audit. 65 template overrides, checkout flow implementation, email templates, shortcodes.

---

## How to add to this file

When a session creates a server-side artifact (backup dir, zipped plugin, dumped SQL, handoff doc):

1. Add a section with the full path as the heading.
2. Note **Created** date + session reference.
3. Note **Retention** — is this kept forever, or is it on a delete-by date?
4. Describe **Contents** in enough detail that someone could tell whether it's what they need.
5. Include a **Rollback command** if the artifact exists for rollback purposes.
6. When deleted, move to "Cleaned-up" section with date.

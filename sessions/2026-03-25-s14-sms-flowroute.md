# Session 14 — 2026-03-25 — SMS notifications (Flowroute fix)

## Context

Kris noticed order status SMS notifications weren't consistently firing. "I don't remember where the developer built that."

## Tasks

1. Trace the SMS system to its actual source
2. Identify why it fires inconsistently
3. Fix and deploy

## Findings

Tracing path through conversation history:
1. First looked at WooCommerce Twilio SMS Notifications plugin — not the active system.
2. Then at RingCentral code — commented out, dead.
3. Eventually found the real active system in `wc-special-product-pricing/sms.php`.

Key clue that led to the right place: the `2mind.us` short domain in SMS content was a custom URL shortener tied to Two Minds Group (confirmed custom-built, abandoned self-hosted "Linkity" shortener), not a third-party plugin.

Real provider: **Flowroute**, access key `23832001`, confirmed active.

**Two root causes identified:**

1. **`is_user_logged_in()` check at the top of `flowroute_send_sms()`** — blocked automated status changes (cron, webhooks, gateway callbacks) because no user is logged in during those. SMS fired when Kris manually changed status, but not automatically. This is why the issue looked intermittent.

2. **`_receive_sms` opt-in meta was never being saved** because the checkout opt-in checkbox was broken. DB query confirmed no orders ever had `_receive_sms` set — the entire opt-in gate had been failing silently for an unknown period.

## Changes made

Kris chose to:
- Remove opt-in requirement entirely. Send to all customers with a valid US phone.
- Remove `2mind.us` short links from messages entirely rather than replace the abandoned shortener.

Final deployed `sms.php` changes:

- Removed `is_user_logged_in()` check
- Removed opt-in requirement
- Added null checks on order and phone number
- Added US phone number validation
- Added deduplication via WordPress transients
- Switched to `json_encode` for the API payload
- Set 10-second curl timeout
- Added success and error logging with order numbers
- Removed all dead checkout checkbox code
- Deployed via SSH heredoc directly

## Artifacts

None preserved — deployed in place.

## New gotchas

- G-23 (`is_user_logged_in()` in hooks silently blocks automation)

## Backlog delta

Closed: SMS notifications workstream.

## Open threads

Kris pasted a Claude Code commit summary about `cs-dashboard` REST API permission callback fixes during this session — unrelated to SMS work, flagged for clarification. cs-dashboard is tracked separately (BL-012).

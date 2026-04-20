# Sessions index

One file per session. Append-only. Filename convention: `YYYY-MM-DD-sNN-short-title.md`.

| # | Date | Title | File |
|---|---|---|---|
| 01 | 2026-02-24 | Staging site setup (PHP 8 upgrade project begins) | [2026-02-24-s01-staging-setup.md](2026-02-24-s01-staging-setup.md) |
| 02 | 2026-02-24/25 | HTTPS 404 blocker on staging (STALLED) | [2026-02-25-s02-ssl-blocker.md](2026-02-25-s02-ssl-blocker.md) |
| 03 | 2026-02-26 | Google Business Profile merge planning + TMG rebrand | [2026-02-26-s03-gbp-merge-plan.md](2026-02-26-s03-gbp-merge-plan.md) |
| 04 | 2026-03-02 | Unrelated Threads post review (no work product) | [2026-03-02-s04-threads-review.md](2026-03-02-s04-threads-review.md) |
| 05 | 2026-03-15 | Status recap session | [2026-03-15-s05-status-recap.md](2026-03-15-s05-status-recap.md) |
| 06 | 2026-03-18 | Pricing table save + custom Stripe gateway build | [2026-03-18-s06-stripe-gateway.md](2026-03-18-s06-stripe-gateway.md) |
| 07 | 2026-03-19 | YL upgrade begins (audit + staging) | [2026-03-19-s07-yl-upgrade-start.md](2026-03-19-s07-yl-upgrade-start.md) |
| 08 | 2026-03-19/20 | YL upgrade completion | [2026-03-20-s08-yl-upgrade-complete.md](2026-03-20-s08-yl-upgrade-complete.md) |
| 09 | 2026-03-20 | Deadlinesigns.com checkout + malware discovery | [2026-03-20-s09-checkout-malware.md](2026-03-20-s09-checkout-malware.md) |
| 10 | 2026-03-21 | Executive business brief + audit passes | [2026-03-21-s10-exec-brief.md](2026-03-21-s10-exec-brief.md) |
| 11 | 2026-03-24 | Stripe gateway history recap | [2026-03-24-s11-stripe-recap.md](2026-03-24-s11-stripe-recap.md) |
| 12 | 2026-03-24 | Convenience fee plugin debugging | [2026-03-24-s12-convenience-fee.md](2026-03-24-s12-convenience-fee.md) |
| 13 | 2026-03-25 | Site loading false alarm | [2026-03-25-s13-loading-false-alarm.md](2026-03-25-s13-loading-false-alarm.md) |
| 14 | 2026-03-25 | SMS notifications (Flowroute fix) | [2026-03-25-s14-sms-flowroute.md](2026-03-25-s14-sms-flowroute.md) |
| 15 | 2026-04-17 | Reference doc creation | [2026-04-17-s15-reference-doc.md](2026-04-17-s15-reference-doc.md) |
| 16 | 2026-04-17 | Convenience fee verification + 43 GB debug.log investigation | [2026-04-17-s16-debug-log-43gb.md](2026-04-17-s16-debug-log-43gb.md) |
| 17 | 2026-04-17/18 | Three-task cleanup: orphan PHP + super-admin hygiene + staging-yl diagnosis | [2026-04-18-s17-three-task-cleanup.md](2026-04-18-s17-three-task-cleanup.md) |
| 18 | 2026-04-18 | Morning security close-out + hardening batch (BL-002/005/006/010/011 + XMLRPC) | [2026-04-18-s18-morning-security-batch.md](2026-04-18-s18-morning-security-batch.md) |
| 19 | 2026-04-19 | BL-001 patch + four-item backlog sweep | [2026-04-19-s19-bl001-patch.md](2026-04-19-s19-bl001-patch.md) |
| 20 | 2026-04-20 | Staging hygiene pass | [2026-04-20-s20-staging-hygiene.md](2026-04-20-s20-staging-hygiene.md) |

---

## Template for a new session

Copy this into a new file when opening a session close-out:

```markdown
# Session NN — YYYY-MM-DD — <short title>

## Context
<Why this session happened. Prior state. What triggered it.>

## Tasks
<Numbered list of workstreams this session covered.>

## Findings
<Things discovered — bugs, surprises, corrections to prior assumptions.>

## Changes made
<What we actually wrote, deleted, reconfigured. Include paths.>

## Artifacts
<On-server backup dirs, zipped plugins, handoff docs. Cross-ref artifacts/INDEX.md.>

## New gotchas
<Anything that should be appended to GOTCHAS.md.>

## Backlog delta
Added:
- BL-XXX — …
Closed:
- BL-XXX — …
Reprioritized:
- …

## Open threads
<What wasn't finished, what's blocked, what's queued for next time.>
```

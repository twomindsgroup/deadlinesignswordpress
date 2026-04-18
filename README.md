# Deadline Signs — Project Docs

**Owner:** Kristopher Daia
**Architect:** Claude (web)
**Executor:** Claude Code (local on Windows)
**Started:** Feb 24, 2026
**Repo convention date:** Apr 18, 2026 (Session 17 restructure)

---

## What is this?

Living documentation for the full technical stewardship of Deadline Signs infrastructure — WordPress multisite network, WooCommerce stores, OVH/Plesk server, custom plugins, brand consolidation.

Replaces the legacy single-file `CLAUDE.md`. Same content, split so the parts that churn (backlog, gotchas) aren't buried inside parts that are append-only (session history).

---

## Read-first for any new Claude/CC session

1. **[ENVIRONMENT.md](ENVIRONMENT.md)** — Stack, paths, server, how to run WP-CLI here.
2. **[BACKLOG.md](BACKLOG.md)** — What's next. Living document. Edit in place.
3. **[GOTCHAS.md](GOTCHAS.md)** — Accumulated landmines, indexed by subsystem.
4. **[PREFERENCES.md](PREFERENCES.md)** — How Kris, Claude, and CC work together.

Those four files answer 90% of "what do I need to know to be useful right now?" Everything else is reference.

---

## Rest of the tree

```
deadline-signs/
├── README.md                 ← you are here
├── ENVIRONMENT.md            ← stack, paths, server facts
├── BACKLOG.md                ← open items, prioritized
├── GOTCHAS.md                ← tagged landmines
├── PREFERENCES.md            ← working style & rules
├── sessions/                 ← one file per session, append-only
│   ├── INDEX.md              ← session list with titles + dates
│   ├── 2026-02-24-s01-staging-setup.md
│   ├── …
│   └── 2026-04-18-s17-three-task-cleanup.md
├── runbooks/                 ← reusable procedures (populate as needed)
│   └── (empty — build over time from session patterns)
├── architecture/             ← per-site technical detail
│   └── (empty — build over time)
└── artifacts/
    └── INDEX.md              ← on-server backups, zips, handoff docs
```

---

## Document conventions

- Dates are UTC unless noted.
- **Main store** = deadlinesigns.com. **YL** = yl.deadlinesigns.com. **TMG** = twomindsgroup.com. **Reseller** = reseller.deadlinesigns.com.
- Session filenames: `YYYY-MM-DD-sNN-short-title.md`.
- Append-only files: `sessions/*`. Everything else is edited in place.
- If any file contradicts memory, the file wins for historical fact. Memory wins for current standing state.

---

## How to close out a session (end-of-session checklist)

1. Write `sessions/YYYY-MM-DD-sNN-<title>.md` using the template at the top of any existing session file.
2. Add a one-line entry to `sessions/INDEX.md`.
3. Update `BACKLOG.md`: mark closed items done, add new items surfaced, re-order if priorities shifted.
4. If new landmines were found, add them to `GOTCHAS.md` with subsystem tags.
5. If any new server artifact was created (backup dir, zip, handoff doc), add it to `artifacts/INDEX.md`.
6. If `ENVIRONMENT.md` drifted (new path, new version, something moved), update the affected line and refresh its "last verified" date.

That's the discipline. If it's not followed, this folder rots back into a single-file mess within 3 sessions.

# Session 15 — 2026-04-17 (early morning) — Reference doc creation

## Context

Kris returned after a ~3-week gap. Asked for a scan of past chats to build a reference markdown that prevents repeating mistakes.

## Tasks

1. Scan conversation history across multiple angles
2. Produce working-reference doc

## Changes made

Produced `/mnt/user-data/outputs/kristopher-working-reference.md` with:

- Setup overview (who does what — the Kris/Claude/CC triad)
- Non-negotiable working rules (complete files, no copy-paste, bash special chars, WP-CLI full path, etc.)

Also discussed workflow patterns for running diagnostic + overnight CC prompts safely:

- Sequential vs parallel execution
- Threshold-based autonomy
- `nohup`/`tmux` for SSH disconnect resilience

## Findings

Key principle reinforced in this session: Kris prefers big batched autonomous runs over one-at-a-time hand-offs, with CC prompts written for incremental reporting so partial progress survives a crash.

## Artifacts

- `/mnt/user-data/outputs/kristopher-working-reference.md`

## Backlog delta

None. This was documentation/reflection work.

## Open threads

None specific — this session was a reset.

## Note

This session is the forerunner to the current PREFERENCES.md file. The working rules surfaced here became formalized in PREFERENCES.md during the s17 restructure.

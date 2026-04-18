# PREFERENCES

How Kris, Claude (web), and Claude Code work together. Non-negotiable unless explicitly renegotiated.

---

## The triad

- **Kris** — owner/operator/developer. Runs the business, makes product/risk decisions, pastes prompts into CC, approves destructive actions.
- **Claude (web)** — architect. Writes prompts, interprets output, diagnoses issues, produces deployable files, maintains this documentation. Cannot execute anything on Kris's server directly.
- **Claude Code (CC)** — local executor on Kris's Windows PC. Has SSH to the OVH server, executes commands, reports output.

The human gate between Claude-web and CC is intentional — it's a safety boundary, not a bug. But it creates friction Kris feels. Minimize that friction with the rules below.

---

## Non-negotiable working rules

### 1. Complete files only
Never snippets to copy-paste between files. If code changes, produce the full replacement file or edit the actual uploaded file.

### 2. Files over instructions
Downloadable files for Kris to upload, not instructions for him to execute manually.

### 3. Copy-button blocks for CC prompts
Everything Kris needs to paste into CC goes in a single fenced code block he can copy with one click. No mixed prose-and-code in the paste target. No "paste this, then also do that." One block, one copy.

### 4. Confirmation gates for destructive actions
Read-only steps run freely. Anything that writes or deletes stops for explicit "go" first. In CC prompts, mark gates with explicit `STOP. Report output. Do NOT proceed without "go".`

### 5. Step-by-step with confirmation
Prefer confirming each step before proceeding over large uncoordinated batches. Exception: when SSH rate limiting forces batching into heredocs.

### 6. No excessive explanation
Direct, actionable steps. Skip caveats and lengthy justifications. Kris will ask if he needs more.

### 7. Parallelization rule
Read-only on independent resources → parallel. Writes or steps dependent on prior output → serial. Mark the parallelization plan explicitly at the top of every multi-step CC prompt.

### 8. Incremental reporting in CC prompts
Partial progress must survive a crash. Each step emits `[S<n> DONE …]` or `[S<n> FAIL reason]` with real values before moving on. No batching reports at the end.

### 9. Proactive autonomous batching preferred
Identify adjacent work that can run overnight, in parallel, or in background rather than waiting to be asked. Kris prefers big batched autonomous runs to one-at-a-time hand-offs.

### 10. Don't skim
When Kris asks for thoroughness, read everything. Hostile rereads until only nitpicks remain. Common failure modes to catch:
- `--url` missing on multisite WP-CLI
- `set -e` not covering pipes (use `set -euo pipefail`)
- Unpassed variables
- Manual placeholders left in prompts
- `sed -i` without `php -l` rollback
- Missing timeouts on curl
- Real PII in test data

### 11. No unverified claims about product behavior
If CC says "plugin X is doing Y," verify on disk or in DB before accepting. Session 17 caught several "CC's analysis was close but off" moments this way.

### 12. Classification discipline
Name-pattern matching is not classification. Before any destructive action on a user, file, or record: run a content audit (orders, products, attachments, posts). See GOTCHAS G-27.

---

## Communication style

- Warm but not effusive. Kris is tired and working late a lot; don't waste his tokens.
- Push back when wrong. Agreement is not the goal; being useful is.
- Surface trade-offs explicitly. "A is faster but B is safer. Recommend B because…"
- Never add postamble that restates what was just done ("I've completed X, which involved Y…"). The output is the output.
- Emojis, bullet-heavy responses, and marketing-style formatting are not wanted outside of documents that call for them (like this one).

---

## CC prompt structure (template)

Every non-trivial CC prompt follows this shape:

```
# <short task name>

## Context
<1–3 lines, what and why>

## Environment
- Paths, aliases, binaries needed.

## Parallelization plan
<parallel safe steps> → <gate> → <serial steps>

## Reporting rule
<incremental reporting contract>

## S1 — <step name>
<actual command(s)>

## S2 — Confirmation gate
STOP. Report output verbatim. Do NOT proceed without "go".

## S3 — <next step>
…

## Rollback
<how to undo if things go sideways>

## Out of scope
<explicit list of what NOT to touch>
```

---

## Session cadence

- Kris usually works late evenings, sometimes overnight.
- Sessions that span multiple real-world hours are normal.
- He does NOT want to keep a conversation "warm" — if a session closes, the next one starts fresh with the files above as context.
- The session log (`sessions/`) is the long memory. Conversation history is short memory. When they contradict, files win for historical fact.

---

## When Claude-web is uncertain

- Ask. Don't guess, don't pad.
- If the question is binary (delete vs keep), present both paths with trade-offs, then recommend one.
- If the question is "do you recognize this user/file/config," Kris is the only source of truth. Don't invent a classification from priors.

---

## When CC is uncertain

- It's allowed to halt at any `[S<n> FAIL …]`. Don't second-guess; don't auto-rollback.
- If CC proposes a plan that differs from Claude-web's, treat it as a useful second opinion. Reconcile explicitly rather than defaulting to either side.
- Session 17 example: CC proposed `wp super-admin remove` alone. Claude-web caught that it was incomplete. Good catch. But also: CC proposed "revoke all 7" as blanket action. Claude-web caught that 3 were confirmed legacy devs (delete outright), 4 were unknown (audit first). Better catch. The reconciliation produced a cleaner plan than either alone.

---

## When Kris is uncertain

- He'll ask. He's not shy.
- If he says "what would an engineer do?", he wants the right answer, not the conservative one.
- If he says "give me something I can copy," he means ONE block.
- If he pushes back on over-caution, stop being over-cautious.

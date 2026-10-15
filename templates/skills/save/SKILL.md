---
name: save
description: Save state. Check inbound, write one memory row per distinct thought, promote durable facts to the KB, rewrite this session's state file, sweep stale files, leave handoffs, and commit code if any. The chat continues afterwards. Trigger on '/save', 'save state', 'checkpoint', 'pause here', 'wrap up', 'done for now', or any sign the operator wants state preserved.
---

# /save: bank the session

Run every step in order. **The chat continues afterwards.** Paths and verbs are the same as `/start` (folder or MCP backend).

## 0. Inbound first

Read your role's `inbox/` again (leader: every tenant's leader inbox + the cross-tenant log). Factor anything new into this save. If this session started more than a day ago, re-read `/memories/leader/config/operating-contract.md`; it wins over what you loaded at start.

Before acting on or tombstoning an inbox item, check it still exists. If a sibling already removed it, they claimed it.

## 1. Memory rows: one per distinct thought

`create` on the role's store, usually 5–15 per productive session:

| Kind | Path |
|---|---|
| decision | `/memories/<role>/decisions/<YYYY-MM-DD>-<slug>.md` |
| session summary | `/memories/<role>/sessions/<YYYY-MM-DD>-<slug>.md` |
| lesson | `/memories/<role>/lessons/<YYYY-MM-DD>-<slug>.md` |
| open thread | `/memories/<role>/notes/<YYYY-MM-DD>-<slug>.md` |

Short, dated, about one thing, with the why. **Role-less sessions** classify each row by content and announce it (step 9). When torn between two roles, file to the more specific one and link the other.

**Data wall:** customer data is saved in full in the tenant that owns it, never in another tenant's rows and never in a KB. Route it; never drop it. Everything else is your knowledge and is saved normally. No secrets anywhere.

## 2. KB promotion

Did anything durable emerge, something true in six months and useful to any session? If yes: `create` or `str_replace` `/kb/<section>/<slug>.md` (frontmatter: title, domain, stream, type, source, updated), add or update its line in `/kb/<section>/INDEX.md`, and bump the root index only if you added a section. Reference tickets or rows for any customer specifics; never copy them. If nothing durable emerged, skip. No filler.

## 3. State file (leader, ticket and role sessions)

Rewrite `/memories/<role>/state/<your-key>.md`. Your own file only.

```
## Lane          (concurrent leaders) lane · repos held · DO-NOT list
## Now           ≤5 lines
## In flight     background jobs, scratch paths, PRs awaiting review, half-applied changes (omit if none)
## Agents        full agent id · specialty · what it holds (if you hold live workers)
## Open threads  pointers to rows
## Resume hint   1–2 lines
```

Contract: **2KB cap. Rewrite the whole body, don't append. Pointers, not prose.** If it won't fit, bank more rows and point at them. Roamer sessions write no state file unless the operator asks to keep the thread (then `state/chat-<topic>.md`).

At the last save of a long day, ask each live agent for a carry-forward note and bank it as a lesson row. Instances die with the session; types persist.

## 4. Embed and link (if you have them)

One semantic-ingest call with every row, state file and KB entry this save wrote. Then link the obvious graph edges (ticket keys mentioned, KB paths referenced, section membership) plus any relationship you actually learned. On failure, note it and continue; a backfill reconciles.

## 5. Hygiene (mechanical)

In your own role's store, at most ~20 tombstones per save:

- your role's `state/code-*` files not written in 36h (no judgment: silence is death),
- your own handoffs older than 30 days,
- leader only: processed leader-inbox items older than 7 days,
- inbox items you acted on this session.

## 6. Code

If code changed: feature branch off an **asserted** up-to-date main, explicit pathspecs, push with an explicit refspec, verify with `git ls-remote`, open the PR with an explicit head branch. Merge it yourself when verified and not gated (gates: data loss, production on the work side, permission widening, draft/hold). Never merge red, never force-push main. Dispatched workers never run git.

## 7. Handoffs

- To another role: `create /memories/<recipient>/inbox/<from>-<date>-<slug>.md` on the recipient's store.
- Leader-relevant outcome from a non-leader session (gated PR, unblocked dependency, ticket to file, cross-domain decision, security finding): **one** handoff to `/memories/leader/inbox/`.
- **Sub-leaders always report back** to the leader inbox if anything outward happened: what was done, what was decided (provisional), what needs the master or the operator. ≤20 lines.
- Cross-tenant news: append a dated block to the cross-tenant log.
- Adopt-now news for sessions already running: also give the operator a one-line paste per live session. Nothing else reaches a running session mid-flight.

## 8. Tickets

Short progress comment. **Never transition status on save.** (Claiming a ticket when you *start* work, and Done on verified merge, are task actions, not save actions.)

## 9. Summary

Terse: rows written (paths), KB promotions, handoffs, PRs, what's open. Then two lines:

```
filed: 3 → <role> · 1 → <other-role> · state → <key>
state: <bytes>B/2048 · hygiene: swept <n> | none-due · telemetry: row | not captured (<why>)
```

`telemetry` is optional: if your setup records session usage (tokens, rate-limit burn) as a row, say `row`; otherwise say `not captured` and why (e.g. no usage source on this surface). Never omit it silently; a missing receipt is how skipped steps hide.

If you can't print the housekeeping line truthfully, the save isn't done.

## Context cycle

If the save was to reset context, end with exactly this and nothing after it (typed one at a time):

```
/compact
/start <role>
```

Never `/clear`. It orphans every live agent.

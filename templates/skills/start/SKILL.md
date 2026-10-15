---
name: start
description: Begin or resume a session. Orient index-first (config, KB index, own role's state and inbox), detect concurrent sessions, then wait for instructions. Trigger on '/start', 'pick up X', 'resume', 'continue', 'fresh chat', or any sign the operator is beginning or returning to work.
argument-hint: "[role | TICKET-KEY | tenant]"
---

# /start: begin or resume

Orient, report, and **don't act until told.**

## Backend

This skill speaks in paths and six verbs: `view`, `create`, `str_replace`, `insert`, `delete`, `rename`.

- **Folder backend (start here):** paths resolve under `$MESH_HOME` (e.g. `~/my-mesh`). `view <dir>` = list the directory and read small files; the rest are ordinary file edits.
- **MCP backend:** the same paths and verbs through your mesh server's tools.

Paths below are written as `/memories/...` and `/kb/...` either way.

## 1. Pick the role and tier

| Argument | Role | Tier |
|---|---|---|
| A persistent role slug from your allowlist (`leader`, `work`, `finance`, ...) | that slug | role preset |
| A ticket key (`PROJ-123`) | the key, lowercased | **ticket** |
| A tenant name | that tenant's default role | **ad-hoc** (light, pinned to one store) |
| Nothing, on Code or a main chat | `leader` | **leader** |
| Nothing, on phone or a blank chat | none | **roamer** |

Unknown slugs are an error, not a fallback. State key: `code-<session-id>` on Code (or `code-<utc-timestamp>-<random>` if there is no session id), `chat` / `chat-<topic>` on chat surfaces, the client name on non-Claude clients.

## 2. Config (always first)

`view /memories/leader/config/` and read: `operator.md`, `sensitive-topics.md` (**non-negotiable, every session**), `context.md`, `operating-contract.md`.

If you have two tenants, config lives in your home tenant. Route by role shape: each non-leader role touches exactly one store; leader reads both.

## 3. KB index

`view /kb/INDEX.md` (one per tenant in play). Do **not** list or read section contents yet.

## 4. Role context, index-first

- **Roamer / ad-hoc:** skim the newest file in `/memories/leader/state/`, then stop. Go to step 8.
- **Small role bucket (≈ under 40 rows):** `view /memories/<role>/` once. Split it into `inbox/*`, `state/*`, and the rest (memory).
- **Large role bucket (including leader):** read only `state/` and `inbox/`. Reach older rows through the recall ladder, never by listing the bucket.

## 5. Ticket anchor (ticket tier)

Load the full ticket and all comments from your tracker. Then run one semantic search on the ticket key plus a few keywords; surface any prior meeting or decision rows as one line each. Zero hits: say nothing.

## 6. Inbound handoffs

Read `/memories/<role>/inbox/` (leader: every tenant's leader inbox, plus the cross-tenant log). Note what you'll act on; tombstoning happens at `/save`. Collapse items already reflected in your own state to "already known."

## 7. Session state and siblings

From `state/`: group by surface, mark ACTIVE (written in the last ~36h), highlight your own key.

- **`## In flight` first.** If your previous state file has this block, check each line (is the job still running, is the PR still open?) and report current status, not what was written. Drop lines that prove dead.
- **Concurrent leaders are normal.** If another ACTIVE leader state exists, print its `## Lane` block, then immediately write your own state file declaring your lane (even `lane: unassigned`). First writer wins a lane.
- **Master seat.** If you are a leader and no other ACTIVE leader holds the master state, you hold it. Otherwise print `SUB-LEADER (master: <key>)`; you will report back on every `/save`.

## 7.5 Agent roster

Your dispatchable agent types are listed in your session context. They were fixed when this session started. Compare with `.claude/agents/`; anything in the directory but not in your context can't be dispatched from this session. Name it in the summary. Never absorb that work inline.

## 8. Summarize and wait

One short block: role, tier, surface, store(s), ticket anchor, inbound items, sibling sessions and lanes, master or sub-leader, `In flight` status, KB index loaded, agent roster drift, suggested first move. Then stop.

## Recall during the session

Index → semantic search → graph (relational questions) → targeted reads. Never bulk-load a bucket. Log misses.

## Rules

- Config and `sensitive-topics.md` first, always.
- Customer data stays in the tenant that owns it: a storage rule, not a processing rule.
- No commits, pushes, ticket transitions or outbound messages during orientation.
- Reads are announced, not asked permission for. Roles gate writes, not reads.
- Main chats are orchestrators: dispatch work to agents with an explicit model.
- `/compact`, never `/clear`.

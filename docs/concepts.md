# Concepts

The vocabulary the rest of the docs assume. Read this first.

## Memory and knowledge are different things

| | Memory | Knowledge base |
|---|---|---|
| Answers | What happened? What's current? | What's true? How does it work? |
| Shape | Many small dated rows, one per thought | Fewer, denser, interlinked pages |
| Lifetime | Working set; old rows get digested or swept | Durable; edited in place, marked superseded |
| Written by | Every `/save` | `/save`, only when something durable emerged |
| Backend (mine) | Key-value store behind an MCP server | Markdown objects in S3 behind the same server |

A memory row says "on Tuesday we chose X over Y because Z." The KB entry says "X is how this works; see the decision row for why."

## Roles are write scopes, not read gates

A **role** names where a session's writes go: a persistent topic (`leader`, `mesh`, `finance`, ...) or a ticket bucket (`proj-123` → the `proj` project bucket). Any session may **read** any role's memory in its own tenant, and should announce it ("that's in finance memory, checking"). Roles never block a read. They only decide where new rows land.

Sessions without a role (a quick phone question) classify each row by content and **announce** where it was filed: `filed: 2 → finance · 1 → mesh`. One-word corrections from the operator fix misfiles.

## Tiers: how heavy `/start` is

| Tier | When | Loads | Budget |
|---|---|---|---|
| **Leader** | The day-starting main chat, once a day | Config, both tenants' leader state + inboxes, the cross-tenant log, comms sweep (dispatched to agents) | Heavy is fine; it happens once |
| **Ticket** | `/start <TICKET>` | Config, the full ticket, its project bucket, relevant KB | Medium |
| **Roamer** | Phone or a blank chat | Config essentials + the KB index + one glance at today's leader state | 2–3 reads; oriented in seconds |
| **Ad-hoc** | `/start <tenant>` for a quick drop-in | Roamer weight, pinned to one tenant | 2–3 reads |

Roamer contract: get oriented enough to route any question, and not one token more.

## State files

Each live session owns exactly one file: `/memories/<role>/state/<key>.md`. The key is per session (`code-<session-id>`, `chat-<topic>`, `chatgpt`, ...). Nobody writes another session's file, so N sessions never collide. It is small (2KB cap), rewritten on every save, and made of pointers. Full contract in [memory-model.md](./memory-model.md).

## Handoffs

A handoff is a file dropped in another role's inbox: `/memories/<recipient>/inbox/<from>-<date>-<slug>.md`. The recipient reads it at `/start`, acts, and tombstones it at `/save`. Deleting an inbox item is also how a session **claims** it, so two concurrent sessions don't both act on it.

Non-leader sessions that produce something the hub must know (a gated PR, an unblocked dependency, a cross-domain decision) send one handoff to the leader's inbox. Cross-tenant news goes to one append-only log that leaders on either side read.

## The leader seat

There is exactly one **master leader** session at a time: whichever leader session holds the leader state file. It owns the operator's queue and decisions of record. Every other session is a **sub-leader** or worker: full tools, subordinate authority, and a mandatory report-back handoff on every `/save`. Work that never reports back didn't happen, as far as the operator brain is concerned.

## Surfaces and accounts

A **surface** is Code, desktop chat, web, phone, a cloud container, or a non-Claude client. Chat is account-level (the same conversation on every device), so devices are not routing destinations; surfaces and accounts are. See [clients.md](./clients.md).

## Two installed skills, everything else fetched

Only `/start` and `/save` are installed. Other skills live as `SKILL.md` files in the repo or KB, listed in a trigger table in `CLAUDE.md`. When a trigger phrase appears, the session reads the skill and follows it. Updating a skill is a commit; no surface needs a reinstall.

## Tenants

A **tenant** is a side of your life with its own stores and its own MCP server: work and personal, in my case. One operator context spans both. See [tenancy-and-data-walls.md](./tenancy-and-data-walls.md).

## Versioning

`x.y.z`: **x** = architectural shift (storage backend, routing). **y** = the operator has to do something (reinstall a skill, re-paste a chat preamble). **z** = Claude can roll it out alone.

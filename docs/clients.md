# Clients and surfaces

The mesh is one brain with several mouths. Anything that can call the MCP server can run `/start` and `/save`.

| Client | Reach | Tiers | State key |
|---|---|---|---|
| Claude Code (laptop) | Both stores, full verbs, git | All | `code-<session-id>` |
| Claude Code (cloud container) | Both stores; fresh container each time | All except the master leader seat | `code-<utc-timestamp>-<rand>` when no session id is set |
| Claude chat (desktop, web, phone) | Connectors for the stores | Leader, ticket, roamer | `chat` or `chat-<topic>` |
| ChatGPT, Codex | Same stores through an OAuth MCP gateway | All | `chatgpt` / `codex` (`-<topic>`) |
| Cursor and other coding agents | Ticket- or task-scoped | Worker only, never leader | per task |

## How each surface gets the skills

- **Code:** `/start` and `/save` are installed (symlinked from your repo, so `git pull` updates them). `CLAUDE.md` carries the trigger table for every other skill.
- **Chat surfaces:** a short preamble in the account's personal preferences says "on a new conversation, orient silently as a roamer; fetch skills from the server by name." Those preferences are one field per account, synced across devices, so one paste covers every chat surface on that account. A server-side skill registry (`list_skills` / `get_skill` over the KB) means skills never need pasting again.
- **Non-Claude clients:** the same registry. Call the server's tool listing for the live verb set rather than pinning a count.

## Surface notes worth knowing

- **Chat is account-level.** The same conversation shows on phone, desktop and web. Devices are access points, not routing destinations.
- **Phone = roamer.** Every fresh phone conversation orients in two or three reads and answers first. No email, calendar or comms sweep on that tier.
- **Voice modes** may not be able to call connectors. Capture by voice, bank it from a text session.
- **Cloud containers** can run any tier, but a cloud leader session is a sub-leader: the master leader seat stays on a persistent machine until cloud sessions can reach everything the seat needs. They also start empty: no user-scope agents or agent memory, no local CLI credentials, no usage snapshot. Put standing specialists in the repo's `.claude/agents/` (project scope) so they travel with the clone. Never pin a remembered session id; resolve by predicate (newest active state file) with a loud no-match branch.
- **Environment variables** like a session id don't exist on non-Claude clients; use the state keys above. Multiple concurrent sessions on any mix of clients is normal; state files and ticket claims are the collision protocol.
- **Ticket systems:** if a client lacks a connector for one tracker, it writes its rows to the store's ticket bucket and hands the tracker update to a surface that has it, in one line.
- **Ticket discipline for new clients:** prefer few durable tickets over many small ones; reserve Done for verified completion.

## A minimal chat preamble

```text
You are part of my Claude Mesh. Memory and knowledge live behind the
"<mesh-connector>" tools. On every new conversation, orient silently as a
roamer: read /memories/leader/config/sensitive-topics.md, /kb/INDEX.md, and
the newest file in /memories/leader/state/. Answer first; read deeper only
on demand using the recall ladder (index, semantic search, graph, targeted
reads). When I say /start or /save, fetch that skill with get_skill and
follow it. Never write customer data outside the tenant that owns it.
```

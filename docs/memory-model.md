# Memory model

## Paths

```
/memories/
  leader/config/            operator.md · sensitive-topics.md · context.md · operating-contract.md
  <role>/
    decisions/<YYYY-MM-DD>-<slug>.md
    sessions/<YYYY-MM-DD>-<slug>.md
    lessons/<YYYY-MM-DD>-<slug>.md
    notes/<YYYY-MM-DD>-<slug>.md        open threads
    state/<session-key>.md              one per live session
    inbox/<from>-<date>-<slug>.md       handoffs addressed to this role
```

Ticket roles (`proj-123`) write into a shared **project bucket**, so a view of one ticket's bucket shows every ticket in that project. Cross-ticket recall is the point.

## Verbs

Six verbs, matching the Anthropic API memory tool: `view` (file or directory), `create`, `str_replace`, `insert`, `delete`, `rename`. In the start-small setup they are plain file operations on a folder. Behind an MCP server they are the same calls over HTTP. The skills only ever speak in paths and verbs, which is why they survived every backend change.

Rules that matter more than the backend:

- **MCP verbs only.** Never query the backing tables directly. The server is where routing, versioning and walls are enforced.
- **Unknown role shapes raise.** A typo in a role name should fail at write time, not create a ghost partition.
- **Deletes are tombstones.** The store keeps version history, so "delete" is recoverable.

## Backend (my setup)

One DynamoDB table: partition key `/memories/<role>`, sort key = the relative path. A `view` of a role directory is one query that returns content inline; a file `view` is one get. Writes are conditional puts. That's it.

Why a key-value store and not a lakehouse: memory traffic is tiny single-item reads and writes. On table formats built for analytics (Iceberg + Athena), each of those cost hundreds of milliseconds to seconds plus a pile of grants. On a KV store they're single-digit milliseconds. Switching made `/start` and `/save` 84–293× faster. Domain data (transactions, logs, measurements) still lives in a lakehouse, because that's what lakehouses are good at.

## Rows: one per distinct thought

A productive session writes 5–15 rows. A row is short, dated, and about one thing. A save that writes one big blob is a defect: it can't be recalled precisely, and it can't be superseded piece by piece.

## The state-file contract

The state file is how a fresh session resumes. It is not a diary.

1. **Hard cap 2KB.** Over the cap means the detail belongs in rows. Bank the rows, then point at them.
2. **Rewrite, don't accrete.** Each save replaces the whole body. No addenda, no "(superseded)" sections. Version history already keeps every prior body.
3. **Pointers.** Anything longer than two lines becomes a row; the state file carries its path. Litmus: could a fresh session resume from the pointers alone?
4. **Version smell.** Past about v20, the session is journaling. Stop and reset to contract shape.

Shape:

```
## Lane            (leader sessions running in parallel: lane, repos held, DO-NOT list)
## Now             ≤5 lines
## In flight       only what lives outside the mesh: background jobs, scratch clones,
                   PRs awaiting review, half-applied changes. Omit when empty.
## Agents          full agent id · specialty · what it holds (if the chat holds live workers)
## Open threads    pointers
## Resume hint     1–2 lines
```

`## In flight` is the block that makes a context reset safe. Everything else is intent, which `/start` restores well. A stale `In flight` line is worse than none, so `/start` checks each one instead of trusting it.

## Hygiene, mechanical on every save

- Tombstone your own `state/code-*` files older than 36 hours. No judgment: a live session refreshes its file on every save, so 36 hours of silence is dead by definition.
- Tombstone your own handoffs older than 30 days, and processed leader-inbox items older than 7 days.
- Cap all sweeps at about 20 tombstones per save, so a save stays bounded.
- Print a housekeeping line so a skipped step is visible: `state: 1432B/2048 · hygiene: swept 2 · telemetry: not captured (why)`.

A step with no output is a step that quietly stops running. The housekeeping line exists because exactly that happened twice.

## Embed and link on save

If you have semantic search, `/save` sends every row it wrote to the embedding index in one call (unchanged content is hash-skipped, so over-sending is free). If you have a graph, it links the obvious edges: ticket keys mentioned, KB paths referenced, plus any relationship the session actually learned. See [recall-ladder.md](./recall-ladder.md).

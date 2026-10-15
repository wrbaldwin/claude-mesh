# Knowledge base

## The idea

Andrej Karpathy described an "LLM wiki": rather than having a model re-retrieve and re-synthesize raw documents on every question, let it build and maintain a persistent wiki of interlinked markdown pages, so knowledge compounds instead of being rediscovered. My KB is that idea applied to one operator's working life.

Three properties make it work:

- **By Claude, for Claude.** Claude decides the sections, writes the entries, keeps the indexes, and fixes the links. I read it when I'm curious, but it is optimized for the model's next session, not for me.
- **Claude-only read/write.** No human edits by hand. Humans get projections (dashboards, summaries) built from it, never the other way round. That keeps the format consistent and the indexes honest.
- **Index-first.** A session loads one small index and pulls entries on demand. It never bulk-loads the KB.

## Layout

```
/kb/
  INDEX.md                    ≤5KB hard cap. Sections, counts, a few top entries each.
  <section>/
    INDEX.md                  one line per entry: path | stream | hook (≤90 chars)
    <slug>.md
```

Sections are domains (the platform itself, each life area, each product, routines, writing...). Mine has around 200 entries in about 14 sections, and the root index still fits in 5KB.

## Entry contract

```markdown
---
title: How X works
domain: <section>
stream: <sensitivity stream>
type: reference | lineage | dossier | digest | catalog
source: where this came from (rows, tickets, docs)
updated: YYYY-MM-DD
---

Dense, factual body. Numbers and dates preserved.
"Superseded by: [[other-slug]]" when it is.
[[cross-links]] to related entries.
```

**An unindexed entry doesn't exist.** Every new entry adds its line to the section index (and bumps the count in the root) in the same write.

## Lifecycle

- **Promotion at `/save`.** After writing memory rows, `/save` asks: did anything durable emerge, something true in six months and useful to any session? If so, create or edit the KB entry and its index line. The memory row stays as the experiential record; the KB entry is the distillation. If nothing durable emerged, write nothing. No filler entries.
- **Librarian pass (periodic).** Digest old memory rows into KB entries, then tombstone the rows. Keeps the hot memory set small.
- **Edit in place.** Prefer `str_replace` on an existing entry over a new near-duplicate. Mark superseded content explicitly.

## What goes where

| Thing | Home |
|---|---|
| What happened, when, who decided | Memory row |
| How something works; the current truth | KB entry |
| Structured facts (transactions, measurements) | Domain tables, with a KB catalog entry pointing at them |
| Code-adjacent docs (runbooks tied to a repo) | Git; the KB points at them, never duplicates |
| Blobs (PDFs, photos) | Object storage, with pointer rows |
| Secrets | A secrets manager. Never in memory or the KB |

## The KB is the shareable layer

Memory is private and in-account. The KB is written **customer-data-free by construction**: it references the ticket or row that holds specifics, never the specifics. That is what makes a team KB possible later. See [tenancy-and-data-walls.md](./tenancy-and-data-walls.md).

## Backend options

| Backend | Good for | Watch out for |
|---|---|---|
| Local folder / OneDrive | Day one. You can see every file | No phone or web access; sync conflicts |
| Wiki (Confluence, Notion) | Human readability | Slow, verbose markup; page-size limits; it pulls you toward writing for humans |
| GitHub | History, diffs, offline | Commit on every save; one clone shared by parallel sessions |
| S3 behind an MCP server | Every surface; versioned; millisecond point reads | Infrastructure to own; you stop casually looking at it |

In S3: versioning on (delete = recoverable marker), create = conditional put, directory listings return metadata only (content comes from the indexes and point reads). Because the KB is never bulk-loaded, S3's slow listing doesn't matter.

## Everyone starts their own

Start a KB in a folder you can see, let `/save` promote into it for a few weeks, and read what it writes. Move it to S3 when you trust it. A shared KB is a later, deliberate step; it works only if every contributor's KB already keeps the data wall.

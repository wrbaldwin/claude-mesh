# The recall ladder

Every session, on every surface and tier, recalls the same way: cheapest rung first, and stop as soon as you have the answer.

```
1. Index      →  2. Semantic search  →  3. Graph  →  4. Targeted reads
```

**Never bulk-load a bucket.** As the stores grow this becomes the single most important rule. One of my type-prefix buckets measured about 386k tokens; a full corpus read was about 773k. A directory view of something that big isn't "thorough", it's a session that's out of context before it starts.

## 1. Index

The KB root index and the memory indexes are loaded at `/start`. Most questions are answered by scanning them and reading one entry. If an index grows past what a session should load, shard it: a tiny root with section names and counts, and one file per section.

## 2. Semantic search

When the index misses, usually because your wording differs from how the entry was written, query the embedding index. Mine covers KB entries and memory rows on both tenants (S3 Vectors with a small embedding model; any vector store works).

- **Returns pointers only**: path, title, score, one-line preview. Never rely on the preview; read the file.
- **Cross-role by default.** Narrow by role or kind only when the hits are noisy.
- Keep it fresh with embed-on-save (see [memory-model.md](./memory-model.md)) plus a periodic backfill.
- Embed prose. Structured columns rarely benefit, except prose-heavy, wording-mismatch fields.

## 3. Graph

When the question is **relational**, not topical, similarity search is the wrong tool:

| Question | Call |
|---|---|
| How are X and Y related? | path between two nodes |
| What's connected to this? | neighbors of a node |
| What breaks if this changes? | impact (downstream walk) |

Node ids are `<type>:<id>`: `ticket:PROJ-12`, `kb:/kb/<section>/<slug>.md`, `table:<schema>.<table>`, `person:<slug>`. `/save` links obvious edges automatically (ticket keys in the text, KB paths referenced, section membership) and the session adds the ones it actually learned. Run impact before any schema change.

## 4. Targeted reads

`view` the winning paths. Read exactly what the work needs.

## Log the misses

When you looked, used the wrong wording, and didn't find something that existed, log it. The miss log is how you know whether recall is good enough to retire older crutches (like pinned topic chats), and it tells you which entries need better titles or links.

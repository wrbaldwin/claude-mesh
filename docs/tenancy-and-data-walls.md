# Tenancy and data walls

## Two tenants, one operator

I run two sides: work and personal. Call them `<tenant-a>` and `<tenant-b>`. Each has:

- its own MCP server,
- its own memory store and its own KB,
- its own cloud account, so the stores are physically separate.

One operator context drives both. That was a deliberate change: two separate brains meant the work side couldn't use anything I'd learned on the personal side (and vice versa), and I kept re-explaining myself.

| Role shape | Store |
|---|---|
| Personal roles and personal ticket buckets | `<tenant-b>` |
| Work role and work ticket buckets | `<tenant-a>` |
| `leader` | Both. Home and config in one; coordination partitions in each |

- **Reads are federated.** A session with access to both stores can read both, and announces it.
- **Writes are routed** by role shape. Every non-leader role touches exactly one store.
- **Cross-tenant news** goes to a single append-only log in one store that leaders on both sides read. Inboxes stay within a tenant.
- **If a store is unavailable**, say so and proceed degraded on the other. Don't silently pretend.

## The data wall

The wall is narrow on purpose:

> **Customer and end-user data stays in the tenant that owns it.** It is a **storage** rule, not a processing rule.

The test is one question: *"Is this fact about a customer or end user?"*

- **Yes:** it can be read, processed and reasoned about by any assistant surface you are permitted to use for that work. It is **saved in full** inside the owning tenant (its ticket bucket, its project files, its systems). It is never authored into the other tenant's rows, and never into **either** KB.
- **No:** schemas, root causes, ticket work, decisions, who your colleagues are and what they own, how a system behaves. That is **your knowledge**, and it flows into your one brain normally.

Two principles fall out of that:

1. **Route, never drop.** An over-cautious session that refuses to save something because it "might be sensitive" causes silent memory loss. If it is customer data, save it in full where it belongs. If it isn't, save it normally. Dropping is never the answer.
2. **KBs are customer-data-free by construction.** They reference the ticket or row that holds specifics, never the specifics. That keeps the KB the shareable layer.

## Enforce walls in architecture, not text

My first version scrubbed rows with pattern matching before embedding them. It was a category error: it guarded an operation that never left the account, held useful rows for manual review, and protected nothing. What actually holds the wall:

- Each tenant's server can only write its own account's buckets and tables. A cross-tenant leak would need a credential the server doesn't have.
- Embeddings for a tenant's memory live in that tenant's vector index.
- The KB rule is a write-time habit enforced in `/save`, and audited.
- Sensitivity rules live in one config file (`sensitive-topics.md`) that every session reads first.

If your wall depends on a regex catching the right string, you don't have a wall.

## Accounts are boundaries too

- A chat surface logged in to the work account should only have the work tenant's connector. Putting a personal store inside an org-administered surface inverts the privacy model; don't request it.
- Things created in a chat account (artifacts, shared pages) are bound to that account. A session under the other login can't update them; hand it off instead of retrying.
- Cloud coding sessions can usually attach only repositories owned by the org the session started in, so give each tenant its own small "home" repo for cloud sessions.

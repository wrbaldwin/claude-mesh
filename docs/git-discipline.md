# Git discipline

Git is where parallel AI sessions do the most damage while every command reports success. These rules came from incidents, not theory.

## HEAD belongs to the clone, not the session

N sessions (or N workers) in one working directory share one HEAD, one index, and one current branch. This is not a laptop quirk. It reproduced in a cloud container with six parallel workers: within four minutes HEAD moved under a running worker, three workers' uncommitted files were swept into a fourth worker's commit, and one commit was left with no ref pointing at it. Every step returned success.

## The rules

1. **Dispatched workers do not touch git.** No `checkout`, `add`, `commit`, `push`, `stash`, `reset`, or PR creation. A worker that changes files writes a unified diff to its own scratch dir and reports the path; the orchestrator applies patches serially. This needs no isolation machinery and cannot corrupt anything.
2. **If a worker genuinely must hold a branch, give it its own worktree** (`isolation: worktree` in Claude Code). Its own worktree means its own HEAD. Costs a little time and disk; use it only for workers that write.
3. **Never `git add -A` or `git commit -a` while any worker is live.** Explicit pathspecs only. `add -A` is how one ticket's work ends up under another ticket's commit message.
4. **Always pass `--head <branch>` when opening a PR** from the CLI. Without it, the tool reads the shared HEAD.
5. **Before you branch, assert the base.** `git checkout -b` inherits whatever HEAD is, and in a shared clone HEAD is often another session's feature branch:
   ```bash
   git fetch origin && git checkout main && git pull --ff-only
   git merge-base --is-ancestor HEAD origin/main   # or compare rev-parse HEAD vs origin/main
   git checkout -b feature/<slug>
   ```
   Every other rule guards the branch you're *on*. None of them guards the commit you branched *from*. One squash merge folded three unrelated commits into main under the wrong ticket that way.
6. **Before every push, assert the branch.** `git branch --show-current` must equal the branch you created. If it doesn't, stop. Don't "fix" it by checking out again; another session may be mid-commit.
7. **A commit isn't safe until a ref contains it.** `git branch --contains <sha>` returning nothing means it is unreachable and will eventually be garbage-collected. The reflog is the only way back.
8. **`git ls-remote origin <branch>` is the only acceptable push verification.** Not the push's exit status, not its output, not "Everything up-to-date", and not `origin/<branch>` (the remote-tracking ref is shared mutable state and is stale in exactly this failure). Observed: a `push -u` exited 0 having created the remote branch two commits behind; the retry said "Everything up-to-date". Prefer an explicit refspec:
   ```bash
   git push origin HEAD:refs/heads/feature/<slug>
   git ls-remote origin feature/<slug>    # must equal: git rev-parse HEAD
   ```

## Merge policy

"Done" means committed, pushed, verified on the remote, and merged.

- **Claude merges its own verified PRs** on low-risk repos: checks green, remote verified, branch deleted afterwards, ticket moved to Done. Presenting a mergeable, non-gated PR as "for your review" is a defect.
- **The only hard guardrail is data loss.** Still gated (a human merges or says go): anything that destroys or replaces a data-bearing resource, production deploys and production data changes on the work side, identity or permission widening, anything marked draft or hold.
- Never force-push main. Never merge a red PR.
- If the harness permission layer blocks a merge Claude is authorized to make, stop and hand over the exact command. Policy and harness permissions are different layers; don't relabel the PR as gated.
- Work repos may have their own conventions (branch naming, a review-bot window before merge). Per-tenant conventions win.

## Pull safely in a repo that also holds local memory files

```bash
git stash && git pull && git stash pop
```

And check sibling sessions' `repos held:` claims before stashing, pulling or branching in a shared working directory.

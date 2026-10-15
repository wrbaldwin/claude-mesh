# HEAD belongs to the clone, not the session

**2026-08-26 / 2026-08-28**

**What happened.** Six parallel workers ran in one cloud clone. Within four minutes, HEAD moved under a running worker, three workers' uncommitted files were swept into a fourth's commit, and one commit was left unreachable. Every git command returned success. Two days later, a session branched off another session's feature branch without checking, and a squash merge put three unrelated commits on main under the wrong ticket, even though every other git rule was followed.

**What we changed.** Dispatched workers never run git; they emit patches and the orchestrator applies them serially (or a writer gets its own worktree). Explicit pathspecs. `--head` on PR creation. Assert the base before branching and the branch before pushing.

**Lesson.** Success exit codes don't mean the right thing happened. Guard the commit you branch *from*, not just the branch you're *on*.

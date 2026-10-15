# A push that exits 0 is not proof

**2026-08-27**

**What happened.** In a cloud container, `git push -u` exited 0 having created the remote branch at a commit two behind the intended one. The retry printed "Everything up-to-date". A session following commit, push, done would have truthfully believed it shipped. The remote-tracking ref agreed, because it is local state too.

**What we changed.** Push with an explicit refspec (`git push origin HEAD:refs/heads/<branch>`) and verify with `git ls-remote origin <branch>` against `git rev-parse HEAD`. Nothing else counts.

**Lesson.** Verify against the remote itself, not against your own cached view of it.

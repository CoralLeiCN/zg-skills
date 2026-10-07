# Merge to Branch: specification

This document records the constraints and accepted limitations for the usage
described in [intent.md](intent.md). The executable instructions live in
[SKILL.md](../../plugins/merge-to-branch/skills/merge-to-branch/SKILL.md).

## Operating constraints

- The source contains committed work and starts clean.
- The user explicitly names a different local target branch, checked out in a
  separate clean worktree.
- One landing runs at a time, without concurrent writes to either participating
  branch or worktree. Work in unrelated worktrees can continue.
- Landing covers the complete committed tree difference. Pushing and cleanup
  require a separate user request.

## Consistency requirements

- Resolve conflicts on the source before landing.
- Review and land a fixed source commit against a recorded target commit.
- Require the staged tree to match the reviewed source tree.
- Reuse successful checks only when their relevant inputs are known to match;
  rerun when equivalence is uncertain.
- Verify one new target commit with exactly the recorded target as its sole
  parent, identical source and target trees, and clean participating worktrees.
- Keep the final upstream refresh when configured. Allow one revalidation if
  the target moves, then stop if it moves again.

## Accepted limitations

The workflow has no shared lock and is not atomic across its Git commands.
Commit, tree, and cleanliness checks detect inconsistencies at checkpoints, but
cannot prevent another task, editor, or Git command from writing between them.
The operating assumptions reduce this risk without eliminating it.

Detection may happen after the target commit has already been created. For
example, a commit hook can modify staged content after the last pre-commit
check. Final verification can detect a differing tree, but does not undo the
commit. A failed verification must report the inconsistency and any target
commit already created without claiming success or automatically discarding
changes.

Repeated target movement can stop a valid landing and require a later retry.
This is an intentional limit on repeated work. Passing the checkpoints confirms
the observed state; it does not guarantee that another writer will leave that
state unchanged afterward.

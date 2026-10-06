---
name: merge-to-branch
description: Squash every committed change from the current Codex task branch into an explicitly named local target branch as one commit while keeping the target history linear. Use when a user asks to merge, squash, land, or finalize the current branch into a branch they provide.
---

# Merge to Branch

Land the complete committed tree difference from the current source branch in a
user-provided target branch as one squash commit.

Success means:

- use the current branch as the source and never infer the target;
- include the complete committed difference between target and source;
- add exactly one non-merge commit to the target;
- leave the target and source with identical file trees; and
- do not push or remove branches or worktrees.

If the user asks only for an explanation, describe the workflow without changing
the repository.

## Handle Git Arguments Safely

Treat every resolved branch, ref, ref range, remote, and worktree path as opaque
data. Prefer a command runner that accepts an argument array and pass each value
as a separate argument. If only a shell command string is available, shell-escape
each complete value before substitution; never concatenate raw values into shell
syntax. Quoted placeholders below mark argument boundaries but do not replace
proper escaping of the resolved values.

## Workflow

Batch independent read-only commands into one tool call where possible. Reuse
their results until a relevant mutation or concurrent change invalidates them.

### 1. Resolve the branches and require clean worktrees

Require the user to provide `<target-branch>`. Run:

```bash
git status --short --branch
git branch --show-current
git worktree list --porcelain
git show-ref --verify --quiet "refs/heads/<target-branch>"
```

Use this source status result to require a clean worktree. If it reports staged,
unstaged, or untracked changes, stop and report the paths. Do not stage, commit,
stash, discard, or otherwise alter pre-existing changes. Continue only after
any separately authorized work is complete and the source worktree is clean.

If the source is detached, create a focused branch and record it as
`<source-branch>`:

```bash
git switch -c "codex/<topic>"
```

Otherwise record the current branch as `<source-branch>`. Stop if the target does
not exist locally or equals the source.

From the worktree list, require exactly one entry with
`branch refs/heads/<target-branch>`. Record its path as `<target-worktree>` and
require it to differ from the current source worktree. Stop if the target has no
worktree or the worktree is ambiguous.

Require the target worktree to be clean:

```bash
git -C "<target-worktree>" status --short --branch
```

Do not alter or stash changes owned by another task.

Treat the complete committed tree difference between `<target-branch>` and
`<source-branch>` as the landing scope. Do not filter individual commits, files,
or hunks from that difference.

### 2. Refresh and record the target

Because `<target-branch>` is checked out in `<target-worktree>`, resolve its
configured upstream through the worktree-local `@{upstream}` shorthand:

```bash
git -C "<target-worktree>" rev-parse --abbrev-ref "@{upstream}"
```

If it succeeds, record the result as `<target-upstream>`. With the target branch
still checked out, fetch without a repository argument so Git uses that branch's
configured remote, then fast-forward from the same upstream shorthand:

```bash
git -C "<target-worktree>" fetch
git -C "<target-worktree>" merge --ff-only "@{upstream}"
```

Stop if the fetch or fast-forward fails. Do not continue with a stale
remote-tracking ref or reset the target to resolve divergence. If the target has
no upstream, use the local target and report that no remote refresh was possible.

Immediately after refresh, record the target HEAD as `<target-before>`, before
updating or validating the source:

```bash
git -C "<target-worktree>" rev-parse HEAD
```

### 3. Update and validate the source

Run from the source worktree:

```bash
git merge-base --is-ancestor "<target-before>" HEAD
```

If the target is not an ancestor (exit status 1), merge that recorded commit into
the source. Stop on other errors. Resolve conflicts only on the source branch:

```bash
git merge "<target-before>"
```

Record the resulting HEAD as `<source-commit>` and its tree as `<source-tree>`.
Use these immutable values for review and landing:

```bash
git rev-parse HEAD "HEAD^{tree}"
git log --oneline "<target-before>..<source-commit>"
git diff --stat --summary "<target-before>" "<source-commit>"
git diff --check "<target-before>" "<source-commit>"
```

If there is no tree difference, stop without creating an empty commit.

Run the relevant repository checks once for the resulting source. Reuse known
successful results from this task when the tested tree, check scope, and relevant
environment, dependencies, and configuration are unchanged. Rerun affected checks
after changed content or conflict resolution; do not reuse results whose inputs
are unknown. Checks that depend on branch or commit metadata must also have
matching inputs. Record what passed or was reused and its tested tree.
Stop if required checks fail.

### 4. Guard the landing point

Before refreshing, require the target worktree to remain clean and on
`<target-branch>`. Keep the final upstream fetch and fast-forward from step 2 when
an upstream exists; stop on failure.

After refresh and immediately before squashing, require both worktrees to remain
clean and on their recorded branches, and the source HEAD to equal
`<source-commit>`. Stop if the source changed unexpectedly; do not silently land
a different snapshot. Compare the target HEAD with `<target-before>`.
If it moved, replace `<target-before>` with the new target HEAD, repeat step 3,
and run this guard again.
Allow at most one such revalidation per invocation. If the target moves again,
stop and report the moving target instead of continuing to loop. Revalidate the
landing difference even when existing test results remain reusable.

### 5. Squash the reviewed snapshot into the target

Run from the target worktree:

```bash
git -C "<target-worktree>" merge --squash "<source-commit>"
git -C "<target-worktree>" write-tree
```

Require the index tree returned by `write-tree` to equal `<source-tree>`. This
verifies the entire staged result matches the reviewed source without repeating
the diff review or whitespace check. Stop on a squash failure, unexpected
conflict, or tree mismatch; do not resolve conflicts on the target.

Reuse the source check results under the same conditions as step 3. Run checks
in the target worktree only when required by repository instructions or when
their inputs differ there or cannot be confirmed equivalent. Keep required
commit hooks enabled.

Immediately before committing, require the target to remain on `<target-branch>`
at `<target-before>`, with the same index tree and no unstaged or untracked
changes. The source must still be clean, on `<source-branch>`, and at
`<source-commit>`. Repeat the tree check after any target-side checks that could
modify files. Stop on unexpected changes and leave them for inspection.

If nothing is staged, do not create an empty commit. Otherwise create one commit
summarizing the landed changes:

```bash
git -C "<target-worktree>" commit -m "<change-summary>"
```

### 6. Verify the result

Run:

```bash
git -C "<target-worktree>" status --short --branch
git -C "<target-worktree>" rev-list --parents -n 1 HEAD
git -C "<target-worktree>" rev-parse "HEAD^{tree}"
```

Then, from the source worktree, run:

```bash
git status --short --branch
git rev-parse HEAD
```

Require both worktrees to remain clean and on their recorded branches, with the
source still at `<source-commit>`. The new target commit must have exactly one
parent, `<target-before>`, and its tree must equal `<source-tree>`. Together these
prove exactly one non-merge commit was added and the source and target have
identical file trees. Do not expect their commit IDs to match after a squash.

## Safeguards

- Do not check out the target in the source worktree.
- Do not use destructive Git commands, force-push, or delete branches or worktrees.
- Stop and report conflicts or product decisions that cannot be resolved safely.
- On failed verification, report the inconsistency and any target commit already
  created; do not claim success or automatically discard changes.
- Do not push or perform cleanup unless the user asks.

Report the source branch, target branch, new target commit, upstream refresh
status, and checks run or reused.

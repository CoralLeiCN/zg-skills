# Merge to Branch: intent

Merge to Branch supports a single user's local development with separate Git
worktrees. It lands a task's committed changes into an explicitly chosen local
branch as one squash commit.

Conflict handling and consistency take priority over speed. The workflow should
avoid repeating reviews and checks when their inputs are known to be unchanged,
while preserving validation when the target advances or conflicts are resolved.

The intended usage has one landing in progress at a time. The user coordinates
tasks and tools so they do not write to the participating branches or worktrees
during landing; independent work in other worktrees can continue.

A shared lock and an atomic transaction covering the entire landing are outside
the current design. The remaining concurrency risk is accepted for this local
usage. The assumptions, consistency requirements, and limitations are recorded
in [spec.md](spec.md).

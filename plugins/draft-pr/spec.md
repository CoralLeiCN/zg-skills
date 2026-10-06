# Draft PR: specification

This document records the constraints and limitations for the purpose described
in [intent.md](intent.md). The operational instructions live in
[SKILL.md](skills/draft-pr/SKILL.md).

## Operating constraints

- The drafting phase preserves branches, working-tree content, remote refs, and
  pull requests. Branch creation, commits, repairs, and publication require a
  separately authorized workflow.
- A targeted fetch may refresh local refs when freshness materially affects
  accuracy and the task and environment permit it; the skill does not check out
  the fetched ref.
- Preserve the user's chosen source, whether it is a ref, range, diff, or
  summary. Resolve the actual PR target rather than assuming it from the source
  branch's upstream or the repository default.
- In a combined publishing request, draft after the intended changes are
  committed. An explicitly requested draft of uncommitted future work must be
  labeled as not independently verified and blocked for publication.
- Follow repository instructions and PR templates. A revision based solely on
  supplied copy can proceed without repository access, with material verification
  limits disclosed.

## Evidence and output requirements

- Ground claims, test outcomes, issue references, and checked template items in
  supplied or inspected evidence.
- For resolvable source and target refs, inspect the merge base and three-dot
  comparison. Classify it as `Clean`, `Contaminated`, or
  `Not independently verified` according to the available evidence.
- Diagnose contaminated comparisons and recommend repair without performing it.
- Return source and target evidence, target resolution, comparison classification,
  exact title and Markdown body, and `Ready for publication` or `Blocked` status.
- A ready handoff requires recorded committed source and target, no material
  comparison blocker, and no material missing evidence. It does not authorize
  publication.
- Regenerate copy if its source, target, changed files, test evidence, or template
  changes before publication. The publisher uses the returned copy exactly.

## Limitations

The draft is only as complete and current as its evidence. Supplied summaries,
unresolvable refs, or inaccessible hosting context may prevent independent
verification. Stale refs must be disclosed; missing facts and test outcomes must
remain explicit rather than being inferred.

Comparison cleanliness and hosting-service mergeability are different facts.
A conflict-free PR can still contain unrelated work, and ready copy does not
guarantee that the hosting service will accept a merge.

This skill diagnoses comparison problems and returns copy. It does not repair
branches, apply replacement copy to an existing PR, or create a draft-status PR.

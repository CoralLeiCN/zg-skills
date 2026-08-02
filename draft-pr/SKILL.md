---
name: draft-pr
description: Draft, revise, or review evidence-backed pull request titles and descriptions after verifying the source, target, and comparison. Use for repository PR templates; stacked or squash-merged branches; comparison contamination; testing, risks, screenshots, and related work. Diagnose branch or history problems without creating branches, rewriting history, pushing, or modifying pull requests.
---

# Draft PR

Create concise, reviewer-oriented PR copy grounded in repository evidence.

Success means:

- identify the intended change source and actual PR target;
- verify the comparison when source and target refs are available;
- trace claims in the title and body to supplied or inspected evidence;
- label missing or unverifiable evidence; and
- avoid source-branch, working-tree, remote, and pull-request mutations.

## Keep the workflow read-only

Inspect the local repository and authorized read-only hosting context. Do not create or switch branches, edit the worktree, create commits, merge, rebase, reset, pull, push remote refs, or open, edit, or publish a pull request.

Fetch a specific source or target ref only when freshness materially affects accuracy and the current task and environment permit it. Do not check out the fetched ref. If the comparison needs repair, diagnose the problem and recommend the next action; leave execution to a separate explicit request.

## Gather evidence

1. Identify the change source:
   - Prefer a user-supplied source ref, commit range, diff, or summary.
   - For an existing PR, use its recorded head ref or hosting-service diff when available.
   - Otherwise use the current branch and `HEAD`.
   - Do not silently substitute `HEAD` for a supplied source. Record whether the source is a ref, range, diff, or summary.
2. Determine the actual PR target before computing a comparison:
   - Prefer an explicitly supplied target.
   - For an existing PR, read its base branch from the hosting service, for example with an authorized GitHub connector or `gh pr view --json baseRefName,headRefName`.
   - Otherwise inspect available stack or branch metadata. Do not mistake the source branch's Git upstream for the PR target; an upstream commonly points to the source branch's remote-tracking counterpart.
   - Do not assume the repository default branch when the PR could target a protected feature branch or another branch in a stack.
   - Verify that source and target refs differ and exist. State when only a stale local or remote-tracking ref is available.
   - If the target is required for an accurate draft and remains ambiguous, ask for the smallest missing fact before drafting.
3. Inspect repository guidance before drafting:
   - Look for `AGENTS.md`, contribution guidance, and PR templates such as `.github/pull_request_template.md` or `.github/PULL_REQUEST_TEMPLATE/*.md`.
   - Follow required headings, checklists, title conventions, and issue-link syntax.
4. Inspect the change itself:
   - review commits, diff statistics, file status, and the substantive diff;
   - inspect relevant tests, docs, configuration, migrations, generated files, and dependency changes; and
   - compare the observed scope with the requested change.
5. Separate committed changes from staged and unstaged work. A normal PR includes only committed changes. Include worktree-only changes only when the user explicitly requests a future-state draft, and label that assumption.
6. Use issue, ticket, design, CI, or existing PR context when supplied or available through an authorized read-only source. Never invent motivation, requirements, test results, issue numbers, rollout details, or performance claims.

For a copy-only revision based on text supplied by the user, preserve its verified facts and improve the writing without requiring repository access. State that the comparison was not independently verified when that limitation matters.

## Validate the PR comparison

When source and target Git refs are available, use the merge base and inspect commands such as:

```bash
git status --short --branch
git merge-base <target-ref> <source-ref>
git log --oneline <target-ref>..<source-ref>
git diff --stat <target-ref>...<source-ref>
git diff --name-status <target-ref>...<source-ref>
git diff <target-ref>...<source-ref>
```

Use qualified refs such as `origin/<target>` when appropriate. Use `HEAD` as `<source-ref>` only when the current checkout is the intended source. Do not treat two-dot and three-dot diffs as interchangeable: prefer the three-dot diff for changes introduced since the source and target diverged.

For a supplied commit range, diff, or summary without resolvable refs, inspect that evidence directly. Report the comparison as `Not independently verified` instead of claiming that a three-dot comparison is clean.

For an existing PR or merge request, compare local findings with the hosting service's changed-files view or diff references when available. Treat the comparison as contaminated when it contains unrelated work, changes belonging to a lower branch in a stack, already-integrated parent work after a squash merge, or an unexpectedly old merge base caused by rewritten history. A target branch being ahead is not by itself contamination.

If the comparison is contaminated:

1. Identify the unexpected commits or files and the likely ancestry cause.
2. Recommend rebasing or restacking onto the verified target. After a stacked parent is squash-merged, recommend replaying only the dependent branch's commits onto the new target.
3. Name the suspected commit boundary only when evidence supports it.
4. State that the three-dot comparison and relevant tests must be rerun after any later repair.

## Draft the PR

Write for a reviewer deciding what changed, why it matters, how it was verified, and where risk is concentrated.

- Match established repository style. Otherwise use a specific, imperative title, usually no more than 72 characters.
- Summarize behavior and intent rather than narrating every changed file.
- Group related implementation details into scannable bullets when useful, usually two to five.
- Call out compatibility changes, migrations, dependencies, security implications, operational impact, rollout needs, and follow-up work only when evidence supports them.
- Report exact test or CI commands and outcomes when known. Write `Not run — <reason>` when tests were not run; never infer execution from the presence of test files.
- Include screenshots or recordings only for user-visible changes. Use a clear placeholder when evidence is still needed.
- Preserve required template checkboxes. Mark a checkbox only when evidence proves it; otherwise leave it unchecked.
- Omit optional sections that add no value unless the repository template requires them.

Use this fallback structure when the repository has no template:

```markdown
## Summary

- <important behavior or outcome>
- <notable implementation detail>

## Testing

<commands and results, or "Not run — <reason>">
```

Append `## Notes` only when supported risks, rollout information, screenshots, related work, or reviewer guidance are material.

## Deliver the result

Return:

1. The verified source and target refs, or the supplied evidence range, and how the target was identified.
2. The comparison status: `Clean`, `Contaminated`, or `Not independently verified`, plus any rebase or restacking recommendation.
3. One recommended title.
4. A copy-ready body in a Markdown code block.
5. Open questions or missing evidence only when they materially affect accuracy.

For revisions of existing PR copy, preserve verified facts and repository-required structure while improving specificity, readability, and reviewer focus. Do not expand drafting into branch repair or remote PR modification.

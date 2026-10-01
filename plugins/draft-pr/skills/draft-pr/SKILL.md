---
name: draft-pr
description: Draft, revise, or review evidence-backed pull request titles and descriptions after verifying the source, actual target, and comparison. Use whenever the user asks to draft, write, revise, or review PR title/body content, including when that request is combined with committing, pushing, or opening a PR. This skill owns the read-only copy-drafting phase; it does not create branches, commits, pushes, or modify pull requests.
---

# Draft PR

Create concise, reviewer-oriented PR copy grounded in repository evidence.

## Route drafting requests correctly

“Draft PR copy” means preparing a PR title and body. “Open a draft PR” means creating a pull request whose hosting-service lifecycle status is draft. For a request such as “commit these changes and draft a PR,” apply both meanings: prepare reviewed title/body copy and let a separately authorized publisher open the PR in draft status.

When a request combines PR-copy drafting with branch, commit, push, or PR-publication work, this skill is still required for the copy-drafting phase. Its read-only constraint applies to this phase only; a separately authorized publishing workflow may create the source commit before this skill runs and may publish the returned title/body afterward. Run this skill after the intended changes are committed and before the PR is created.

Success means:

- identify the intended change source and actual PR target;
- verify the comparison when source and target refs are available;
- trace claims in the title and body to supplied or inspected evidence;
- label missing or unverifiable evidence; and
- avoid source-branch, working-tree, remote, and pull-request mutations.

## Keep the workflow read-only

Inspect the local repository and authorized read-only hosting context. Do not create or switch branches, edit the worktree, create commits, merge, rebase, reset, pull, push remote refs, or open, edit, or publish a pull request.

Fetch a specific source or target ref only when freshness materially affects accuracy and the current task and environment permit it. Do not check out the fetched ref. If the comparison needs repair, diagnose the problem and recommend the next action; leave execution to a separate explicit request.

## Coordinate with a publishing workflow

This skill neither authorizes nor performs mutations. When the user has separately authorized branch, commit, push, or PR-publication work, coordinate the phases in this order:

1. Publisher confirms scope and creates or confirms the intended feature branch.
2. Publisher validates, stages, and commits the intended changes.
3. `draft-pr` reruns source identification and reviews committed `HEAD` against the verified target.
4. `draft-pr` returns the exact title and body.
5. Publisher pushes and creates the PR using that copy.
6. Publisher verifies the resulting PR.

A push is not required before step 3. A committed source ref is sufficient.

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
5. Separate committed changes from staged and unstaged work. A normal PR includes only committed changes.
   - In a combined publish request, do not draft from an unchanged `HEAD` while the intended changes remain staged or unstaged. Allow the authorized publishing phase to create the commit, then rerun source identification and comparison before drafting.
   - If the user explicitly wants copy before committing, inspect the requested future-state evidence and label it `Comparison: Not independently verified — includes uncommitted future-state changes.`
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

Classify the comparison precisely:

- `Clean` means the verified three-dot diff contains only the intended change.
- `Contaminated` means the comparison contains unrelated work, inherited stack changes, already-integrated work, or another evidenced scope mismatch.
- `Not independently verified` means the available evidence does not support a conclusive ref-based comparison.

Comparison cleanliness and hosting-service mergeability are different facts. `Mergeable` means the hosting service currently reports no merge conflict; it does not prove that the three-dot diff is clean. Never substitute mergeability for the comparison classification.

For a supplied commit range, diff, or summary without resolvable refs, inspect that evidence directly. Report the comparison as `Not independently verified` instead of claiming that a three-dot comparison is clean.

For an existing PR or merge request, compare local findings with the hosting service's changed-files view or diff references when available. Treat the comparison as contaminated when it contains unrelated work, changes belonging to a lower branch in a stack, already-integrated parent work after a squash merge, or an unexpectedly old merge base caused by rewritten history. A target branch being ahead is not by itself contamination.

If the comparison is contaminated:

1. Identify the unexpected commits or files and the likely ancestry cause.
2. Recommend rebasing or restacking onto the verified target. After a stacked parent is squash-merged, recommend replaying only the dependent branch's commits onto the new target.
3. Name the suspected commit boundary only when evidence supports it.
4. State that the three-dot comparison and relevant tests must be rerun after any later repair.

A draft is bound to the evidence used to produce it. If `HEAD` or the source commit, the target ref or commit, the changed-file set, test results, or repository template changes after drafting, rerun source identification and comparison and regenerate the PR copy before publication.

## Draft the PR

Write for a reviewer deciding what changed, why it matters, how it was verified, and where risk is concentrated.

- Match established repository style. Otherwise use a specific, imperative title, usually no more than 72 characters.
- Summarize behavior and intent rather than narrating every changed file.
- Group related implementation details into scannable bullets when useful, usually two to five.
- Call out compatibility changes, migrations, dependencies, security implications, operational impact, rollout needs, and follow-up work only when evidence supports them.
- Report exact reproducible test or CI commands and outcomes when known and practical. When a custom validation is evidenced only descriptively, report it honestly, for example `Custom documentation validation — passed`; do not convert it into an implied standard command. Write `Not run — <reason>` when tests were not run, and never infer execution from the presence of test files.
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

Return these stable fields, using explicit unresolved or unavailable values rather than inventing refs or commits:

````text
Source: <ref and commit, or supplied evidence and unresolved commit>
Target: <ref and commit, or unresolved>
Target resolution: <how identified>
Comparison: Clean | Contaminated | Not independently verified
Title: <exact title>
Body:
```markdown
<exact Markdown body>
```
Handoff: Ready for publication | Blocked
````

The title and body are exact publisher inputs. A later workflow must not silently rewrite them; if it needs different copy, rerun this skill against the current evidence.

Use `Ready for publication` only when the copy corresponds to the recorded committed source and target, the comparison has no material blocker, and no material evidence is missing. Use `Blocked` for contaminated comparisons, uncommitted future-state drafts, unresolved source or target commits, or other missing evidence that must be resolved before publication. This handoff status describes whether the copy is ready to publish; it does not assert GitHub mergeability.

Include open questions, missing evidence, and any rebase or restacking recommendation only when they materially affect accuracy or the handoff.

For revisions of existing PR copy, preserve verified facts and repository-required structure while improving specificity, readability, and reviewer focus. Return proposed replacement copy only. Applying it to the existing PR requires a separate explicitly authorized write workflow; do not expand drafting into branch repair or remote PR modification.

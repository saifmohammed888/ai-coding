---
name: pr
description: Take the current changes through a clean pull request workflow - branch, commit, self-review, raise the PR, then merge only after explicit user approval. Use when the user asks to create, raise, review, or merge a PR.
disable-model-invocation: true
argument-hint: "[optional: PR title, issue number, or existing PR number]"
---

# Pull request workflow

Take the work from local changes to a merged PR: **create → review → raise → approve → merge**.

Requested for this run: $ARGUMENTS

If an existing PR number is given, skip to the step that matches its state.

**The merge needs the user's explicit approval. Nothing else in this workflow can stand in for it.**

## 1. Check the starting point

Run these and read the output before changing anything:

```bash
git status
git diff HEAD
git log --oneline -10
gh repo view --json defaultBranchRef,nameWithOwner,squashMergeAllowed,mergeCommitAllowed,rebaseMergeAllowed
```

Stop and ask the user if:

- there is nothing to commit and no unmerged branch to raise
- `gh` is not authenticated or the repo has no GitHub remote
- the changes mix unrelated work, and it is unclear how to split them

## 2. Create the branch

- Never commit directly to the default branch.
- Update from the remote default branch, then branch from it.
- Name the branch `<type>/<short-kebab-description>`, for example `fix/login-redirect-loop`.

## 3. Commit cleanly

Each commit is one logical change. Stage files by name rather than with `git add .`, and leave out anything unrelated: debug code, stray formatting, generated files, local config, and secrets such as `.env` files or keys. If a likely secret is in the diff, stop and tell the user.

Run the project's lint, type-check, and tests. Fix failures before going on. If a check cannot be run, record that for the PR description.

Use Conventional Commits:

```text
<type>(<optional scope>): <imperative summary, 72 characters or fewer>

Why this change is needed, and any context a future reader cannot get
from the diff. Wrap at 72 characters. Omit the body if the summary
already says everything.

Refs #123
```

- Types: `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `build`, `ci`, `chore`.
- Write what the change does, in the imperative: `fix: stop redirect loop on expired sessions`.
- Avoid vague messages such as `update code`, `fix bug`, `changes`, or `WIP`.

## 4. Review before raising

Review the full branch diff against the default branch as a strict reviewer would:

```bash
git diff origin/<default-branch>...HEAD
```

Use the `code-review` skill if it is available. Check for:

- bugs, unhandled edge cases, and missing error handling
- security problems and exposed secrets
- missing or weak tests for the changed behaviour
- leftover debug code, commented-out code, and unrelated changes
- a diff too large to review well; if it covers several concerns, propose splitting it

Fix real problems and commit the fixes. Tell the user about anything left unfixed, and why.

## 5. Raise the PR

Push with `git push -u origin HEAD`, then open the PR with `gh pr create`. The title follows the same format as the commit summary.

Use this body, dropping any section that does not apply:

```markdown
## Summary
What changed and why, in 1-3 sentences.

## Changes
- The main changes, written for a reviewer

## Testing
- What was run and the actual result
- What was not tested, and why

## Notes for reviewers
Risks, trade-offs, follow-ups, or migration steps.

Closes #123
```

Report only testing that actually happened. Open a draft with `--draft` if the work is incomplete.

## 6. Wait for approval

Check the PR's state:

```bash
gh pr view <number> --json url,state,isDraft,mergeable,reviewDecision,headRefOid
gh pr checks <number>
```

Then **stop and end your turn** with:

- the PR link and title
- check status and review status
- the review findings that remain open
- the merge method you propose, and the final commit message
- a direct question: "Do you approve merging this PR?"

Prefer squash merge when the repo allows it, so the default branch gets one clean commit per PR.

Only a clear yes from the user for this PR counts as approval, such as "approve", "merge it", or "yes". These do **not** count:

- passing checks, or an approving review on GitHub
- the original request to run this workflow
- approval given for a different PR
- approval given before the latest commit was pushed

If anything is pushed after approval, ask again.

## 7. Merge

Immediately before merging, confirm the PR is not a draft, is mergeable, has passing checks, and that its head SHA is the one the user approved. If any of these fails, stop and report it.

```bash
gh pr merge <number> --squash --delete-branch --match-head-commit <approved-sha> \
  --subject "<final commit summary> (#<number>)" \
  --body "<final commit body>"
```

Use `--merge` or `--rebase` in place of `--squash` if that is the method the user approved or the only one the repo allows.

Afterwards, switch to the default branch, pull, and confirm the merge landed. Report the merged commit SHA and the PR link.

## Never do these

- Merge without approval, or with `--admin` to get around branch protection
- Skip hooks or signing with `--no-verify` or `--no-gpg-sign`
- Force-push to the default branch; on a feature branch, use `--force-with-lease` only with the user's agreement
- Disable, delete, or skip a failing test to get checks passing
- Resolve a merge conflict by guessing; ask the user when the intent is unclear

---
name: post-merge-cleanup
description: Clean up after the user's own pull request or merge request has merged - confirm the merge from the remote (squash merges included), remove its worktree and branch, revert churn in the main checkout, fast-forward the default branch, and retire notes or memory files that still describe the work as open. Use when the user says "PR merged", "MR landed", "do the cleanups", "prune worktrees", "clean up merged branches", or names a merged branch or PR number. GitHub through gh, GitLab through glab, git always.
---

# Post-merge cleanup

A merged change leaves three kinds of residue: its branch and worktree, churn in the main
checkout, and notes that still call it open. Close all three. Delete nothing the remote does not
confirm as merged; rewrite nothing in notes the current default branch does not confirm.

With no branch or number given, list the local branches `git branch -vv` marks `gone` and take
them one at a time.

## Confirm the merge

Never from recall or from the user's word alone. Where the forge answers, `gh pr view <n>` or
`glab mr view <n>` must say merged. Always: `git fetch --prune origin`, then `git branch -vv` shows
`[origin/<branch>: gone]`, and the branch's own files diff empty against `origin/<default>`
(`git diff --stat <branch> origin/<default> -- <paths>`). Squash merges leave `--merged` blind.
Content on the branch that is not on default: name the commits and stop. That change is not
closed, in git or in notes.

## Git

1. `git -C <worktree> status --ignored --short`. Anything listed: name it and stop.
2. List what goes: worktree path, branch name, churned files with their diff read. Stop for the
   user's word, unless they said to proceed without it.
3. `git worktree remove <path>`, `git worktree prune`, `git branch -D <branch>`, then
   `git checkout -- <churned file>`.
4. `git pull --ff-only` on the default branch in the main checkout.

Never remove the main checkout, the default branch, or a branch whose remote ref still exists.

## Notes and memory

Skip this section when the agent keeps no memory or notes files for the project.

In scope: files whose body names the merged branch, PR or MR number, or ticket. Out of scope:
every other claim in every other file, including one that looks wrong. Those go in the report
under "seen, not touched".

Each claim in a scoped file gets one of four fates, decided by reading the current default branch
or the review thread, never by recall:

- **Chronology** (dates, commit SHAs, pipeline ids, review rounds, "awaiting review"): retire.
  The forge keeps it.
- **Done or false** (a follow-up already on default, a claim the code contradicts): drop, and name
  it in the report with the line that decides it.
- **Durable and unrecorded** (a toolchain gotcha, a design reason, a fact from outside the repo,
  confirmed absent from the repo with a search): fold into the durable file that owns the topic as
  an added paragraph. The rest of that file stays as it was.
- **Ongoing work** (a follow-up still true, an open change this one blocked): keep, in that
  change's file or a follow-ups file, with the line that proves it still applies.

Then delete the merged change's file, fix every link to it (search the directory), drop its entry
from any index file, and update the index entry of each file that gained a paragraph.

## Report

Four lists, at most five items each:

- **Removed**: worktree, branch, churn reverted, notes file, and the default branch's move
  (from, to).
- **Folded**: fact, destination file.
- **Dropped**: claim, deciding line.
- **Seen, not touched**: anything outside scope that looked wrong.

Then one line: what was assumed rather than read. Nothing else.

## Red flags

| Thought | Reality |
|---|---|
| "This durable file's claim is wrong, I'll fix it while here" | Out of scope. Report it; the user decides. |
| "`--merged` lists it" or "the remote branch is gone, so it merged" | Squash hides both. Diff the content against default. |
| "The user said it merged" | The remote says it merged. Fetch, then look. |
| "Lockfile changes are always churn" | Read the diff first. |
| "The report should explain the reasoning" | Four lists and one line. The deciding line is the reasoning. |

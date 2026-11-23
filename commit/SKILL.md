---
name: commit
description: "Write commits in the repository's style. Use when the user asks to commit changes, wants a commit message, or says the work is ready to wrap up."
compatibility: Requires git, a local repository, and the unslop skill. Creating commits requires write access to the repository.
metadata:
  author: razzeee
  source: https://github.com/razzeee/skills/tree/main/commit
---

# Commit

Explain why the work was needed. Invoke `unslop` for all prose, including
questions, commit messages, and the final summary. Preserve exact refs,
trailers, and repository formatting.

For a message-only request, inspect the relevant diff, follow **Write the
message**, and return the text without switching branches, staging, or committing.

## Choose the scope

From the repository root, run these in parallel. Use the same diff options
for later scope checks.

```bash
git status --long --untracked-files=all
git diff --no-relative --ignore-submodules=none
git diff --staged --no-relative --ignore-submodules=none
```

If `git status` reports an operation in progress, including `git am`, ask
before switching branches, splitting groups, or committing. If the user
asked to continue the operation, use its continuation command.

Use the request and current conversation to identify the intended changes.
Leave unrelated or uncertain files out and tell the user why. If the scope
is unclear, offer concrete choices from the changed files, refs, or commit
ranges and wait for the user to choose.

Honor the user's commit count, groups, and order. If the user asks to commit
what's staged, make one commit unless they request or confirm a split.
When grouping is up to you, keep a change with its tests and documentation.
Split changes that a maintainer could revert separately. Ask if the grouping is unclear. Keep
shared-file hunks together unless the user assigns them to separate commits.

## Check the branch

Check `git branch --show-current` before staging. If HEAD is detached, ask
whether to create a branch or commit in place. Keep a suitable topic branch.
Create a new one for work on a protected, default, integration, or release
branch, or when the current branch describes a different task.

For decisions that depend on the default branch, verify its name and tip
with the host or `git ls-remote --symref origin HEAD`. Fetch the branch if
the tracking ref is stale. For a local-only repository, check its configured
default. If you cannot verify the default, ask before switching branches.

If you suspect the current topic branch has already merged, follow
[Check a suspected merge](references/merged-topic.md) before switching.
For other branch switches, create the new branch from HEAD with
`git checkout -b <branch-name>`. Match existing branch names or use a short
kebab-case name. Tell the user which branch you created and why.

## Write the message

Read the repository's contribution instructions and commit template, if
present. Inspect complete messages with `git log -20 --format='%B%n---'`
for subject style, body format, and trailers. Follow explicit requirements
first, then established conventions. Without history or a required format,
use plain imperative prose.

Use the current conversation to understand why the user wanted the work.
If the reason is still unclear, ask. Leave the change list to the diff.
Mention a surprising change only when it helps explain the reason.

- **Subject:** State the reason from the user's perspective when it involves
  a user goal, problem, or tradeoff. For example: "Avoid making users reapply
  search filters". Keep it within 72 characters unless the repository says
  otherwise. If the subject format leaves no room for the reason, use the body.
- **Body:** Explain motivation or tradeoffs the diff cannot show. Include a
  body when readers need context, such as migration instructions, or when the
  repository requires one. Separate it from the subject with a blank line
  and wrap at about 72 characters unless the repository uses another format.

## Commit and verify

Before staging, check whether the index contains changes outside the agreed
groups. If so, leave it intact and ask how to handle them. For staged-only
requests, use the existing index. Staging a partially staged file again can
include edits the user left out. Otherwise, stage only the current group's
files or hunks.

If the index contains multiple approved groups, use
[Split staged changes](references/staged-split.md). If the staged diff
already matches one complete group and the order is flexible, commit it first.

For every commit, inspect the staged diff in the index you will commit.
When it matches the approved group, save its tree ID with `git write-tree`.
Commit without path arguments. After success, compare `git rev-parse 'HEAD^{tree}'`
with the saved ID. If they differ, report the difference and stop before
adjusting the index or creating another commit.

Confirm the result with `git show --stat HEAD` or `git log --oneline -<N>`
for multiple commits. Report each hash and briefly explain any split.

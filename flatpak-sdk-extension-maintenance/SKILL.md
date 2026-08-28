---
name: flatpak-sdk-extension-maintenance
description: >
  Maintains versioned Flatpak SDK and SDK-extension repositories by propagating
  a dependency or release update from a reference pull request across every
  maintained branch/* runtime branch. Use this whenever a user asks to update,
  backport, cherry-pick, synchronize, or roll out changes across Freedesktop,
  GNOME, KDE, or other Flatpak SDK and extension branches, especially when
  branches are several releases behind or conflicts must preserve
  runtime-specific manifest settings. Prepares and validates one update branch
  per target, then requires approval before pushing or opening GitHub pull
  requests. Do not use for ordinary Flatpak application updates, creating a new
  Flatpak package, or general application-code backports.
compatibility: Requires git. GitHub operations require gh and network access.
---

# Flatpak SDK And Extension Maintenance

Carry the **release intent** of a reference PR across `branch/*`; never carry
the source branch's identity. Prefer a verified chain of original update
commits because it preserves intermediate release metadata and produces useful
history on branches that have fallen several versions behind.

## Inputs

Obtain or infer:

- The reference PR URL or `<owner>/<repo>#<number>`.
- The local checkout, or permission to clone the PR's repository.
- Any branch exclusions. With no exclusions, target every remote `branch/*`.

Ask only for information that repository and GitHub state cannot answer.

## 1. Establish The Reference Update

1. Check the primary worktree and remotes. Preserve all existing user changes;
   perform branch work in isolated temporary worktrees.
2. Use `gh pr view` to record the repository, PR number, base branch, head
   commits, title, changed files, merged state, and checks. Fetch the PR head
   through `refs/pull/<number>/head` rather than assuming its branch still
   exists.
3. Inspect every commit and file diff. Identify the semantic transition:
   dependency or component, old version/ref, new version/ref, source URL,
   checksum, release date, and metadata entries.
4. Confirm this is a focused update. Treat unrelated code, permissions,
   runtime changes, or unexplained generated-file changes as a boundary that
   needs user guidance before propagation.

The reference is established when every changed hunk is classified as either
release intent or unrelated scope, and no unexplained hunk remains.

## 2. Inventory Runtime Branches

1. Fetch and prune remote refs without changing the primary worktree.
2. Enumerate remote refs matching exactly `branch/*`. Do not include similarly
   named refs such as `beta`, update branches, or PR heads.
3. For each branch, inspect its manifest and update metadata to determine its
   current version/ref and branch-specific identity. Record at least manifest
   `branch`, runtime, runtime version, SDK, and any differing build settings.
4. Classify each branch:
   - **current**: already contains the desired update; skip it.
   - **behind**: has a version from the same update line; build a chain.
   - **ahead or divergent**: updating would downgrade or cross release lines;
     report it and request guidance.
   - **equivalent PR exists**: an open PR already targets the same branch and
     desired version; skip it and report the URL.

The inventory is complete when every `branch/*` ref has exactly one
classification.

## 3. Build A Verified Update Chain

For each distinct version gap, discover merged update PRs against the reference
base branch and inspect their original commits. Build a contiguous chain where
each transition's old version equals the previous transition's new version,
ending at the reference PR's desired version.

Accept a chain link only when:

- Its component and release line match the reference update.
- Its diff is a focused predecessor of the reference update.
- Its declared source URL, checksum, release date, and metadata agree with the
  commit diff.
- The next link starts at the version produced by this link.

Prefer original PR commits over merge commits. Fetch them through PR refs or
their immutable commit IDs. Never infer a missing checksum or fabricate a
release entry. If no verified contiguous chain exists, explain the missing
transition and request guidance.

## 4. Prepare Each Target

Use one isolated worktree and one new local branch per outdated target. Choose
a deterministic, collision-free name that identifies the runtime branch and
desired version.

1. Start from the current remote target ref.
2. Cherry-pick verified commits oldest-first, preserving their commit messages.
3. If a cherry-pick conflicts, resolve by release intent:
   - Preserve the target's branch, runtime, runtime version, SDK, permissions,
     build options, formatting, and unrelated metadata.
   - Apply the verified source URL/ref and checksum from the picked commit.
   - Add missing release entries once, maintaining the target file's ordering
     and style. Keep target-specific historical entries.
   - Compare the staged resolution with the picked commit's diff before
     continuing.
4. If a conflict reaches outside the classified update hunks or admits more
   than one plausible result, abort that target's cherry-pick, leave other
   targets intact, and report the exact ambiguity.

Release intent crosses branches; branch identity does not.

## 5. Validate Before Publication

Run static and source checks for every prepared target:

1. Confirm the diff changes only files and fields justified by the verified
   update chain.
2. Confirm branch identity matches the original target exactly.
3. Confirm the final manifest names the desired version/ref and source URL.
4. Download each changed source and verify its declared checksum. Reuse a
   securely cached download when branches declare the same source.
5. Parse JSON, YAML, and XML with available repository or system tooling. Run
   relevant lightweight repository lint commands when present.
6. Verify release metadata is newest-first, contains each chained release once,
   and uses the verified dates.
7. Confirm commits are based on the latest remote target and the worktree is
   otherwise clean.

Do not substitute a full local Flatpak build for these checks unless the user
asks. Flathub PR builds provide compilation coverage after publication.

## 6. Draft The Pull Request Body

Write each body after validation so every claim is backed by a completed check.
Keep it short enough to scan alongside the diff: one branch-specific reference
line followed by the checks that passed.

```markdown
Backport of #<reference-pr> to `branch/<runtime>` (includes #<prerequisite-pr>).

Validation:
- source checksum matches the manifest
- manifest and AppStream metadata parse successfully
- release entries are ordered and unique
- branch-specific runtime settings are unchanged
- diff is limited to the expected update files
```

Omit the parenthetical when no prerequisite update is needed. Omit any
validation bullet whose check did not run or pass. If conflict resolution
changed what reviewers need to inspect, replace the relevant generic bullet
with one concrete statement about the preserved target setting.

Apply an unslop writing pass before presenting the draft:

- Remove restatements of the title, process narration, filler, and claims that
  merely say the change is successful or comprehensive.
- Keep only the `Validation:` label; headings such as Summary, Changes, Notes,
  and Testing add structure without information in these small update PRs.
- Name the target branch and link the reference and prerequisite PR numbers.
- Use the same checklist order across branches so differences are visible.

Save each final body to a file for review and later pass that exact file to
`gh pr create --body-file`.

## 7. Approval Gate

Before any push or PR creation, present a table with:

| Target | Starting version | Chain | Local branch | Validation | Result |
|---|---|---|---|---|---|

Include blocked, skipped, and already-current branches. Show the exact proposed
pushes and PR base branches, then show each exact PR title and body. Summarize
resolved conflicts and ask for explicit approval to publish. Preparation
approval from earlier in the conversation is not publication approval.

## 8. Publish After Approval

After explicit approval:

1. Re-fetch each target and stop if its remote head moved.
2. Push each prepared branch without force.
3. Open one PR per target branch with the approved focused title and its saved
   body via `gh pr create --body-file`. Do not regenerate or expand the body
   during publication.
4. Return every PR URL and any branch that failed to publish.

Leave merging to the user and Flathub's checks. Never merge, enable
auto-merge, delete remote branches, or publish a partially validated target.

## Completion

A run is complete when every remote `branch/*` branch is represented by a PR
URL, a verified already-current/open-PR skip, or a precise blocker. Clean up
temporary worktrees after their commits are safely published or the user asks
to discard them; retain unpublished local branches until the user decides.

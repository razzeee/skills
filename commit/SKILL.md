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

## Step 1: Understand the changes

Run these in parallel:

```bash
git status
git diff
git diff --staged
```

If the request or session context does not make the intended changes clear, do
not guess. Inspect the available targets:

```bash
git status --short
git branch --all
git log --oneline -20
```

Ask the user what to compare and commit. Offer the relevant choices you found,
such as:

- All uncommitted changes
- Only staged changes
- Specific changed files or groups of related files
- Changes since a named branch, tag, or commit
- A specific commit or commit range
- Another ref supplied by the user

Show concrete branch names, commit hashes, and changed file groups in the
choices. Allow the user to select one or more groups. Do not stage or commit
anything until the user chooses.

If nothing is staged, don't blindly stage everything. First look at what's changed and decide what actually belongs in a commit:

- **Exclude** files that look like temporary edits, local experiments, environment config (`.env`, `*.local`, IDE settings), debug logging added during development, or anything unrelated to the work described in the session
- **Include** the files that form the logical change and make sense together in the diff
- Use session context to guide this: if the user described what they were working on, the commit should reflect that work, not everything `git status` happens to show

When unsure about a file, leave it out and tell the user why.

Once you've decided what belongs, keep that file selection for the commit step.
Check the branch before changing the index, then stage only the intended changes
as described in Step 5.

## Step 1b: Decide whether to split into multiple commits

A single logical change belongs in a single commit. But if the staged changes clearly contain multiple unrelated things, splitting them produces a cleaner history.

**Split when:**
- The changes address distinct concerns that could be reverted independently (e.g. a bug fix in `auth.js` + an unrelated dependency bump in `package.json`)
- The diff spans files from different parts of the codebase with no connection between them
- The session context suggests separate tasks were completed (e.g. "I fixed the login bug and also added the export feature")

**Don't split when:**
- The changes work together as one unit (e.g. a new feature file + its test + a README update)
- The changes are small and the split would be pedantic
- You would have to stage parts of files to separate the changes cleanly

**When it is ambiguous**, ask whether the user wants one commit or separate commits for the distinct changes.

Plan the groups here. After checking the branch, stage and commit each group
separately in Step 5, keeping later groups out of the index until their turn.

## Step 2: Check the current branch

```bash
git branch --show-current
git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null || echo "no remote default"
```

Before committing, decide whether the current branch is the right place for this work. There are three situations that call for a new branch:

**1. You're on a protected/shared branch**

These branches should receive changes through pull requests, not direct commits:
- The repo's default branch (`main`, `master`, or whatever `origin/HEAD` points to)
- Common integration branches: `develop`, `dev`, `staging`, `next`
- Release branches matching patterns like `release/*`, `release-*`

**2. The current branch has already been merged**

Resolve the remote default branch from `origin/HEAD`, then check whether the
current branch has been merged into that ref:

```bash
git symbolic-ref --short refs/remotes/origin/HEAD
git branch --merged <remote-default-ref>
```

If the current branch appears in that list, it has already been merged. Create a new branch instead of adding commits to it. If the remote default ref is unavailable, inspect the repository's refs or ask rather than assuming its name.

**3. The branch topic doesn't match the work**

Compare the current branch name to what the user is actually working on. If you're on `fix/login-redirect` but the user just finished building the CSV export feature, that branch is the wrong home for this commit. Create a new branch that reflects the actual work.

Use the session context to decide whether the branch name still describes the work.

---

In all three cases, name the new branch by first checking existing conventions:

```bash
git branch -a
```

Common patterns: `feat/add-login`, `fix/null-pointer`, `username/short-description`, `TICKET-123-description`. Match what you see. If there's no clear pattern, use a short kebab-case description of the change.

Create and switch before staging or committing anything:

```bash
git checkout -b <branch-name>
```

Tell the user why you're switching and what branch you created: "The current branch `fix/login-redirect` doesn't match this work, so I'll create `feat/csv-export`."

**If HEAD is detached** (not on any branch), warn the user and ask whether to create a branch or commit in place.

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

## Step 5: Commit

For a staged-only request, preserve the existing index: do not run `git add`
again on those files. A partially staged file may contain unrelated unstaged
edits that must remain outside the commit.

For other requests, stage only the intended changes. Stage whole files explicitly
only when all their changes belong in the commit; otherwise stage the selected
hunks while preserving existing staged content. Inspect `git diff --staged`
before committing to confirm the index matches the requested scope.

```bash
git commit -m "<subject>" -m "<body paragraph 1>" -m "<body paragraph 2>"
```

Or use a heredoc if the message is long:

```bash
git commit -F - <<'EOF'
<subject>

<body>
EOF
```

## Step 6: Confirm

Show the result. For a single commit:

```bash
git show --stat HEAD
```

For multiple commits:

```bash
git log --oneline -<N>
```

Tell the user what was committed and provide each commit hash. If you split the work, briefly explain why.

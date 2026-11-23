# Check a suspected merge

Before replacing a topic branch, confirm that its current tip reached the
verified default branch during this use of the branch name. A newly created
branch can point at the default tip. An old pull request can refer to a
different branch with the same name.

Check the available merge record:

- **Hosted pull request:** Match the source repository, branch, and head
  commit of a merged pull request. For a squash or rebase, follow the
  resulting commit, not the original tip.
- **Local merge commit:** Find the source tip among the non-first parents in
  `git log --merges --format=%P <receiving-ref>`. Use the receiving branch's
  reflog or merge record to identify the source branch. Two branches may
  share a tip. Check the topic's reflog to see whether someone reused its name.
- **Local fast-forward or squash:** For a fast-forward, match the receiving
  reflog entry to the source name and tip. For a squash, compare the source's
  combined diff since its fork point with the candidate commit's diff and
  check the merge record. Matching content alone does not identify its source.

If the topic landed through an integration branch, trace each subsequent
merge on its receiving branch. Verify that the merged integration tip
contains the preceding result. It can include newer work. At each squash or
rebase, follow the replacement result through to the verified default branch.

Keep a suitable topic branch if you find no evidence of a merge. If the evidence
suggests a merge but its source or destination is unclear, ask before
switching. For a confirmed merge, start the new branch at the verified
default ref so its history includes the landed work:

```bash
git checkout --no-overwrite-ignore --no-track -b <branch-name> <verified-default-ref>
```

If the switch would overwrite local work, stop and ask. Do not force it.

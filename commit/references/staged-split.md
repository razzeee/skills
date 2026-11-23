# Split staged changes

Use a temporary index to commit one group while the others stay staged.
Copy the group's contents from the real index. Reading from the working
tree could commit an untracked replacement for a file staged for deletion.

1. If the current group still needs staging, stage only its approved files
   or hunks. Leave other groups staged. If the split within a shared path is
   unclear, ask which hunks belong together before changing the index.
2. Snapshot the real index with `git write-tree`. In a subshell, export
   `GIT_INDEX_FILE` pointing to a new file path inside a temporary directory.
   Initialize it with `git read-tree HEAD`, or `git read-tree --empty` for
   the first commit.
3. Copy complete paths from the snapshot with
   `git restore --staged --source=<snapshot> -- <group-paths>`. Include both
   paths of a rename. For a shared file, use
   `git restore --patch --staged --source=<snapshot> -- <path>` and select
   only the assigned hunks. Stop and ask if you cannot separate them exactly.
4. Follow [Commit and verify](../SKILL.md#commit-and-verify) in the temporary
   index, starting at "For every commit". On success, exit the subshell and
   check the remaining staged diff in the real index. Remove the temporary
   directory when finished.

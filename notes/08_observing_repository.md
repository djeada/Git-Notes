## Checking and Understanding Changes

Git's powerful suite of commands offers an insightful look into your codebase's progression. By probing changes, tracking progress, identifying anomalies, and fostering effective collaboration becomes easier.

### Viewing Changes

Different commands in Git allow you to inspect modifications in your project. Here’s a comprehensive guide to viewing changes:

I. **List Uncommitted Files:**

Use the `git status` command to see which files have been changed, added, or deleted but haven’t been committed yet.

```bash
git status
```

This command provides a summary of the current state of your working directory and staging area.

II. **View Unstaged Changes:**

To see the differences in files that have not yet been staged for commit, use the `git diff` command.

```bash
git diff
```

This command shows the changes between your working directory and the staging area.

III. **See Staged but Uncommitted Changes:**

If you have staged changes but haven’t committed them yet, you can view these differences with the `git diff --cached` command.

```bash
git diff --cached
```

This command reveals the differences between the staging area and the last commit.

IV. **Inspect All Changes Since the Last Commit:**

To see all changes (both staged and unstaged) since the last commit, use the `git diff HEAD` command.

```bash
git diff HEAD
```

This command compares your working directory and staging area against the last commit.

V. **Highlight Word-Level Changes:**

For a more granular view of changes, the `git diff --word-diff` command shows differences at the word level rather than the line level.

```bash
git diff --word-diff
```

This command highlights the exact words that have been added or removed, providing a clearer view of the changes.

VI. **Audit a File's Change History:**

To track changes line by line in a specific file, use the `git blame` command followed by the filename. This command annotates each line with information about the last commit that modified it.

```bash
git blame filename
```

Example: To audit changes in `index.html`, use:

```bash
git blame index.html
```

VII. **Viewing Changes in a Specific File:**

You can use `git diff` followed by the filename to see changes in a particular file.

```bash
git diff filename
```

Example: To view changes in `style.css`, use:

```bash
git diff style.css
```

### Comparing Commits

Dive deeper into differences between commits using the following Git commands:

I. **Visualize Commit History:**

To see a chronological list of commits along with their messages, authors, and dates, use the `git log` command.

```bash
git log
```

This command provides a detailed history of all commits in the repository, helping you trace changes over time.

II. **Visualize a More Detailed Log:**

For a more detailed commit history, including differences between commits, use:

```bash
git log -p
```

III. **Graphical Representation of Commit History:**

To visualize the commit history as a graph, showing branches and merges, use:

```bash
git log --graph --oneline --all
```

IV. **Contrast Two Specific Commits:**

To compare the differences between two specific commits, use the `git diff` command followed by the commit IDs.

```bash
git diff commit_1 commit_2
```

Example: To compare commit `a1b2c3d4` with commit `e5f6g7h8`, you would run:

```bash
git diff a1b2c3d4 e5f6g7h8
```

This command details the changes made between the two specified commits.

V. **Review a File's State at a Given Commit:**

To view the state and changes of a specific file at a particular commit, use the `git show` command followed by the commit ID and the filename.

```bash
git show commit_id:filename
```

Example: To see the state of `index.html` at commit `a1b2c3d4`, use:

```bash
git show a1b2c3d4:index.html
```

This command displays the content of the specified file as it was at the given commit.

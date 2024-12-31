## Checking and Understanding Changes

Git provides commands to examine your codebase’s changes, track progress, identify issues, and support collaboration. Knowing how to check and interpret these changes is important for maintaining a clear and organized project history.

### Viewing Changes

Different commands in Git allow you to inspect modifications in your project. Here’s a comprehensive guide to viewing changes:

Viewing changes helps you stay informed about what has been altered in your project. Whether you want to review your own changes before committing or understand modifications made by others, these commands provide the necessary insights.

I. **List Uncommitted Files:**

Use the `git status` command to see which files have been changed, added, or deleted but haven’t been committed yet.

```bash
git status
```

This command provides a summary of the current state of your working directory and staging area. It highlights untracked files, modified files, and files staged for commit, giving you a clear picture of what needs attention.

II. **View Unstaged Changes:**

To see the differences in files that have not yet been staged for commit, use the `git diff` command.

```bash
git diff
```

This command shows the changes between your working directory and the staging area. It displays added, removed, and modified lines, allowing you to review what will be included if you decide to stage these changes.

III. **See Staged but Uncommitted Changes:**

If you have staged changes but haven’t committed them yet, you can view these differences with the `git diff --cached` command.

```bash
git diff --cached
```

This command reveals the differences between the staging area and the last commit. It helps you verify what exactly will be included in your next commit, ensuring that only the intended changes are recorded.

IV. **Inspect All Changes Since the Last Commit:**

To see all changes (both staged and unstaged) since the last commit, use the `git diff HEAD` command.

```bash
git diff HEAD
```

This command compares your working directory and staging area against the last commit. It provides a comprehensive view of all modifications, helping you understand the full scope of changes made since the last recorded state.

V. **Highlight Word-Level Changes:**

For a more granular view of changes, the `git diff --word-diff` command shows differences at the word level rather than the line level.

```bash
git diff --word-diff
```

This command highlights the exact words that have been added or removed, providing a clearer and more precise view of the changes. It's particularly useful for reviewing small edits within lines, such as fixing typos or modifying specific terms.

VI. **Audit a File's Change History:**

To track changes line by line in a specific file, use the `git blame` command followed by the filename. This command annotates each line with information about the last commit that modified it.

```bash
git blame filename
```

To audit changes in `index.html`, use:

```bash
git blame index.html
```

`git blame` is invaluable for understanding the history of a file, identifying when and why specific changes were made, and attributing those changes to the respective authors. This insight aids in debugging and maintaining accountability within the project.

VII. **Viewing Changes in a Specific File:**

You can use `git diff` followed by the filename to see changes in a particular file.

```bash
git diff filename
```

To view changes in `style.css`, use:

```bash
git diff style.css
```

This command focuses on the specified file, displaying only the differences related to it. It helps you concentrate on the modifications of a single file without the distraction of changes in other parts of the project.

### Comparing Commits

Dive deeper into differences between commits using the following Git commands:

Comparing commits allows you to understand how your project has evolved over time. It helps in identifying when specific changes were introduced and how different parts of the codebase have been modified across various commits.

I. **Visualize Commit History:**

To see a chronological list of commits along with their messages, authors, and dates, use the `git log` command.

```bash
git log
```

This command provides a detailed history of all commits in the repository, helping you trace changes, understand the progression of the project, and identify key milestones or updates made by contributors.

II. **Visualize a More Detailed Log:**

For a more detailed commit history, including differences between commits, use:

```bash
git log -p
```

The `-p` option generates patch output for each commit, showing the exact changes introduced. This enhanced log is useful for in-depth reviews and understanding the specific alterations made in each commit.

III. **Graphical Representation of Commit History:**

To visualize the commit history as a graph, showing branches and merges, use:

```bash
git log --graph --oneline --all
```

This command creates a visual representation of the commit history, displaying branches and merges in a concise format. It helps you grasp the branching structure and how different lines of development have interacted over time.

IV. **Contrast Two Specific Commits:**

To compare the differences between two specific commits, use the `git diff` command followed by the commit IDs.

```bash
git diff commit_1 commit_2
```

To compare commit `a1b2c3d4` with commit `e5f6g7h8`, you would run:

```bash
git diff a1b2c3d4 e5f6g7h8
```

This command details the changes made between the two specified commits, allowing you to understand what has been added, modified, or removed over that range of commits.

V. **Review a File's State at a Given Commit:**

To view the state and changes of a specific file at a particular commit, use the `git show` command followed by the commit ID and the filename.

```bash
git show commit_id:filename
```

To see the state of `index.html` at commit `a1b2c3d4`, use:

```bash
git show a1b2c3d4:index.html
```

This command displays the content of the specified file as it was at the given commit, enabling you to review past versions of the file and understand how it has evolved over time.

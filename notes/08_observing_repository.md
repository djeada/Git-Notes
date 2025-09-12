## Checking and Understanding Changes

Git offers several ways to inspect and understand what has changed in your codebase. Mastering these commands helps you monitor progress, spot issues early, and keep your project history organized. Think of it like reading the "track changes" feature in a word processor—but for your entire code project.

### Viewing Changes

Different Git commands let you look at what’s been modified, who made the changes, and exactly how your files differ now from previous versions. Having a solid grasp of these tools makes your collaboration and debugging sessions a lot more efficient.

#### List Uncommitted Files

To list uncommitted files, use:

```bash
git status
```

It shows you which files are tracked, which are untracked, and which have been modified since your last commit. It also tells you which files are in the staging area.

Example Output:

```
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   index.html

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	newscript.js
```

- `modified: index.html` means `index.html` is being tracked by Git but has changes not yet staged.
- `newscript.js` is untracked, so Git doesn’t know about it yet. You’ll need to stage it before committing.

#### View Unstaged Changes

To view unstaged changes, use:

```bash
git diff
```

It compares your working directory (the files you’re actively editing) to the staging area. It highlights the lines that have been added, removed, or changed but are not yet staged for commit.

Example Output:

```
diff --git a/style.css b/style.css
index f7c2963..3e0f18e 100644
--- a/style.css
+++ b/style.css
@@ -12,6 +12,8 @@ body {
  margin: 0;
  background-color: #fff;
+ color: #333;
+ font-family: Arial, sans-serif;
}
```

- Lines prefixed with a `+` sign indicate new additions (`color: #333;` and `font-family: Arial, sans-serif;`).
- Lines prefixed with a `-` sign would show removed code.
- The context lines show surrounding code for reference.

#### See Staged but Uncommitted Changes

To see staged but uncomitted changes, use:

```bash
git diff --cached
```

`git diff --cached` (sometimes written as `git diff --staged`) lets you see the differences between what’s in the staging area and the last commit. This way, you know exactly which changes will go into your next commit.

Example Usage and Output:

```
# Stage a file first
git add style.css

# Then check the staged changes
git diff --cached
```

You might see output similar to the previous example, but only for `style.css` if that’s the file you staged.

This is useful for a final check before committing, ensuring you don’t accidentally include changes you’re not ready to commit.

#### Inspect All Changes Since the Last Commit

To inspect all changes since the last commit, use:

```bash
git diff HEAD
```

It compares everything in your working directory and staging area to the most recent commit (`HEAD`). It’s like an all-in-one snapshot of what’s changed, both staged and unstaged, since you last committed.

If you see output showing differences in multiple files, that’s Git telling you, “Here’s what’s changed everywhere since your last commit.”

This provides a broad overview. It’s a quick way to review all modifications before deciding what to stage and commit.

#### Highlight Word-Level Changes

To highlight word-level changes, use:

```bash
git diff --word-diff
```

By default, `git diff` compares changes line by line. When you add `--word-diff`, Git breaks down each line further, highlighting exactly which words or tokens have changed.

Example Output:

```
@@ -5,7 +5,7 @@
 Here is some text that has changed a little.

 - This is the old line
 + This is the new line
```

Instead of simply showing a whole line as changed, `--word-diff` zeroes in on the specific words you altered. If you often fix minor typos or modify small text segments, this command saves you from scanning entire lines to find those tiny tweaks.

#### Audit a File's Change History

To audit a file's change history, use:

```bash
git blame filename
```

It shows you who last modified each line of a file, along with the commit hash and timestamp. It’s helpful when you’re curious about why a particular change was made or who to ask about it.

Example:

```bash
git blame index.html
```

You’ll see something like:

```
a4d6e439 (Alice   2025-04-01 10:12:56) <header>
d35f7abc (Bob     2025-04-02 14:33:10)   <h1>Welcome</h1>
a4d6e439 (Alice   2025-04-01 10:12:56) </header>
```

- Each line is listed with the commit hash (`a4d6e439`), the author’s name (`Alice`), the date/time, and the code snippet.
- This is especially good for pinpointing when (and who) introduced a bug or feature.

#### Viewing Changes in a Specific File

To viewing changes in a specific file, use:

```bash
git diff filename
```

By specifying a filename, you can focus on changes in just that file, ignoring the rest of the codebase. This is handy if you have a large project but only want to review a single file’s modifications.

Example:

```bash
git diff style.css
```

If you have many files updated across your repo, but you’re only concerned about `style.css`, you can run this command to zero in on those changes and not get distracted by others.

### Comparing Commits

Git offers various ways to look at differences between commits, so you can see how your project has evolved over time. If you want to find out when a piece of code was introduced or track how a function has changed, these commands will get you there quickly.

#### Visualize Commit History

To visualize commit history, use:

```bash
git log
```

Displays a chronological list of commits with messages, authors, and dates. It’s like a timeline that shows how your codebase reached its current state.

Example Output (Shortened):

```
commit 5f86bc9fd2f4e38f7c43f953690a568248f6dfd0
Author: Alice <[email protected]>
Date:   Wed Apr 2 14:33:10 2025 +0000

    Add new feature to handle user input

commit a37c3bdcb743aebf5d264dfb167ae2b16c7f6891
Author: Bob <[email protected]>
Date:   Mon Mar 31 09:15:01 2025 +0000

    Fix layout issue on homepage
```

- Each commit is listed with its unique ID (like `5f86bc9fd2f4e...`), the author, the date, and the commit message.
- This helps you pinpoint when changes were introduced and by whom. It’s great for an overview of the project’s progress.

#### Visualize a More Detailed Log

To visualize a more detailed log, use:

```bash
git log -p
```

Generates a patch (`-p` stands for “patch”) for each commit in your log, showing the exact changes made. It’s similar to reading a series of diffs, one commit at a time.

Example Output (Shortened):

```
commit 5f86bc9fd2f4e38f7c43f953690a568248f6dfd0
Author: Alice <[email protected]>
Date:   Wed Apr 2 14:33:10 2025 +0000

    Add new feature to handle user input

diff --git a/app.js b/app.js
index 7cddf2d..b15343b 100644
--- a/app.js
+++ b/app.js
@@ -10,6 +10,9 @@ function processInput() {
     let userInput = document.getElementById('inputField').value;
     // Additional checks
+    if (!userInput) {
+       alert("Input cannot be empty!");
+    }
     return userInput.trim();
}
```

- You see not only the commit message but also the actual lines changed.
- This can be a deep dive into the evolution of the code. It’s the best way to trace exactly how each commit modified files.

#### Graphical Representation of Commit History

To see graphical representation of commit history, use:

```bash
git log --graph --oneline --all
```

Visualizes commits as a branching graph in your terminal. Each branch and merge shows up as separate lines or paths, making it easier to follow complex development workflows.

Example Output:

```
* 5f86bc9 (HEAD -> main) Add new feature to handle user input
* a37c3bd Fix layout issue on homepage
|\
| * b47c582 (feature-branch) Start working on search functionality
| * 1c9a6bd Add basic search form
|/
* 2d3a1f3 Initial commit
```

- The `*` symbols and lines illustrate how branches diverged and merged.
- It’s a quick way to see which commits belong to which branch and how they tie together.

#### Contrast Two Specific Commits

To contrast two specific commits, use:

```bash
git diff commit_1 commit_2
```

Shows you the differences between any two commits you specify by their commit IDs. This is perfect when you need to see how your code changed from one milestone to another.

Example:

```bash
git diff a1b2c3d4 e5f6g7h8
```

- You’ll see a diff output indicating what was added or removed between commits `a1b2c3d4` and `e5f6g7h8`.
- Great for pinpointing exactly when a bug or feature was introduced if you have an idea of which commits to compare.

#### Review a File's State at a Given Commit

To review a file's state at a given commit, use:

```bash
git show commit_id:filename
```

Retrieves the version of a particular file as it was at the specified commit. It’s like time-travel for a single file, letting you see or copy-paste the content from that point in history.

Example:

```bash
git show a1b2c3d4:index.html
```

- You’ll see the contents of `index.html` exactly as it was in commit `a1b2c3d4`.
- Useful when you need to revert or refer to a previous implementation without restoring the entire repository to that commit.

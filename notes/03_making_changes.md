## Working with Git: Staging, Committing, and Undoing Actions

At the core of Git are a few fundamental actions: staging changes, committing those changes, and, when necessary, undoing certain actions. These notes provide a clear overview of these basic operations and some common scenarios where they are used.

### Staging files

To prepare changes for a commit, we "stage" them. 

```
+--------------+    +--------------+     +-------------------+
|              |    |              |     |                   |
| Working      |    | Staging      |     | Repository        |
| Directory    |    | Area (Index) |     | (Commit History)  |
|              |    |              |     |                   |
|  (Changes)   | -> | (Changes to  | ->  | (Commits)         |
|              |    | be committed)|     |                   |
+--------------+    +--------------+     +-------------------+
```

Here's how:

- **Stage a specific file:**

```bash
git add filename
```

- **Stage all changed files:**

```bash
git add .
```

### Unstaging files

If you've staged changes but haven't committed them, you can unstage them:

- **Unstage a specific file (but keep changes):**

```bash
git reset filename
```

- **Revert everything to the state of the last commit:**

```bash
git reset --hard
```

### Committing files

Commits are similar to saves in that they capture the current state of your project and associate it with a message. The commit message should describe the changes made since the last commit. You can use the git checkout command to go back to a previous state of your project that was captured in a commit.

```
+----------+
|  commit  |
|   hash   | A unique identifier (SHA-1 hash) for the commit.
+----------+
|  author  | The person who authored the commit.
|  date    | The date and time the commit was made.
+----------+
| message  | A brief description of the changes made in the commit.
+----------+
| changes  | The actual changes made to the files.
+----------+
```

- **Commit all staged files with a message:**

```bash
git commit -m "Your descriptive commit message"
```

- **Push your commits to a remote repository (like GitHub):**

```bash
git push
```

- **Go back to a previous state captured in a specific commit:**

```bash
git checkout commit_id
```

### Undoing commits

Made a mistake? Here's how you can undo it:

- **Undo the last commit but keep all files staged:**

```bash
git reset --soft HEAD~
```

- **Undo the last commit and unstage the changes:**

```bash
git reset HEAD~
```

- **Erase all changes made in the last commit:**

```bash
git reset --hard HEAD~
```

### Editing commit messages

Your commit message could be better? Let's fix that:

- **Edit the message of the most recent commit:**

```bash
git commit --amend -m "New, improved message"
git push --force
```

- **Modify an older commit message:**
1. Start an interactive rebase for the last `n` commits:
 
  ```bash
  git rebase -i HEAD~n
  ```

3. Change "pick" to "reword" for commits you'd like to modify.
4. Save and close. For each commit, provide a new message.
5. Push the changes:
 
  ```bash
  git push --force
  ```

### Removing commits

Deleted more than you intended? Here's how to go back:

- **Remove the last N commits:**

```bash
git reset HEAD~N
git push origin +HEAD
```

### Cleaning up your commit history

Have sensitive data or a large file in your history? Here's how to remove it:

- **Remove a file from the commit history:**

```bash
git filter-branch --index-filter 'git rm -r --cached --ignore-unmatch file_to_remove' HEAD
```

Note: Always be careful with commands that alter commit history, especially if collaborating with others. If you're unsure, consult with your team or check Git documentation.

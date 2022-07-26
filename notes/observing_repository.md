<h1>Check the changes since the last commit</h1>

List new or modified files not yet committed:

```bash
git status
```

Show the changes to files not yet staged:

```bash
git diff
```

Show the changes to staged files:

```bash
git diff --cached
```

Show all staged and unstaged file changes:

```bash
git diff HEAD
```

Show word changes in diff:

```bash
git diff --word-diff 
```

List the change dates and authors for a file:

```bash
git blame filename
```

<h1>History of commits</h1>

Show the repository's complete history of changes:

```bash
git log
```

<h1>Comparing commits</h1>

Show the changes between two commit ids:

```bash
git diff commit_1 commit_2
```

Show the file changes for a commit id and/or file:

```bash
git show commit_id:filename
```

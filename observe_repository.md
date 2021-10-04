List new or modified files not yet committed:

git status

Show the changes to files not yet staged:

git diff

Show the changes to staged files:

git diff --cached

Show all staged and unstaged file changes:

git diff HEAD

Show the changes between two commit ids:

git diff commit_1 commit_2

List the change dates and authors for a file:

git blame filename

Show the file changes for a commit id and/or file:

git show commit_id:filename

Show full change history:

git log

<h1>Show word changes in diff</h1>

```bash
git diff --word-diff 
```

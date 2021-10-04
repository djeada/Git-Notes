TODO: 
* add undo for add and commit 
* change commit message
* remove commit

Stages the file, ready for commit:

git add filename

Stage all changed files, ready for commit:

git add .

Commit all staged files to versioned history:

git commit -m “commit message”

Unstages file, keeping the file changes:

git reset filename

Revert everything to the last commit:

git reset --hard

<h1>How can I edit the commit message after a git push?</h1>

Last commit:

```bash
git commit --amend -m "new message"
git push --force
```

Old commit:
Where N is a number, e.g. 6:

```bash
git rebase -i HEAD~N
# N is the commit number, beginning from the most recent.
git commit --amend
git rebase --continue
git push --force
```

<h1>How to remove n last commits?</h1>

Where N is a number, e.g. 6:

```bash
git reset HEAD~N
git push origin +HEAD
```

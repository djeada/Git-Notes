TODO: 
* add undo for commit 

<h1>Staging files</h1>

Stages the file, ready for commit:

```bash
git add filename
```

Stage all changed files, ready for commit:

```bash
git add .
```

<h1>How to undo git add?</h1>

Unstages file, keeping the file changes:

```bash
git reset filename
```

Revert everything to the last commit:

```bash
git reset --hard
```

<h1>What are the commits?</h1>

Commits are similar to saves.
To capture the present status of your project, commit files from the staging area by associating them with a message.
The commit message should indicate what modifications have been done since the last commit.
If you wish to go back to a prior state of your project that was connected with a commit, you may simply do so with the <i>checkout</i> command. 

Commit all staged files to versioned history:

```bash
git commit -m “commit message”
git push
```

Restore project state from commit with id commit_id:

```bash
git checkout commit_id
```

<h1>How to edit the commit message after a git push?</h1>

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

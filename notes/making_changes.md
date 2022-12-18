## Staging files

To stage a file for commit, use the git add command followed by the file name:

```bash
git add filename
```

To stage all changed files for commit, use:

```bash
git add .
```

### Undoing git add

To unstage a file and keep the changes, use:

```bash
git reset filename
```

To revert everything to the last commit, use:

```bash
git reset --hard
```

## Committing files

Commits are similar to saves in that they capture the current state of your project and associate it with a message. The commit message should describe the changes made since the last commit. You can use the git checkout command to go back to a previous state of your project that was captured in a commit.

To commit all staged files to the version history, use:

```bash
git commit -m "commit message"
git push
```

To restore the project state from a commit with the ID commit_id, use:

```bash
git checkout commit_id
```

### Undoing commits that have not been pushed

To undo the last commit and keep all files staged, use:

```bash
git reset --soft HEAD~
```

To undo the last commit and unstage all files, use:

```bash
git reset HEAD~
```

To completely remove all changes made in the last commit, use:

```bash
git reset --hard HEAD~
```

### Editing commit messages

To edit the message of the last commit:

```bash
git commit --amend -m "new message"
git push --force
```

To edit the message of an old commit (where N is the number of commits back, e.g. 6):

```bash
git rebase -i HEAD~N
# N is the commit number, starting from the most recent.
git commit --amend
git rebase --continue
git push --force
```

### Removing commits

To remove the last N commits (where N is the number, e.g. 6):

```bash
git reset HEAD~N
git push origin +HEAD
```

### Removing a file from commit history

To remove a file from the commit history, use:

```bash
git filter-branch --index-filter 'git rm -r --cached --ignore-unmatch file_to_remove' HEAD
```

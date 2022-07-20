## Staging files

Stages the file, ready for commit:

```bash
git add filename
```

Stage all changed files, ready for commit:

```bash
git add .
```

## How to undo git add?

Unstages file, keeping the file changes:

```bash
git reset filename
```

Revert everything to the last commit:

```bash
git reset --hard
```

## What are the commits?

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

## Undo a commit that has not been pushed yet

To undo commit and keep all files staged, use: 

```bash
git reset --soft HEAD~
```

To undo commit and unstage all files, use: 

```bash
git reset HEAD~
```

To undo the commit and completely remove all changes, use: 

```bash
git reset --hard HEAD~
```

## How to edit the commit message after a git push?

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

## How to remove n last commits?

Where N is a number, e.g. 6:

```bash
git reset HEAD~N
git push origin +HEAD
```

## How to remove a file from commit history?

```bash
git filter-branch --index-filter 'git rm -r --cached --ignore-unmatch file_to_remove' HEAD
```

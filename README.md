# Git
Notes on git

TODO:
* merge tool
* change commit message
* squash last 5 commits
* remove commit

<h1> How to merge to master </h1>

```bash
git push origin feature_branch
git checkout master
git pull origin master
git merge feature_branch
git push origin master
```

<h1> How to see what branches are merged/not merged? </h1>

```bash
git checkout master 
git branch --merged
git branch --no-merged
```

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

<h1>Fetch a file from another branch without changing the current branch.</h1>

```bash
git checkout other_branch -- path/to/file 
```

<h1> How to remove branch feature?</h1>

```bash
git brach -d feature
git push origin --delete feature
```

<h1>Show word changes in diff</h1>

```bash
git diff --word-diff 
```

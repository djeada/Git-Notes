Get the latest changes from origin (no merge):

git fetch

Fetch the latest changes from origin and merge:

git pull

Fetch the latest changes from origin and rebase: 

git pull --rebase

Push local changes to the origin:

git push

Rebasing with Merge Conflicts
git pull --rebase


<h1>Difference between fetch and pull</h1>

<h1>Fetch a file from another branch without changing the current branch.</h1>

```bash
git checkout other_branch -- path/to/file 
```


TODO: mergetool

<h1>Merge conflicts</h1>

If merge conflicts, read the super-helpful tips in terminal:

1. git diff to resolve the merge conflicts I have
2. git rebase --continue


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

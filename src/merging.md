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

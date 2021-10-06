TODO: mergetool

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

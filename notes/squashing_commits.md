## What is squashing?

Squashing is the process of combining multiple commits into a single commit.

## Squashing vs merging

Merging is reserved for branches.
Squashing is used to combine two or more commits that are usually on the same branch.
Common practice is to make many smaller commits for yourself and then combine them into larger ones with more informative messages before merging with the *dev* or *master*. 

## Squashing N last commits

To combine 3 last commits into a single commit and write a new commit message from scratch, use:

```bash
git rebase -i HEAD~3 
```

There is also an option to use <code>merge</code> command instead of <code>rebase</code>:

```bash
git reset --hard HEAD~3
git merge --squash HEAD@{1}
git commit
```

An alternative way (be cautious since force pushing might cause problems while working with multiple people on the same project):

```bash
git reset --soft HEAD~3
git commit
git push -f
```

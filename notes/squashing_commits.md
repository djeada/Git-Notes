## Squashing N last commits

Squashing is the process of combining multiple commits into a single commit: 

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

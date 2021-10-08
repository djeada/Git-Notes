
Squashing is the process of combining multiple commits into a single commit: 

To combine 3 last commits into a single commit and write a new commit message from scratch, use:

```
git rebase -i HEAD~3 
```

An alternative way:

```bash
git reset --soft HEAD~3
git commit
```

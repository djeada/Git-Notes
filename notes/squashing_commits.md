## What is squashing?

Squashing is the process of combining multiple commits into a single commit in a Git repository. It is often used to clean up a branch or history before merging it into the main branch (such as master).

## Squashing vs merging

Merging is the process of taking the changes from one branch and applying them to another branch. Squashing is the process of combining multiple commits into a single commit. While merging is typically used to bring changes from one branch into another branch, squashing is used to condense the commit history of a branch. It is common practice to make many smaller commits for oneself while working on a branch and then squash them into larger, more informative commits before merging the branch into `dev` or `master`.

## Squashing the last N commits

To combine the last 3 commits into a single commit and write a new commit message from scratch, use the following command:

```bash
git rebase -i HEAD~3
```

Alternatively, you can use the merge command instead of rebase:

```bash
git reset --hard HEAD~3
git merge --squash HEAD@{1}
git commit
```

Note: Be cautious when using the following alternative method, as force pushing can cause problems when working with multiple people on the same project:

```bash
git reset --soft HEAD~3
git commit
git push -f
```

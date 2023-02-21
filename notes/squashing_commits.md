# Git Squashing

When working with Git, it's common to have a branch with multiple small commits that are used to develop new features or fix bugs. However, when it comes time to merge these changes into the main branch, it can be helpful to condense those multiple commits into a single, more informative commit. This is where squashing comes in.

## What is Squashing?

Squashing is the process of combining multiple commits into a single commit in a Git repository. It is often used to clean up a branch or history before merging it into the main branch, such as dev or master. Squashing is not a necessary step, but it can make it easier to understand the changes made in a branch and can also make the commit history cleaner and more concise.

## Squashing vs Merging

While merging is typically used to bring changes from one branch into another branch, squashing is used to condense the commit history of a branch. Merging applies the changes from one branch onto another, while squashing combines the changes from multiple commits into one. It is common practice to make many smaller commits for oneself while working on a branch and then squash them into larger, more informative commits before merging the branch into the main branch.

## Squashing the last N Commits

To combine the last N commits into a single commit and write a new commit message from scratch, use the following command:

```bash
git rebase -i HEAD~N
```

Alternatively, you can use the merge command instead of rebase:

```bash
git reset --hard HEAD~N
git merge --squash HEAD@{1}
git commit
```

Note: Be cautious when using the following alternative method, as force pushing can cause problems when working with multiple people on the same project:

```bash
git reset --soft HEAD~N
git commit
git push -f
```

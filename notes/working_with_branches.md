## Git branching

Git branches are an essential tool for managing different versions of your codebase and for collaborating with others. In this section, we'll cover the basics of Git branches and how to use them effectively.

### What are branches?

In Git, a branch is a pointer to a specific commit in the repository's history. When you create a new branch, Git creates a new pointer and adds it to the repository's history. This allows you to work on a specific feature or bug fix without affecting the main codebase (typically referred to as the master branch).

For example, consider a repository with the following commit history:

```bash
A <-- B <-- C <-- D
```

Here, the master branch is a pointer to the most recent commit (D). If we create a new branch at this point, Git will create a new pointer and add it to the history:

```bash
A <-- B <-- C <-- D  <-- new_branch
            ^
            |
            master
```

Now, new_branch is a separate branch with its own pointer, and any new commits will be added to this branch's history instead of the master branch's history.

## How do branches work technically?

In Git, every commit has one or more parent commits, forming a directed acyclic graph (DAG) of snapshots. This means that each commit refers to one or more previous commits, allowing Git to track the history of the repository.

For example, consider a repository with the following commit history:

```bash
A <-- B <-- C <-- D
```

Here, A is the first commit in the repository, and B, C, and D are subsequent commits with A as their parent.

If we create a new branch at this point and make a new commit, the history will look like this:

```bash
A <-- B <-- C <-- D  <-- new_branch
            ^
            |
            master
```

Here, the new commit (new_branch) is added to the new_branch branch's history, but not to the master branch's history. This allows us to work on new features or bug fixes without affecting the main codebase.

## When to create branches

There are many situations where creating a new branch can be helpful. Some common scenarios include:

### Bug fixes
If you discover a bug in your code, creating a new branch allows you to work on a fix without affecting the main codebase. This is especially useful if you're not sure how long it will take to fix the bug or if the fix might break other parts of the code.

### New features
If you're working on a new feature for your codebase, creating a new branch allows you to develop the feature without affecting the main codebase. This is useful if the feature is still in the early stages of development or if you're not sure if it will be successful.

### Long-lived branches
In many projects, there are typically a few long-lived branches that are used for specific purposes. For example, the master branch is often used for the project's main codebase, while a dev branch is used for integrating various short-lived branches.

### Continous integration
Many development teams practice continuous integration, which involves regularly merging short-lived branches into the main codebase (often at least once per day, and sometimes several times per day). 

Remember that a few lines of code that can be promptly merged into the master branch are preferable than a complete feature that will be held on a separate branch for months only to be forgotten and never merged. 

## Listing branches

To see a list of all the branches in your repository, you can use the git branch command. This will show a list of all the local branches in the repository, with the current branch marked with an asterisk (*).

For example:

```bash
$ git branch
  branch1
* branch2
  branch3
```

To see a list of all branches, including remote branches, you can use the -a flag:

```bash
$ git branch -a
  branch1
* branch2
  branch3
  origin/branch1
  origin/branch2
  origin/branch3
```

## Creating branches

To create a new branch, use the git branch command followed by the name of the new branch. For example:

```bash
git branch new_branch
```

This will create a new branch called new_branch, but it will not switch you to that branch. To switch to the new branch, you can use the git checkout command:

```bash
git checkout new_branch
```

You can also create a new branch and switch to it in one step using the git checkout -b command:

```bash
git checkout -b new_branch
```

## Switching branches

To switch to an existing branch, use the git checkout command followed by the name of the branch you want to switch to. For example:

```bash
git checkout branch1
```

This will switch you to the branch1 branch, and any new commits you make will be added to this branch's history.

## Merging branches

To merge one branch into another, first switch to the branch you want to merge into, then use the git merge command followed by the name of the branch you want to merge. For example, to merge branch1 into branch2, you would do the following:

```bash
git checkout branch2
git merge branch1
```

This will create a new commit that combines the changes from both branches. If there are conflicts between the two branches, Git will ask you to resolve them before the merge can be completed.

## Deleting branches

To delete a branch, use the git branch -d command followed by the name of the branch you want to delete. For example:

```bash
git branch -d branch1
```

This will delete the branch1 branch, but only if it has already been merged into another branch. If the branch has not been merged, you will need to use the -D flag instead:

```bash
git branch -D branch1
```

## Best practices

When working with branches, it's important to follow a few best practices to keep your repository organized and easy to work with:

* Keep branches short-lived: Long-lived branches can be difficult to merge and can lead to conflicts. Try to keep your branches short and merge them into the main codebase as soon as possible.
* Merge often: Regularly merging your branches into the main codebase helps to avoid conflicts and makes it easier to integrate changes from multiple branches.

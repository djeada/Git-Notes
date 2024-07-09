## Git Squashing

In the world of Git, the iterative development process often results in multiple commits for minor changes. But before merging changes to a primary branch, it's valuable to have a clean, linear history. This is where the concept of "squashing" steps in.

### What is Squashing?

At its core, squashing is about compressing several commits into one. This not only provides a concise view of the work done but also allows for a more streamlined commit history. While squashing can make the commit history neater, it is crucial to remember that squashing rewrites history, which can be problematic in shared branches.

```
Before Squashing:
-----------------

      master
        |
  A--B--C
        |
     feature
        |
  D--E--F--G
        |
     topic

After Squashing:
----------------

      master
        |
  A--B--C
        |
     feature
        |
        H
        |
     topic

Where:
D--E--F--G are combined into a single commit H.
```

### Squashing vs. Merging

The distinction between merging and squashing is vital when managing branches in a Git repository. Understanding the differences can help you choose the right method for incorporating changes.

- When performing a **merge**, the complete history of two branches is combined. This means that every commit from the feature branch is applied to the target branch, preserving a detailed history of all changes made in the feature branch.

- On the other hand, **squashing** involves compressing multiple commits into a single commit. Instead of having a detailed history of every small change, squashing results in one comprehensive commit that summarizes the changes. This approach simplifies the commit history, making it cleaner and easier to read, but it sacrifices the granular detail of individual commits.

### How to Squash the last N Commits

When it comes to squashing, the interactive rebase tool is your best friend:

I. **Using Rebase**:

```bash
git rebase -i HEAD~N
```

After this command, a text editor will open showing the last N commits. Simply change the word 'pick' to 'squash' (or 's' for short) for every commit you want to squash into the previous one. Save and close the editor, and Git will squash the specified commits.

II. **Alternative using Merge**:

If you're not a fan of rebasing, you can achieve a similar result using reset and merge:

```bash
git reset --hard HEAD~N
git merge --squash HEAD@{1}
git commit
```

III. **Force Pushing**:

Sometimes, after squashing locally, you need to update a remote branch. Do this with caution, as rewriting shared history can be disruptive to other collaborators:

```bash
git push origin branch_name --force
```

### A Word of Caution

Squashing, especially with force pushing, can be dangerous in collaborative environments. It's crucial to communicate with your team and ensure everyone understands the changes made to the shared branch's history.

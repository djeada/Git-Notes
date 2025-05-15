## Squashing

In Git, you might accumulate multiple small commits over the course of developing a new feature, fixing small bugs, or refactoring code. While these incremental commits are crucial during active development, they can clutter the project history in the long term. This clutter becomes especially evident when you merge your work into a primary branch like `main` or `master`.

**Squashing** is the process of compressing multiple commits into a single, consolidated commit. By doing so, you ensure that the branch’s history remains clean, succinct, and more understandable to collaborators who later need to review or track changes.

### Visualising a commit squash

Before squashing `feature` into a single commit:

```
main branch
│
A──B──C          ◄─ tip of main
    \
     D──E──F──G  ◄─ feature (4 commits)
```

The four commits **D → G** capture one logical feature but were made in stages.

One-liner to squash everything on `feature` into a single commit:

```bash
# run from the feature branch
git reset --soft $(git merge-base main HEAD) \
  && git commit --amend -m "feat: brief, imperative summary of the feature"
```

* `git merge-base main HEAD` finds the common ancestor (**C**) of `main` and `feature`.
* `reset --soft` moves `HEAD` back to **C** while keeping all changes staged.
* `commit --amend` creates the new squashed commit (**H**) with your message.
* If the branch is already on a remote: `git push --force-with-lease`.

After squashing:

```
main branch
│
A──B──C
    \
     H              ◄─ feature (single squashed commit)
```

* **H** now contains the combined changes from **D, E, F, G**.
* Write the commit message as a concise changelog entry for the entire feature (and reference issue IDs, e.g. `(#123)`).

### Potential Advantages of Squashing

- Squashing results in a **cleaner project history**, reducing the number of commits in `git log` and making it easier to understand the evolution of the project.
- It simplifies **code review** by presenting a single, summarized set of changes instead of multiple incremental commits.
- The process reduces **noise in the commit history**, combining minor fixes or corrections into a coherent and logical narrative of changes.

### Potential Drawbacks of Squashing

- Squashing can cause **loss of granularity**, removing detailed step-by-step commit messages that might contain important context or debugging information.
- Since squashing involves **history rewriting**, it can disrupt shared or public branches if others have already pulled or based their work on the original commit structure.

### Squashing vs. Merging

Before diving into the mechanics of squashing, it is essential to distinguish it from a standard merge. Both operations can bring changes from one branch into another, but the outcomes differ significantly in terms of commit history.

#### Merging

- A standard merge **preserves the complete history** by incorporating all commits from the source branch into the target branch, maintaining detailed records of all changes.
- It provides a **detailed timeline** in the commit log, showing how a feature evolved through experimental changes or incremental fixes.
- Merging is **simple and transparent**, requiring minimal effort to execute, making it a default approach in many Git workflows.

#### Squashing

- Squashing creates a **condensed commit** by combining a series of commits into a single, unified commit.
- It results in a **streamlined log** that removes clutter from fix-ups or partial changes, providing a clean summary of the branch's work.
- Squashing is **ideal for cleanup**, especially in open-source or large-scale projects where a tidy and professional commit history is preferred.

In short, **merging** preserves detailed commit-by-commit progress, while **squashing** condenses all those commits into a single “final” commit that captures the end result.

### How to Squash the Last N Commits

#### Using Interactive Rebase

The **interactive rebase** workflow is the most common and flexible method for squashing commits. It allows you to selectively rewrite commit history, adjust commit messages, reorder commits, and squash any subset of commits.

I. **Identify the Number of Commits**: Determine how many commits you want to squash. For example, if you have 4 commits in your current feature branch that you want to combine into 1 commit, note that number (N = 4).

II. **Run Interactive Rebase**:

```bash
git rebase -i HEAD~N
```

Replace **N** with the number of commits you want to rewrite. If you wanted to squash the last 4 commits, you’d use `git rebase -i HEAD~4`.

III. **Modify the Rebase To-Do List**:  

Your default text editor will open, showing something like:

```
pick a1b2c3d4 First commit message
pick e5f6g7h8 Second commit message
pick i9j0k1l2 Third commit message
pick m3n4o5p6 Fourth commit message
```

- Change `pick` to `squash` (or `s` for short) for the commits you want to squash **into** the commit above it.
- If you want to end up with just **one** commit, you typically mark the first commit as `pick` and the rest as `squash`.

**Example**:

```
pick a1b2c3d4 First commit message
squash e5f6g7h8 Second commit message
squash i9j0k1l2 Third commit message
squash m3n4o5p6 Fourth commit message
```

IV. **Save and Close the Editor**:

After you close the editor, Git will apply your rebase instructions. It will then open another editor window to let you customize the final combined commit message.  

**Tip**: Use this opportunity to write a clear and descriptive commit message that summarizes all the changes you’ve squashed together.

V. **Complete the Rebase**:

Once you save the final combined commit message, Git finishes the rebase. You can run:

```bash
git log
```

to verify that you now have a single commit in place of the previous multiple commits.

#### Alternative: Reset + Merge Squash

If you prefer not to use an interactive rebase, or you want a more manual approach:

I. **Reset to a Previous Commit**:

```bash
git reset --hard HEAD~N
```

This moves your branch pointer back by **N** commits, effectively discarding the last N commits from the branch tip (but they’re still in Git’s reflog for a while).

II. **Merge with the `--squash` Option**:

```bash
git merge --squash HEAD@{1}
```

`HEAD@{1}` references the commit your branch pointed to before the reset. The `--squash` option collects all the changes introduced in those commits without creating individual commits for each.

III. **Commit the Squashed Changes**:

```bash
git commit -m "Your new squashed commit message"
```

Now you have a single commit that represents all the changes from the last N commits.

### Dealing with Remote Repositories

If the commits you have squashed were already **pushed** to a remote repository, you will need to **force push** your rewritten history:

```bash
git push origin your-branch --force
```

Force pushing effectively replaces the remote branch with your local branch’s new commit structure. This can cause major confusion for anyone who has already pulled those original commits. They may encounter merge conflicts or will need to rebase their work.

**Therefore**, always coordinate with your team or ensure that no one else depends on the commit history you are about to rewrite. It might be safer to create a new branch that contains your squashed commit, then open a pull request from that branch, so you don’t disrupt existing references on the remote repository.

### Best Practices and Warnings

I. **Avoid Squashing on Shared or Public Branches**  

Once a branch is in use by multiple collaborators, rewriting its history can lead to conflicts and confusion. Squash your commits on personal feature branches or on branches that are clearly marked as “under development.”

II. **Provide Meaningful Squash Commit Messages**  

Since your new commit message is now the single source of documentation for multiple changes, put extra effort into writing a clear, descriptive summary of what changed and why.

III. **Use Squash Merges in Pull Requests**  

Many code hosting platforms like GitHub, GitLab, and Bitbucket offer a “Squash and Merge” button. This automatically squashes all commits from a pull/merge request into one commit, preserving the clean main-branch history without manual rebase steps.

IV. **Check Local Testing Before Force Pushing**  

After squashing, ensure your project still builds and all tests pass locally. The squashing itself shouldn’t break code, but always confirm to avoid introducing subtle errors or regressions.

V. **Communicate with Your Team**  

If you do need to force push, post a heads-up in your team’s communication channel, or coordinate a short time window for rewriting history so that no one else is pushing or pulling on that branch.

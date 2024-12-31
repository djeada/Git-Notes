## Squashing

In Git, you might accumulate multiple small commits over the course of developing a new feature, fixing small bugs, or refactoring code. While these incremental commits are crucial during active development, they can clutter the project history in the long term. This clutter becomes especially evident when you merge your work into a primary branch like `main` or `master`.

**Squashing** is the process of compressing multiple commits into a single, consolidated commit. By doing so, you ensure that the branch’s history remains clean, succinct, and more understandable to collaborators who later need to review or track changes.

**Key Advantages of Squashing**:

- **Cleaner Project History**: Fewer commits to scan through, making `git log` more readable.
- **Easier Code Review**: Reviewers see one summarized set of changes rather than multiple incremental commits.
- **Less Noise**: Minor fix-ups or revert commits get combined into a cohesive storyline of changes.
- 
**Potential Drawback**:
  
- **Loss of Granularity**: You lose the step-by-step commit messages that might have had valuable context.  
- **History Rewriting**: Squashing rewrites the Git commit history, so it can be disruptive in shared or public branches if teammates have already pulled or based their work on the original commits.

## 2. Squashing vs. Merging

Before diving into the mechanics of squashing, it is essential to distinguish it from a standard merge. Both operations can bring changes from one branch into another, but the outcomes differ significantly in terms of commit history.

### 2.1 Merging
- **Complete History Preservation**: When you perform a standard merge (e.g., `git merge feature-branch`), Git incorporates all commits from the source branch into the target branch, preserving each individual commit.  
- **Detailed Timeline**: Anyone looking at the commit log will see exactly how the feature evolved, including small experimental changes or incremental bug fixes.  
- **Simple and Transparent**: Merging is straightforward to execute and is often the default approach in many Git workflows.
### 2.2 Squashing
- **Condensed Commit**: Squashing transforms a series of commits into one unified commit.  
- **Streamlined Log**: Instead of a cluttered commit history with fix-ups and partial changes, you have a single commit that represents the entire set of changes that were developed on a branch.  
- **Ideal for Cleanup**: Often used in open-source or large corporate projects to keep the main branch tidy and professional.

In short, **merging** preserves detailed commit-by-commit progress, while **squashing** condenses all those commits into a single “final” commit that captures the end result.

## 3. How to Squash the Last N Commits

### 3.1 Using Interactive Rebase

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

### 3.2 Alternative: Reset + Merge Squash

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


## 4. Dealing with Remote Repositories

### 4.1 When to Force Push

If the commits you have squashed were already **pushed** to a remote repository, you will need to **force push** your rewritten history:

```bash
git push origin your-branch --force
```

Force pushing effectively replaces the remote branch with your local branch’s new commit structure. This can cause major confusion for anyone who has already pulled those original commits. They may encounter merge conflicts or will need to rebase their work.

**Therefore**, always coordinate with your team or ensure that no one else depends on the commit history you are about to rewrite. It might be safer to create a new branch that contains your squashed commit, then open a pull request from that branch, so you don’t disrupt existing references on the remote repository.

## 5. Best Practices and Warnings

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

## 6. Visualizing Squashing with an Example

Before Squashing:

```
master branch
    |
A---B---C   <--- Tip of master
        \
         D---E---F---G   <--- feature branch with multiple commits
```

The `feature` branch has four commits (D, E, F, G) that collectively represent the development of a single feature.

After Squashing:

```
master branch
    |
A---B---C
        \
         H   <--- feature branch now has a single squashed commit
```

- Commits D, E, F, and G are now replaced by a single commit H.  
- The resulting commit message for H should outline what was done in the entire feature.

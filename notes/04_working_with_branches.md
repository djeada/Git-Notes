## Git branching

Git branches help you keep different versions of your codebase separate and make collaboration smoother. Think of them like parallel timelines for your project. By creating a branch, you're effectively saying, “I want to try something new without messing up the main timeline.”

Below, we’ll walk through what branches are, how they work, why and when to create them, and how to manage them with some straightforward Git commands.

```
     .----( feature_x )----.
    /                       \
---A------B------C------D----E---( main )
          \                 /
       '----( fix_bug )-----'
```

Above is a sketch showing a simplified idea: multiple branches coming off a main history and potentially rejoining it after some work.

### What are branches?

A **branch** in Git is just a pointer to a specific commit. By default, Git starts with a main branch (often called `master` or `main`). Whenever you make a new commit, Git moves that branch pointer forward to track your latest changes.

When you create a new branch, you’re simply telling Git to mark a specific commit as a “starting point” for a separate line of development. Any commits you make in that branch won’t appear on the main branch until you decide to merge them back.

For instance, consider a history like this:

```
A <-- B <-- C <-- D  (master)
```

If you create a new branch at commit `D`, Git sets up a pointer for that branch:

```
A <-- B <-- C <-- D
                ^
                |------( new_branch )
                
(master also points to D)
```

From here, any commits you make on `new_branch` won’t affect `master` until a merge happens.

### How do branches work technically?

Under the hood, Git uses commits that form a directed acyclic graph (DAG). Each commit knows its parent commit(s). When you branch, you’re just adding a new pointer to one of those commits.

If you make a commit on that new branch, Git records the parent commit as the one you branched from, forming a new path:

```
A <-- B <-- C <-- D (master)
                  \
                   E (new_branch)
```

Here, `E` is a new commit. It only exists on `new_branch` unless or until you merge it back into `master`.

### When to create branches

A common question is: **“When should I branch out?”**. Here are some typical scenarios:

#### Bug fixes
When you spot a bug, it’s often a good idea to create a new branch to fix it. You can then test the fix thoroughly without breaking the main code. If you’re not sure how long a fix will take (or it might cause unexpected side effects), isolating your changes on a new branch is safer.

#### New features
If you’re adding a new feature, do the work on a feature branch. This helps keep the main branch stable. You can experiment freely and only merge when your feature is ready to share with everyone else.

#### Long-lived branches
Some teams maintain long-running branches for major releases or specific versions. The `master` (or `main`) branch is usually considered stable, while a `dev` branch might be used to integrate features from multiple short-lived branches before they’re merged to `master`.

#### Continuous integration
Teams often merge code into the main branch frequently—sometimes daily or even multiple times per day. The goal here is to keep the code in a “working” state and prevent large, unmerged branches from drifting too far behind. Smaller, more frequent merges reduce the risk of huge merge conflicts later.

```
  (short_feature_branch)
        /
---A----B----C----D----E---(master)
                \
                 F----G----(another_feature_branch)
```

The shorter your branch’s lifespan, the smoother the merge back into the main line typically is.

### Listing branches

**To list all local branches**, you can run:

```bash
git branch
```

Git will print something like:

```bash
  branch1
* branch2
  branch3
```

The asterisk (`*`) shows the branch you’re currently on.

**To list all branches (local and remote)**, use:

```bash
git branch -a
```

Output example:

```bash
  branch1
* branch2
  branch3
  remotes/origin/branch1
  remotes/origin/branch2
  remotes/origin/branch3
```

Remote branches typically show up as `remotes/origin/...`, indicating branches that live on the remote repository.

### Creating branches

**To create a new branch**, use:

```bash
git branch new_branch
```

This creates the branch `new_branch` but **doesn’t** switch you to it. If you immediately check `git branch`, you’ll see `new_branch` listed, but it won’t have the asterisk next to it. To switch to that branch, run:

```bash
git checkout new_branch
```

Alternatively, you can create and switch in one go:

```bash
git checkout -b new_branch
```

That command says, “Make a new branch called `new_branch` and check it out right away.”

### Switching branches

Once a branch exists, you switch to it with:

```bash
git checkout branch1
```

Any commits you make now will belong to `branch1`. If you switch back to `master` (`git checkout master`), you’ll return to the previous state of the code in that branch.

### Merging branches

When your work in a branch is ready to join another branch (often `master`), you’ll merge. **First**, switch to the branch you want to merge **into**:

```bash
git checkout branch2
```

Then run:

```bash
git merge branch1
```

This tells Git: “Take all the commits from `branch1` and merge them into `branch2`.” If everything goes smoothly, Git will create a new “merge commit.” Sometimes you’ll encounter conflicts if both branches changed the same lines in different ways. If that happens, Git will pause the merge and let you resolve the conflicts manually in the affected files.

Here’s an example merge flow and its typical output:

```bash
$ git checkout master
Switched to branch 'master'

$ git merge new_branch
Updating d34f00d..a1b2c3
Fast-forward
 file1.txt | 2 +-
 file2.txt | 1 +
 2 files changed, 3 insertions(+), 1 deletion(-)
```

In the output above, Git performed a **fast-forward** merge, which just moves the branch pointer forward because `master` was directly behind `new_branch`.

### Deleting branches

Once you’ve merged a branch (or if you decide it’s no longer needed), you can delete it:

```bash
git branch -d branch1
```

Git will warn you if the branch hasn’t been merged. In that case, you can force-delete with:

```bash
git branch -D branch1
```

But beware: forcing a delete can throw away commits that aren’t merged anywhere else. Make sure you don’t lose important work.

### Best practices

When working with branches, it's important to follow a few best practices to keep your repository organized and easy to work with:

* Keep branches short-lived: Long-lived branches can be difficult to merge and can lead to conflicts. Try to keep your branches short and merge them into the main codebase as soon as possible.
* Merge often: Regularly merging your branches into the main codebase helps to avoid conflicts and makes it easier to integrate changes from multiple branches.

## Understanding HEAD

`HEAD` is Git’s pointer to the snapshot you’re currently working on—the **bookmark of your checkout**. Most of the time, `HEAD` points to the tip of a branch (like `master` or `main`). When you commit, `HEAD` (and that branch) advance to the new commit.

* Under the hood, `HEAD` is a tiny text file: `.git/HEAD`. Running `cat .git/HEAD` might show `ref: refs/heads/master`, meaning `HEAD` points to the branch named `master`. If it shows a raw commit hash instead, `HEAD` is detached (explained below).
* Switching branches (via `git switch` or `git checkout`) moves `HEAD` to point at the target branch. Knowing where `HEAD` points makes branching, merging, and navigating history far more predictable.

```
Time →                                                     (commits left→right)

                 ┌───────────── Merge ──────────────┐
main        ──●──┼──●───────────●───────────●───────●────────▶
               A1    A2         M1          A3     A4     (branch tip: main)
                 \               ^                   \
                  \              │                    \
feature/login      ●───●───●─────┘                     ●─────●──▶
                  F1   F2  F3                          M2    F4   (HEAD -> feature/login)

hotfix/urgent           ●────●─────────────────────────┘
                       H1   H2

Tags/Remotes:
  (origin/main) at A3     (v1.2) tag at M1

Legend:
  ●   = commit node
  A*, F*, H* = commit short IDs (on main, feature/login, hotfix/urgent)
  M*  = merge commits
  HEAD -> <branch> means your working tree points to that branch’s tip
```

* `HEAD -> feature/login` (on the **feature/login** row) tells you your current checkout is at **F4**. New commits will extend **feature/login**.
* If you `git switch main` (or `git checkout main`), `HEAD` moves to **A4**, and the indicator becomes `HEAD -> main`.

### Detached HEAD

A **detached HEAD** is when `HEAD` points **directly to a commit** instead of to a branch tip—usually after checking out a specific commit hash or a tag.

* In a detached state, any new commits aren’t on a named branch. They’re easy to lose if you move `HEAD` elsewhere because nothing points to them.
* If you commit while detached and want to keep that work, attach it to a branch: create one at that commit (`git switch -c my-work`) or merge/cherry-pick it into an existing branch. (You can often recover recent detached commits with `git reflog` if you move away accidentally.)

This is what it looks like when you check out a specific commit:

```
main        ──●──●──●──●──●──▶
               A1  A2  A3  A4

(HEAD) ----------------^
                     (checked out at commit A3, no branch name)
```

Here `HEAD` points to **A3** itself, not a branch. New commits would form an orphaned line unless you first make a branch, e.g. `git switch -c temp-work`.

### What Happens When You Commit on a Detached HEAD

If you commit (F) while `HEAD` is detached at commit C, history forks:

```
A <-- B <-- C (HEAD) <-- F
             \
              D <-- E (master)
```

Commit **F** is **not** on `master`. If you switch back to `master`, there’s no branch reference to **F** unless you merge, cherry-pick, or create a branch pointing to it. If left unattached, **F** can be garbage-collected once unreachable.

### Detaching HEAD: Switching to a Specific Commit

You enter a detached HEAD state by checking out a commit hash or tag. For example:

```bash
git switch b4d373k8990g2b5de30a37bn843b2f51fks2b40
```

Or equivalently:

```bash
git checkout b4d373k8990g2b5de30a37bn843b2f51fks2b40
```

`HEAD` now points to that exact commit rather than a branch tip. Git will say:

```
Note: switching to 'b4d373k8990g2b5de30a37bn843b2f51fks2b40'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.
```

That’s your cue: commits made now live off to the side until you attach them to a branch (e.g., `git switch -c keep-this`).

### Creating a Branch from a Detached HEAD

Sometimes you realize you’ve made valuable commits while detached and want to keep them. The safe fix is to create a branch **at the commit you’re on** and then switch to it.

```bash
git branch new_branch
```

Then switch to it:

```bash
git switch new_branch
```

Now `new_branch` contains all commits you made while `HEAD` was detached, and you’re no longer detached—`HEAD` points to the branch tip again. Your work is preserved and anchored to `new_branch`.

```
Before:
A <-- B <-- C (HEAD) <-- F

After creating and switching to new_branch:
A <-- B <-- C <-- F (new_branch, HEAD)

F is no longer “floating”; it’s anchored to the new_branch branch.
```

*Tip:* You can do this in one step with `git switch -c new_branch`. Verify where you are with `git status` or `git branch --show-current`.

### Preventing a Detached HEAD State

It’s cleaner to avoid a detached `HEAD` unless you’re only **looking**. If you intend to make changes, create a branch at that commit first so new work has a name and a home.

**Example Scenario**: You want to check out an older commit to debug or reproduce behavior.

I. Create a branch named `old_version` at that commit:

```bash
git branch old_version b4d373k8990g2b5de30a37bn843b2f51fks2b40
```

II. Switch to `old_version`:

```bash
git switch old_version
```

Now, any commits you make stay on `old_version`—no work drifts unreferenced.

*Tip:* One-step variant: `git switch -c old_version b4d373k8990g2b5de30a37bn843b2f51fks2b40`.

### Switching Back to a Branch from a Detached HEAD

When you’re done exploring a detached commit, just jump back to a normal branch (like `master`):

```bash
git switch master
```

or

```bash
git checkout master
```

You’ll be right where `master` last pointed. If you made commits while detached and haven’t put them on a branch yet, do so **before** switching—or recover them later via `git reflog` and then create a branch to keep them.

### Merging Changes from a Detached HEAD into a Branch

You’ve made useful commits while detached and want them on `master`. The trick is to merge **the commit that contains your work** (usually the latest one on that detached line) into `master`.

I. Switch to `master` (the branch you want to bring changes into):

```bash
git switch master
```

(Or `git checkout master`.)
*Tip:* If your default branch is `main`, substitute accordingly.

II. Merge the detached commit (use the **tip** commit’s hash):

```bash
git merge b4d373k8990g2b5de30a37bn843b2f51fks2b40
```

This creates a merge commit tying your detached line into `master`. You’re effectively stitching that “loose thread” back into the main history. If Git reports conflicts:

```bash
git status               # see which files need attention
# resolve conflicts in your editor
git add <file_with_conflicts>
git merge --continue     # or: git commit
```

Now your detached changes are part of `master`, recorded with a merge commit.

```
(master)                (HEAD detached)
   |                          |
   D <-- E                   C <-- F
    \                      /
     ------ Merge Commit --

Result: a single continuous history where F’s changes are integrated into master.
```

*Notes*

* Merging a specific commit brings in that commit **and its unique history** up to that point.
* If you only want a subset of commits from the detached line, use `git cherry-pick <hash>` instead.

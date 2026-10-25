# Branches, HEAD, Tags and Stash Quiz

Notes: [Working with branches](../notes/04_working_with_branches.md), [HEAD](../notes/05_head.md), [Tags](../notes/06_tags.md), [Stashing files](../notes/07_stashing_files.md)

Tests creating, switching, merging, rebasing and deleting branches, how HEAD and revision syntax work, lightweight versus annotated tags, and saving work with the stash.

#### Q. You are on `main` and run the commands below. Which branch is checked out afterwards?

```bash
git branch feature
git branch
```

* [x] `main`, because `git branch feature` only creates the branch
* [ ] `feature`, because a new branch is checked out automatically
* [ ] Neither, because HEAD is detached at the new branch's commit
* [ ] `feature`, but only after the first commit is made on it

> **Explanation:** `git branch <name>` creates a pointer at the current commit and leaves HEAD where it was, so `git branch` still marks `main` with the asterisk. Use `git switch -c` to create and switch in one step.

#### Q. Which single command creates a branch called `login` at the current commit and switches to it?

* [ ] `git branch -c login`
* [ ] `git switch login`
* [x] `git switch -c login`
* [ ] `git checkout login`

> **Explanation:** `git switch -c` (like `git checkout -b`) creates and checks out the branch. `git switch login` and `git checkout login` expect an existing branch, and `git branch -c` copies the current branch without switching to it.

#### Q. You want to look at an old commit and run the command below. What happens?

```bash
git switch 1332864a242db6b0831408a94b979696ebfe313e
```

* [ ] HEAD is detached at that commit and a warning is printed
* [ ] A new branch named after the commit hash is created
* [ ] The current branch is reset to point at that commit
* [x] It fails with `fatal: a branch is expected, got commit`

> **Explanation:** `git switch` only switches to branches unless you pass `--detach`, so this fails with a hint to retry using `git switch --detach <commit>`. `git checkout <commit>` would detach HEAD without asking.

#### Q. You are on `main`. Branch `feat` has one commit that has not been merged anywhere. What does `git branch -d feat` do?

* [ ] Deletes `feat` and prints a warning about the lost commit
* [ ] Deletes `feat` and moves its commit onto `main`
* [x] Refuses with `the branch 'feat' is not fully merged`
* [ ] Asks interactively whether to delete the unmerged commit

> **Explanation:** `-d` is the safe delete: it exits with an error and keeps the branch when its commits are not merged. Only `git branch -D feat` (force) deletes it regardless.

#### Q. You force-deleted a branch by mistake. How do you get it back?

```text
$ git branch -D lost
Deleted branch lost (was 1cd5425).
```

* [ ] `git branch -u lost 1cd5425`
* [x] `git branch lost 1cd5425`
* [ ] `git switch lost`
* [ ] `git stash branch lost`

> **Explanation:** Deleting a branch removes only the name; the commit still exists until it is garbage-collected. Recreating a branch at the hash Git printed (or one found with `git reflog`) restores it. `git switch lost` fails because the name no longer exists.

#### Q. Branch `feature` has been merged into `dev` but not into `main`. You are on `dev` and `feature` has no upstream. What does `git branch -d feature` do?

* [ ] Refuses, because `-d` only accepts branches merged into `main`
* [ ] Refuses, because `-d` cannot delete a branch with no upstream
* [ ] Deletes the branch and also removes its commits from `dev`
* [x] Deletes the branch, because its commits are reachable from HEAD

> **Explanation:** Without an upstream, `git branch -d` checks whether the branch is merged into the current HEAD, not into `main`. Its commits stay in `dev`; only the branch name is removed.

#### Q. You are on `main` and run `git branch -D main`. What happens?

* [ ] `main` is deleted and HEAD becomes detached at its last commit
* [x] Git refuses because `main` is the branch currently checked out
* [ ] `main` is deleted and Git switches to the most recent other branch
* [ ] `main` is deleted only if it is fully merged into its upstream

> **Explanation:** Git will not delete the branch checked out in a worktree, even with `-D`; it reports `cannot delete branch 'main' used by worktree`. Switch to another branch first.

#### Q. In a fresh clone, `git branch` lists only `main`, but `origin/feature` exists. What does `git switch feature` do?

* [x] Creates a local `feature` branch that tracks `origin/feature`
* [ ] Fails with `invalid reference: feature` because no local branch exists
* [ ] Detaches HEAD at `origin/feature` so the remote branch can be inspected
* [ ] Checks out `origin/feature` itself so new commits update the remote

> **Explanation:** When exactly one remote has a matching branch, `git switch <name>` creates a local branch from it and sets up tracking (`branch 'feature' set up to track 'origin/feature'`). Remote-tracking branches themselves are never committed to directly.

#### Q. You have unsaved edits in `notes.txt` and there is no branch called `notes.txt`. You meant to type a branch name. What do these two commands do?

```bash
git switch notes.txt
git checkout notes.txt
```

* [ ] Both fail with `invalid reference: notes.txt`
* [ ] Both discard your uncommitted edits to `notes.txt`
* [ ] `switch` discards the edits; `checkout` fails with an error
* [x] `switch` fails with an error; `checkout` discards the edits

> **Explanation:** `git switch` only handles branches, so it reports `invalid reference`. `git checkout` falls back to treating the argument as a path and silently restores the file from the index, losing the edits. That ambiguity is why `switch` and `restore` were added.

#### Q. You edited `f` on `main`, and `f` is different on branch `other`. What happens when you run `git switch other`?

* [ ] Git switches and writes conflict markers into `f`
* [ ] Git switches and your edit to `f` is discarded
* [x] Git refuses because your local changes would be overwritten
* [ ] Git stashes your edit automatically before switching

> **Explanation:** If a file you modified differs between the two branches, Git aborts with `Your local changes to the following files would be overwritten by checkout`. Uncommitted edits to files that are identical on both branches are simply carried across.

#### Q. `main` has no new commits since `feature` was created. You run `git merge feature` on `main` and see the output below. What is true afterwards?

```text
Updating b4ae40f..9b2bb65
Fast-forward
 g | 1 +
 1 file changed, 1 insertion(+)
```

* [ ] A merge commit with two parents was added on top of `main`
* [ ] The commits from `feature` were copied onto `main` with new hashes
* [ ] `feature` was moved forward to match the tip of `main`
* [x] `main` now points at the same commit as `feature`

> **Explanation:** A fast-forward just moves the `main` pointer forward to `feature`'s tip; no new commit is created and no hashes change. Use `--no-ff` if you want a merge commit anyway.

#### Q. `main` has commit A, and `topic` adds commit B on top of A. On `main` you run the commands below. What is printed?

```bash
git merge --no-ff --no-edit topic
git rev-list --count HEAD
```

* [ ] `2`
* [ ] `1`
* [x] `3`
* [ ] `4`

> **Explanation:** A fast-forward was possible, but `--no-ff` forces a merge commit, so `main` now contains A, B and the merge commit M. Without `--no-ff` the count would be 2.

#### Q. `main` and `side` have diverged. What does `git merge --ff-only side` do on `main`?

* [x] Fails with `Not possible to fast-forward, aborting` and leaves `main` unchanged
* [ ] Creates a merge commit, because a fast-forward is impossible here
* [ ] Rebases `main` onto `side` so that the result is a fast-forward
* [ ] Moves `main` to `side`, discarding the commit that only `main` had

> **Explanation:** `--ff-only` merges only when `main` can simply move forward. With diverged history it aborts, and the hint suggests `git merge --no-ff` or `git rebase` instead.

#### Q. You are on `main`, run `git merge feature`, and `app.ini` now looks like this. Which statement is correct?

```text
<<<<<<< HEAD
color=green
=======
color=blue
>>>>>>> feature
```

* [ ] `color=green` is from `feature` and `color=blue` is from `main`
* [ ] `color=green` is the merge base and `color=blue` the merged result
* [x] `color=green` is from `main` and `color=blue` is from `feature`
* [ ] `color=green` is the staged copy and `color=blue` the working copy

> **Explanation:** The block after `<<<<<<< HEAD` is the version on the branch you are on (`main`), and the block before `>>>>>>> feature` is the incoming branch. Neither shows the merge base in the default conflict style.

#### Q. A merge stopped with a conflict in `app.ini`. What does `git status --short` show for that file?

* [ ] `MM app.ini`
* [ ] `M  app.ini`
* [ ] `AA app.ini`
* [x] `UU app.ini`

> **Explanation:** `UU` means unmerged, both modified. `AA` is only used when both sides added the same new file, and `M`/`MM` appear once the file has been resolved and staged.

#### Q. During a conflicted merge you run `git commit -m "merge"` before resolving anything. What happens?

* [ ] The merge commit is created with the conflict markers inside
* [x] It fails with `Committing is not possible because you have unmerged files`
* [ ] The merge commit is created using the `HEAD` version of each file
* [ ] Git aborts the merge and restores the pre-merge state

> **Explanation:** Git will not commit while the index still has unmerged entries. You must edit the files, `git add` them to mark them resolved, then `git commit` or `git merge --continue`.

#### Q. A merge on `main` has produced conflicts you do not want to deal with now. Which command puts the branch back exactly as it was before the merge?

* [ ] `git merge --continue`
* [ ] `git reset --soft HEAD`
* [ ] `git stash`
* [x] `git merge --abort`

> **Explanation:** `git merge --abort` discards the merge in progress, removes `MERGE_HEAD` and restores the pre-merge files. `git reset --soft` is refused in the middle of a merge, and `--continue` requires the conflicts to be resolved first.

#### Q. With `merge.conflictStyle=diff3`, a conflict looks like this. What is `color=red`?

```text
<<<<<<< HEAD
color=green
||||||| dd36785
color=red
=======
color=blue
>>>>>>> feature
```

* [x] The version in the common ancestor (merge base)
* [ ] The version currently staged in the index
* [ ] The version Git suggests as the resolution
* [ ] The version from the previous commit on `feature`

> **Explanation:** The `diff3` style adds a `|||||||` section showing the merge base. Seeing the original line makes it clear that both branches changed `red`, one to `green` and one to `blue`.

#### Q. During a merge you run `git add app.ini` while the file still contains `<<<<<<<` markers, then `git commit --no-edit`. What happens?

* [ ] `git add` refuses to stage a file that contains markers
* [x] The commit succeeds and the markers are committed
* [ ] `git commit` refuses because markers are still present
* [ ] Git strips the markers and keeps the `HEAD` version

> **Explanation:** Git treats `git add` as "this file is resolved" and does not inspect the content, so the merge commit records the markers. Always check the file (or `git diff --check`) before staging.

#### Q. On `feature` (commits F1 and F2), you run `git rebase main` after `main` gained commit M1. It succeeds. Which statement is true?

* [x] F1 and F2 are replaced by new commits with different hashes
* [ ] F1 and F2 keep their hashes; only their parent pointer is updated
* [ ] A merge commit joining `main` into `feature` is created
* [ ] `main` is moved forward to include F1 and F2

> **Explanation:** Rebase replays each commit on top of `main`, creating new commits because a commit's hash covers its parent. The old tip is still available as `ORIG_HEAD` or in the reflog; `main` itself does not move.

#### Q. You rebased `feature` onto `main` without conflicts. What happens when you then run `git merge feature` on `main`?

* [x] A fast-forward, because `feature` now sits directly on top of `main`
* [ ] A merge commit, because rebased branches always need a merge commit
* [ ] A conflict, because the rebased commits duplicate commits on `main`
* [ ] Nothing, because the rebase already added the commits to `main`

> **Explanation:** After the rebase, `main`'s tip is an ancestor of `feature`, so the merge just moves `main` forward. This is why rebasing before merging gives a linear history.

#### Q. On `feature` you run `git rebase main` and it stops with a conflict in `f`. `main` has `mainline` in `f`; your feature commit has `feat`. What does `f` contain after `git checkout --ours f`?

* [ ] `feat`, because "ours" is always the branch you started on
* [x] `mainline`, because during a rebase "ours" is the branch being rebuilt on
* [ ] The merge-base content, because "ours" means the common ancestor
* [ ] The file with conflict markers, because `--ours` only works for merges

> **Explanation:** A rebase checks out `main` and replays your commits onto it, so HEAD ("ours") is the `main` side and "theirs" is the commit being replayed. This is the reverse of what happens in a merge.

#### Q. A rebase of `feature` stopped with a conflict and you decide to give up. Which command returns `feature` to exactly the commits it had before the rebase?

* [ ] `git rebase --skip`
* [ ] `git merge --abort`
* [ ] `git rebase --continue`
* [x] `git rebase --abort`

> **Explanation:** `git rebase --abort` stops the rebase and resets the branch to its original tip. `--skip` drops the conflicting commit and continues, and `git merge --abort` fails because no merge is in progress.

#### Q. You run `cat .git/HEAD` first while on `main`, then again after `git switch --detach HEAD~1`. What do you see?

* [x] `ref: refs/heads/main`, then a raw 40-character commit hash
* [ ] A raw 40-character commit hash both times
* [ ] `ref: refs/heads/main` both times
* [ ] `ref: refs/heads/main`, then `ref: refs/heads/HEAD~1`

> **Explanation:** Normally HEAD is a symbolic reference to a branch. When detached, `.git/HEAD` holds the commit hash directly, which is why new commits there are not recorded on any branch.

#### Q. In a CI job that checked out a commit in detached HEAD state, what do these commands print?

```bash
git rev-parse --abbrev-ref HEAD
git branch --show-current
```

* [ ] The commit hash, then the commit hash
* [ ] `main`, then `main`
* [ ] An error, then an error
* [x] `HEAD`, then an empty line

> **Explanation:** With no branch checked out, `rev-parse --abbrev-ref HEAD` falls back to the literal `HEAD`, and `branch --show-current` prints nothing. Scripts that read the branch name this way need to handle both cases.

#### Q. While detached you made a commit, then ran `git switch main`. Git warned you were leaving one commit behind. How do you save it?

```text
$ git reflog -3
31d66d4 HEAD@{0}: checkout: moving from 1e9a71a9d56c499ecc094ec3446b465011df3367 to main
1e9a71a HEAD@{1}: commit: detached work
60158eb HEAD@{2}: checkout: moving from main to HEAD~1
```

* [ ] `git switch 1e9a71a`
* [x] `git branch rescue 1e9a71a`
* [ ] `git reset --hard HEAD@{0}`
* [ ] `git stash pop`

> **Explanation:** The reflog still records the detached commit, so creating a branch at `1e9a71a` gives it a permanent name. `git switch <hash>` fails without `--detach`, and `HEAD@{0}` is the current commit, so that reset does nothing.

#### Q. You are in detached HEAD state and have made two commits you want to keep. Which command keeps them on a new branch?

* [ ] `git switch keep`
* [ ] `git checkout -b keep main`
* [x] `git switch -c keep`
* [ ] `git branch keep main`

> **Explanation:** `git switch -c keep` creates the branch at the current (detached) commit and attaches HEAD to it. Giving `main` as the start point would create a branch that does not contain the two commits.

#### Q. HEAD is the merge commit M below. Which commit does `HEAD^2` refer to?

```text
*   M  (HEAD -> main) Merge branch 'side'
|\
| * S  (side) side
* | P  main2
|/
* B  feat1
* A  base
```

* [ ] P
* [ ] B
* [x] S
* [ ] A

> **Explanation:** `^2` selects the second parent of a merge commit, which is the branch that was merged in (S). `HEAD^` or `HEAD^1` would be the first parent, P.

#### Q. Using the same history, which commit does `HEAD~2` refer to?

```text
*   M  (HEAD -> main) Merge branch 'side'
|\
| * S  (side) side
* | P  main2
|/
* B  feat1
* A  base
```

* [ ] S
* [x] B
* [ ] P
* [ ] A

> **Explanation:** `~2` follows the first parent twice: M to P, then P to B. It is the same as `HEAD^^`, and it never goes through the second parent the way `HEAD^2` does.

#### Q. You commit B on top of A, then run `git reset --hard HEAD~1`. What does the last command print?

```bash
git log -1 --format=%s HEAD@{1}
```

* [ ] `A`
* [ ] Nothing, because the reset deleted B
* [ ] `fatal: ambiguous argument 'HEAD@{1}'`
* [x] `B`

> **Explanation:** `HEAD@{1}` means where HEAD was one move ago in the reflog, which was B before the reset. This differs from `HEAD~1`, the parent of the current commit.

#### Q. `feature` branched from `base` and has F1, F2, F3; `main` has since gained M1 and M2. Which commits does `git log --oneline main..feature` list?

* [ ] M2, M1
* [ ] F3, F2, F1, base
* [ ] All five of F1–F3 and M1–M2
* [x] F3, F2, F1

> **Explanation:** `A..B` lists commits reachable from B but not from A, so it shows what `feature` has that `main` lacks. Swapping the order (`feature..main`) would list M2 and M1.

#### Q. With the same history (F1–F3 only on `feature`, M1–M2 only on `main`), what does `git rev-list --count main...feature` print?

* [ ] `3`
* [ ] `2`
* [x] `5`
* [ ] `6`

> **Explanation:** For log-style commands, `A...B` is the symmetric difference: commits reachable from either side but not both, here F1–F3 plus M1–M2. Add `--left-right` to see which side each commit is on.

#### Q. You want to review only the changes `feature` made since it branched off, ignoring what landed on `main` meanwhile. Which command shows that?

* [ ] `git diff main..feature`
* [x] `git diff main...feature`
* [ ] `git diff feature..main`
* [ ] `git diff feature main`

> **Explanation:** For `git diff`, `main...feature` compares the merge base with `feature`. The two-dot form compares the two tips directly, so `main`'s new changes appear as reversed edits.

#### Q. You created `light` with `git tag light` and `v1.0` with `git tag -a v1.0 -m "Release 1.0"`. What do these commands print?

```bash
git cat-file -t light
git cat-file -t v1.0
```

* [x] `commit`, then `tag`
* [ ] `tag`, then `tag`
* [ ] `tag`, then `commit`
* [ ] `commit`, then `commit`

> **Explanation:** A lightweight tag is just a ref pointing straight at a commit. An annotated tag creates a separate tag object holding the tagger, date and message, which then points at the commit.

#### Q. A teammate says `git show v2.0` displays the commit but no `Tagger:` line and no tag message. What is the most likely reason?

* [x] `v2.0` is a lightweight tag, which has no tag object
* [ ] The tag was created locally and has not been pushed yet
* [ ] `git show` needs `--tags` to display tag metadata
* [ ] The tag points to a merge commit, which hides the message

> **Explanation:** Only annotated tags (`git tag -a`) store a tagger, date and message, which `git show` prints before the commit. A lightweight tag is just a name for the commit, so only the commit is shown.

#### Q. You tag a release with `git tag -a v1.2.0 -m "v1.2.0"` and then run `git push origin main`. What does the remote have?

* [ ] The updated `main` and the `v1.2.0` tag
* [x] The updated `main` but no `v1.2.0` tag
* [ ] The `v1.2.0` tag but not the updated `main`
* [ ] Neither, because the tag blocks the push

> **Explanation:** Pushing a branch does not push tags. Push the tag explicitly with `git push origin v1.2.0`, all tags with `git push origin --tags`, or annotated ones automatically with `--follow-tags`.

#### Q. HEAD has a lightweight tag `light2` and an annotated tag `v2`. Neither is on the remote. Which tags does `git push --follow-tags origin main` send?

* [x] Only `v2`
* [ ] Only `light2`
* [ ] Both `v2` and `light2`
* [ ] Neither of them

> **Explanation:** `--follow-tags` pushes annotated tags that point at commits being pushed, but ignores lightweight tags. That is why release tags should be annotated.

#### Q. You need to publish every local tag, including lightweight ones, to `origin`. Which command does that?

* [ ] `git push --all origin`
* [ ] `git push --follow-tags origin main`
* [x] `git push origin --tags`
* [ ] `git push origin main`

> **Explanation:** `--tags` pushes all refs under `refs/tags`, of either kind. `--all` pushes all branches but no tags, and `--follow-tags` skips lightweight tags.

#### Q. Tag `light` is already on the remote. You move it locally with `git tag -f light HEAD` and run `git push origin light`. What happens?

* [ ] The remote tag is updated to the new commit
* [ ] A second tag `light` is added beside the old one
* [ ] The remote tag is deleted and must be pushed again
* [x] The push is rejected because the tag already exists

> **Explanation:** Git refuses to overwrite an existing tag on the remote (`! [rejected] light -> light (already exists)`). Forcing it with `git push -f origin light` works, but moving a published tag breaks anyone who already fetched it.

#### Q. The only tag in your history is a lightweight tag `v0.9`, one commit behind HEAD. What does plain `git describe` do?

* [ ] Prints `v0.9-1-g<hash>`
* [x] Fails with `No annotated tags can describe`
* [ ] Prints `v0.9`
* [ ] Prints only the abbreviated commit hash

> **Explanation:** By default `git describe` only considers annotated tags. It suggests `--tags`, which would print `v0.9-1-g<hash>` (tag, commits since tag, abbreviated hash).

#### Q. A bug was found in release `v1.2.0`. You want to start a fix from exactly that release on a new branch. Which command does it?

* [x] `git switch -c hotfix v1.2.0`
* [ ] `git switch v1.2.0`
* [ ] `git branch hotfix`
* [ ] `git checkout v1.2.0 hotfix`

> **Explanation:** `git switch -c <branch> <start-point>` creates a branch at the tagged commit and switches to it. `git switch v1.2.0` fails without `--detach`, and `git branch hotfix` starts at the current HEAD instead of the tag.

#### Q. You modified the tracked file `app.js` and created a new file `untracked.txt`. After `git stash`, what does `git status --short` show?

* [x] `?? untracked.txt`
* [ ] ` M app.js`
* [ ] Nothing, because the working tree is clean
* [ ] `A  untracked.txt`

> **Explanation:** Plain `git stash` saves tracked changes (staged and unstaged) and leaves untracked files where they are. Use `git stash -u` to include them.

#### Q. You edited `app.js` and created an untracked `config.local`. `build/` is in `.gitignore`. You want to stash `app.js` and `config.local` but leave `build/` alone. Which command?

* [ ] `git stash -a`
* [x] `git stash -u`
* [ ] `git stash`
* [ ] `git stash --keep-index`

> **Explanation:** `-u` (`--include-untracked`) adds untracked files to the stash but skips ignored ones. `-a` (`--all`) would also stash the ignored `build/` directory, and plain `git stash` would leave `config.local` behind.

#### Q. On `main` you run `git stash push -m "wip login"`. What line does `git stash list` show for it?

* [ ] `stash@{0}: WIP on main: wip login`
* [ ] `stash@{0}: WIP on main: 687b321 wip login`
* [ ] `stash@{0}: On main: 687b321 Initial commit`
* [x] `stash@{0}: On main: wip login`

> **Explanation:** A stash with a message is listed as `On <branch>: <message>`. Without `-m`, Git uses `WIP on <branch>: <hash> <commit subject>` instead.

#### Q. You have one stash entry and run `git stash apply`. It applies cleanly. What does `git stash list` show now?

* [ ] Nothing, because applying a stash removes it
* [x] The same `stash@{0}` entry as before
* [ ] The entry renamed to `stash@{1}`
* [ ] The entry marked as applied

> **Explanation:** `apply` restores the changes but keeps the stash entry, so you can apply it elsewhere too. `git stash pop` is apply followed by drop, and `git stash drop` removes an entry on its own.

#### Q. You run `git stash pop` and it reports a conflict in `app.js`. What happened to the stash entry?

* [ ] It was dropped, so the conflicted file is the only copy left
* [ ] It was moved to a new branch named after the stash
* [x] It was kept, and Git says so in its output
* [ ] It was split into a resolved and an unresolved part

> **Explanation:** When `pop` hits a conflict, Git prints `The stash entry is kept in case you need it again`. After resolving, run `git stash drop` yourself.

#### Q. Before stashing, `git status --short` showed the output below. After `git stash` and then `git stash apply` (no flags), what does it show?

```text
M  app.js
 M lib.js
A  new.js
```

* [x] ` M app.js`, ` M lib.js`, `A  new.js`
* [ ] `M  app.js`, ` M lib.js`, `A  new.js`
* [ ] ` M app.js`, ` M lib.js`, `?? new.js`
* [ ] `M  app.js`, `M  lib.js`, `A  new.js`

> **Explanation:** Without `--index`, `apply` restores modifications to the working tree only, so the staged change to `app.js` comes back unstaged. A newly added file stays added. `git stash apply --index` would restore the original staging exactly.

#### Q. You have an uncommitted edit to `app.js`, and the newest stash also changes `app.js`. What does `git stash apply` do?

* [ ] Merges both edits and marks conflicting lines with markers
* [ ] Overwrites your edit with the stashed version of `app.js`
* [x] Refuses because your local changes would be overwritten
* [ ] Applies the stash and drops it, keeping both edits

> **Explanation:** Git will not merge a stash into uncommitted changes to the same file; it aborts with `Your local changes to the following files would be overwritten by merge` and the stash stays. Commit or stash your edit first.

#### Q. You stashed work on `main`, then `main` moved on and popping now conflicts. What does `git stash branch fix stash@{0}` do?

* [ ] Creates `fix` at the current tip of `main`, applies the stash and keeps it
* [ ] Creates `fix` from the stash's changes but stays on `main`
* [ ] Commits the stash onto a new branch `fix` without touching your files
* [x] Creates `fix` at the commit the stash was made on, applies it, then drops it

> **Explanation:** Starting from the original commit means the stash applies without conflicts. Git switches to `fix`, applies the changes and prints `Dropped stash@{0}` when it succeeds.

#### Q. You stash some work on `main`, then switch to `other` and run `git stash list`. What do you see?

```text
$ git stash -q
$ git switch -q other
$ git stash list
```

* [ ] Nothing, because stashes are stored per branch
* [ ] An error, because the stash belongs to `main`
* [x] `stash@{0}: WIP on main: f59a74e init`
* [ ] `stash@{0}: WIP on other: f59a74e init`

> **Explanation:** There is one stash list for the whole repository; the entry only records which branch it came from. You can apply it on `other`, which is a common way to move uncommitted work between branches.

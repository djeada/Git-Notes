# History, Remotes and Recovery Quiz

Notes: [Observing the repository](../notes/08_observing_repository.md), [Synchronization](../notes/09_synchronization.md), [Squashing commits](../notes/10_squashing_commits.md), [Archive](../notes/11_archive.md), [Dangerous commands](../notes/16_dangerous_commands.md)

Tests inspecting history, working with remotes and remote-tracking branches, rewriting commits safely, packaging releases, and recovering lost work.

#### Q. You run the commands below. What change does the last command show for `f`?

```bash
echo v1 > f && git add f && git commit -qm one
echo v2 > f && git add f
echo v3 > f
git diff --cached
```

* [ ] `v2` replaced by `v3`
* [x] `v1` replaced by `v2`
* [ ] `v1` replaced by `v3`
* [ ] No output, because `f` also has unstaged changes

> **Explanation:** `git diff --cached` compares the index (`v2`) with `HEAD` (`v1`). Plain `git diff` would show `v2` to `v3`, and `git diff HEAD` would show `v1` to `v3`.

#### Q. You edited the tracked file `app.js` and created a new file `notes.txt` that you have not added. What does `git diff HEAD` show?

* [x] The changes to `app.js` only, because untracked files never appear in a diff
* [ ] Both files, because it compares the whole working directory with `HEAD`
* [ ] Only `notes.txt`, because tracked changes need `git diff --cached`
* [ ] Nothing, until `notes.txt` is either staged or ignored

> **Explanation:** `git diff` and its variants only compare tracked content. An untracked file shows up in `git status`, but it appears in a diff only after you stage it (or mark it with `git add -N`).

#### Q. `feature` branched off `main`, and both branches have new commits since then. What does `git log main..feature` list?

* [ ] Commits on `main` that are not on `feature`
* [ ] Commits on either branch but not on both
* [x] Commits reachable from `feature` that are not reachable from `main`
* [ ] Every commit between the two branch tips in date order

> **Explanation:** `A..B` means "reachable from B, excluding anything reachable from A", so it lists what `feature` would add to `main`. The symmetric "either but not both" set is `main...feature`.

#### Q. Since branching, `main` added file `h` and `feature` added file `g`. Which files appear in `git diff --stat main...feature`?

* [ ] `g` and `h`, with `h` shown as deleted
* [ ] Only `h`, shown as deleted
* [ ] `g` and `h`, both shown as added
* [x] Only `g`

> **Explanation:** With three dots, `git diff` compares the merge base with `feature`, so you see only what `feature` changed. Two dots (or a space) compares the two tips, which makes `h` look deleted.

#### Q. The line `timeout = 30` was added to a file in commit c2, an unrelated line was added in c3, and the timeout line was deleted in c4. Which commits does `git log --oneline -S 'timeout'` list?

* [ ] Only c2, the commit that added the string
* [x] c2 and c4
* [ ] c2, c3 and c4, since the file contained the string in all of them
* [ ] Only c4, the most recent commit that touched it

> **Explanation:** The pickaxe `-S` lists commits that change how many times the string occurs, so both adding and removing it count. c3 left the count unchanged, so it is not listed.

#### Q. What does `git show HEAD~2:config.yml` do?

* [x] Prints `config.yml` as it was two commits before `HEAD`, leaving the working tree alone
* [ ] Restores `config.yml` from two commits ago into the working tree
* [ ] Shows the diff that commit `HEAD~2` made to `config.yml`
* [ ] Checks out `HEAD~2` as a detached `HEAD` and opens the file

> **Explanation:** The `<commit>:<path>` syntax names the file's blob in that commit, and `git show` prints its contents. Nothing is checked out or restored; use `git restore --source` to actually bring the old version back.

#### Q. Which command shows, for each of lines 10 to 20 of `app.js`, the commit and author that last changed it?

* [ ] `git log -p -- app.js`
* [ ] `git show -L 10,20 app.js`
* [x] `git blame -L 10,20 app.js`
* [ ] `git diff HEAD -- app.js`

> **Explanation:** `git blame` annotates each line with the last commit, author and date that touched it, and `-L` limits it to a line range. `git log -p` shows whole commits, not per-line attribution.

#### Q. Alice wrote the logic in `f.py`; later Bob reindented every line. `git blame f.py` now shows Bob on every line. Why?

* [ ] `git blame` always shows the most recent committer of the file as a whole
* [x] `git blame` reports the last commit that changed each line, and the reindent changed every line
* [ ] `git blame` shows whoever last ran `git add` on the file
* [ ] `git blame` attributes lines to the person who merged them into `main`

> **Explanation:** Blame is per line and per last change, so a whitespace-only commit takes over every line it touched. `git blame -w` ignores whitespace changes and shows Alice again.

#### Q. You know tag `v1.0` was good and the current `HEAD` is bad. Which sequence correctly starts a bisect session?

* [ ] `git bisect start v1.0 HEAD`
* [ ] `git bisect start`, `git bisect good HEAD`, `git bisect bad v1.0`
* [x] `git bisect start`, `git bisect bad`, `git bisect good v1.0`
* [ ] `git bisect run v1.0..HEAD`

> **Explanation:** `git bisect bad` without an argument marks the current commit. `git bisect start` takes the bad commit first, so `start v1.0 HEAD` swaps them and Git complains that the good rev is not an ancestor of the bad one.

#### Q. During `git bisect run ./test.sh`, the script exits with code 125 on a commit that does not compile. What does bisect do with that commit?

* [ ] Marks it bad, like any non-zero exit code
* [ ] Aborts the whole bisect session
* [ ] Marks it good, because 125 means success with warnings
* [x] Skips it and tests a nearby commit instead

> **Explanation:** For `bisect run`, exit 0 marks a commit good, other small codes such as 1 mark it bad, and 125 is reserved for "cannot test this commit", which works like `git bisect skip`.

#### Q. You rebased `feature` onto `main` and want to see where `feature` pointed just before the rebase. Which reference names that commit?

* [ ] `HEAD@{1}`
* [x] `feature@{1}`
* [ ] `HEAD~1`
* [ ] `feature~1`

> **Explanation:** The branch reflog records the whole rebase as one update, so `feature@{1}` is the old tip (as is `ORIG_HEAD`). `HEAD@{1}` points at the last picked commit, because `HEAD` moved once per replayed commit.

#### Q. A teammate pushed a commit to `main` on `origin` a minute ago. You have not run any network command since. What does `git log origin/main` show you?

* [ ] The new commit, because `origin/main` always reflects the server
* [ ] An error, because `origin/main` can only be read after `git pull`
* [x] `main` as it was on `origin` when you last fetched or pushed, without the new commit
* [ ] Your local `main`, because `origin/main` is just another name for it

> **Explanation:** `origin/main` is a remote-tracking branch: a local record of the server's branch, updated only by `fetch`, `pull` or `push`. That is also why `git status` can say "up to date" while the server has moved on.

#### Q. You run `git checkout origin/main` and then make a commit. What happens?

* [x] `HEAD` is detached at that commit; the new commit moves only `HEAD`, not `origin/main`
* [ ] The commit is added to `origin/main` and goes to the server on the next `git push`
* [ ] Git creates a local `main` tracking `origin/main` and commits there
* [ ] Git refuses, because remote-tracking branches cannot be checked out

> **Explanation:** Checking out a remote-tracking branch detaches `HEAD`; only fetches and pushes move `origin/main`. `git switch origin/main` refuses outright, and `git switch main` is what creates a tracking local branch.

#### Q. After `git fetch origin`, which of these has changed?

* [ ] Your current branch, fast-forwarded if possible
* [ ] Your working tree, but not the index
* [ ] Your local branches and their remote-tracking branches, but not the working tree
* [x] Only remote-tracking branches such as `origin/main`, plus the downloaded objects

> **Explanation:** Fetch downloads objects and updates `refs/remotes/origin/*`. Your branches, index and files stay as they were until you merge, rebase or reset.

#### Q. Your `main` and `origin/main` have diverged, and `pull.rebase` is not configured. With Git 2.43 you run plain `git pull`. What happens?

* [ ] Git creates a merge commit, because merge is the default
* [x] Git fetches, then stops and asks you to choose merge, rebase or fast-forward only
* [ ] Git rebases your local commits on top of `origin/main`
* [ ] Git aborts before fetching and leaves `origin/main` unchanged

> **Explanation:** The fetch half runs and updates `origin/main`, then Git fails with `fatal: Need to specify how to reconcile divergent branches.` You pick with `--rebase`, `--no-rebase`, `--ff-only` or the `pull.rebase` setting.

#### Q. You have one unpushed commit and `origin/main` has one commit you do not have. After `git pull --rebase`, what does the first line of `git status -sb` show?

* [ ] `## main...origin/main [ahead 1, behind 1]`
* [ ] `## main...origin/main [ahead 2]`
* [x] `## main...origin/main [ahead 1]`
* [ ] `## main...origin/main`

> **Explanation:** The rebase replays your commit (with a new hash) on top of the fetched one, so you are no longer behind and are one commit ahead, ready to push as a fast-forward.

#### Q. `git push` prints the output below. What is the right next step?

```text
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'https://example.com/team/app.git'
hint: Updates were rejected because the remote contains work that you do not
hint: have locally.
```

* [ ] Run `git push --force` so the remote matches your branch
* [ ] Run `git push -u origin main` to fix the missing upstream
* [ ] Run `git fetch --prune`, then push again unchanged
* [x] Integrate the remote commits with `git pull` (merge or rebase), then push

> **Explanation:** The server's branch has commits that your push would throw away, so it is not a fast-forward. Integrate them first; forcing would delete a teammate's work.

#### Q. You amend a commit that you already pushed, then run `git push`. It is rejected as `(non-fast-forward)`. Why?

* [ ] The remote rejects any commit whose message has changed
* [x] The new commit does not build on the old remote tip, so it is not a fast-forward
* [ ] You must run `git push -u` again after amending to re-link the upstream
* [ ] The remote already has a commit with the same message and refuses the duplicate

> **Explanation:** Amending creates a new commit that replaces the old one instead of building on it. Publishing it requires a force push, which is why you should not amend commits others may have pulled.

#### Q. On a new local branch `topic`, `git push` fails with `fatal: The current branch topic has no upstream branch.` Which command pushes and sets up tracking in one step?

* [ ] `git branch -u origin/topic`, then `git push`
* [ ] `git push --force origin topic`
* [x] `git push -u origin topic`
* [ ] `git fetch origin topic`, then `git push`

> **Explanation:** `-u` (`--set-upstream`) creates `topic` on the remote and records `origin/topic` as its upstream. `git branch -u origin/topic` fails first, because `origin/topic` does not exist until something is pushed.

#### Q. `git branch -vv` shows `* topic c87ab1c [origin/topic: ahead 2] Add parser`. What does `ahead 2` mean?

* [x] `topic` has 2 commits that `origin/topic`, as last fetched, does not have
* [ ] `origin/topic` has 2 commits that you have not merged yet
* [ ] The server's `topic` is 2 commits ahead of what you last fetched
* [ ] 2 commits on `topic` failed to push and must be retried

> **Explanation:** Ahead/behind counts compare your branch with its local remote-tracking branch, so they show unpushed and unmerged commits as of your last fetch, not live server state.

#### Q. You rewrote `feature` locally. Meanwhile a teammate pushed a new commit to `origin/feature`, and you have not fetched. What does `git push --force-with-lease` do?

* [ ] It overwrites the teammate's commit, just like `--force`
* [ ] It merges the teammate's commit into yours, then pushes
* [ ] It pushes your branch and keeps the teammate's commit as a separate branch
* [x] It is rejected as `(stale info)`, and the teammate's commit stays

> **Explanation:** By default the lease expects the server's branch to still match your `origin/feature`, and here it does not. Plain `--force` has no such check and would silently overwrite the teammate's commit.

#### Q. Same situation, but your editor ran `git fetch` in the background before you pushed, and you did not look at the new commit. What does `git push --force-with-lease` do now?

* [ ] It is rejected, because the lease remembers the value from before the fetch
* [x] It succeeds and discards the teammate's commit, because `origin/feature` now matches the server
* [ ] It is rejected, because `--force-with-lease` refuses whenever the remote has commits you lack
* [ ] It succeeds, but the server keeps the teammate's commit on a backup branch

> **Explanation:** The lease only compares the server with your remote-tracking ref, and the fetch updated it. Check `git log HEAD..origin/feature` before pushing, or pass an explicit expected value with `--force-with-lease=feature:<sha>`.

#### Q. A force push to a bare server repository overwrote a commit you need. Where are you most likely to find that commit's hash?

* [ ] In the server's reflog, which records every push by default
* [ ] In `git log origin/feature`, which still shows the old tip
* [x] In `git reflog show origin/feature` in a clone that had fetched the old tip
* [ ] Nowhere, because a force push deletes the commit from every clone

> **Explanation:** Bare repositories keep no reflog by default, but each clone records every past value of its remote-tracking branches. A teammate's local branch may also still contain the commit.

#### Q. A teammate deleted `topic` on the server. You had a local `topic` tracking it. What does `git fetch --prune` do?

* [x] Deletes `origin/topic`; local `topic` stays and `git branch -vv` shows `[origin/topic: gone]`
* [ ] Deletes both `origin/topic` and your local `topic`
* [ ] Deletes local `topic` only if it is fully merged into `main`
* [ ] Changes nothing until you also run `git remote prune origin`

> **Explanation:** Pruning removes stale remote-tracking refs only; Git never deletes your local branches for you. Without `--prune` or `fetch.prune=true`, a plain `git fetch` would keep `origin/topic` as well.

#### Q. You want your local `main` to match `origin`'s `main` exactly, discarding your two local commits and uncommitted changes to tracked files. Which commands do that?

* [ ] `git fetch` then `git reset --soft origin/main`
* [x] `git fetch` then `git reset --hard origin/main`
* [ ] `git fetch` then `git checkout origin/main`
* [ ] `git pull --force`

> **Explanation:** `reset --hard` moves `main` to `origin/main` and overwrites the index and tracked files. `--soft` would keep the discarded changes staged, and checking out `origin/main` only detaches `HEAD`.

#### Q. You forked a project and cloned your fork as `origin`. How do you bring the original project's new commits into your local `main`?

* [ ] `git pull origin main`
* [ ] `git push upstream main`
* [ ] `git fetch --all` then `git merge origin/main`
* [x] `git remote add upstream <url>`, `git fetch upstream`, then `git merge upstream/main`

> **Explanation:** Your fork does not receive the original project's commits automatically, so pulling from `origin` gets nothing new. Add the original as a second remote, fetch it, and merge its `main`.

#### Q. You run `git rebase -i HEAD~4`. Which commit is on the first line of the todo list?

* [ ] `HEAD`, the newest commit
* [ ] `HEAD~4`, the base you passed
* [x] `HEAD~3`, the oldest of the four commits being rewritten
* [ ] The first commit of the repository

> **Explanation:** The todo list runs top to bottom in the order commits will be replayed, so the oldest comes first. `HEAD~4` itself is the base and is not rewritten.

#### Q. You change the todo list to `pick A`, `fixup B`, `fixup C` and save. What is the result?

* [x] One commit with all three changes and only A's message, with no message editor opened
* [ ] One commit with all three changes, after an editor opens pre-filled with all three messages
* [ ] Three commits, with B's and C's messages removed
* [ ] One commit with only A's changes; B and C are dropped

> **Explanation:** `fixup` melds a commit into the one above it and discards its message, so no editor is needed. `squash` would open an editor with the combined messages.

#### Q. In a `git rebase -i HEAD~3` todo list you change the first line from `pick` to `squash`. What happens?

* [ ] Git squashes that commit into its parent, `HEAD~3`
* [x] Git stops with `cannot 'squash' without a previous commit`
* [ ] Git silently treats the first `squash` as `pick`
* [ ] Git squashes that commit into the commit on the next line

> **Explanation:** `squash` and `fixup` combine a commit with the one before it in the list, and the first line has none. The base commit is outside the list; widen the range if you want to fold into it.

#### Q. In an interactive rebase todo list you delete the whole line for one commit, then save. What happens to that commit?

* [ ] It is kept, because missing lines default to `pick`
* [ ] Git aborts the rebase because the todo list is incomplete
* [ ] Its changes are squashed into the previous commit
* [x] It is dropped from the rewritten branch

> **Explanation:** Removing a line has the same effect as `drop`. The original commit is still reachable from the reflog if you change your mind.

#### Q. You run `git commit --fixup a1b2c3d` and later `git rebase -i --autosquash main`. What does the todo list look like?

* [ ] The `fixup!` commit stays last in the list, marked `pick`
* [ ] The `fixup!` commit is listed first, so it applies before its target
* [x] The `fixup!` commit is moved directly below `a1b2c3d` and marked `fixup`
* [ ] The `fixup!` commit is left out, and its changes are applied after the rebase

> **Explanation:** `--fixup` writes a `fixup! <subject>` message, and `--autosquash` uses it to reorder and mark the line. Set `rebase.autoSquash=true` to make this the default for `rebase -i`.

#### Q. `feature` has 4 commits since it branched from `main`. On `feature` you run `git reset --soft $(git merge-base main HEAD)`. What is the state now?

* [x] `feature` points at the fork point, and the changes from all 4 commits are staged
* [ ] `feature` points at `main`'s tip, and the changes are unstaged
* [ ] The 4 commits are deleted together with their changes
* [ ] `feature` is unchanged; only the index is reset to the fork point

> **Explanation:** `--soft` moves only the branch pointer and keeps the index and working tree, so a single `git commit -m "..."` now records the whole feature as one commit.

#### Q. Right after that `reset --soft`, you run `git commit --amend -m "feat: search"` instead of a plain commit. What happens?

* [ ] It amends the last of the 4 feature commits, as intended
* [ ] Git refuses, because there is nothing to amend after a reset
* [ ] It creates a new commit on top of the fork point
* [x] It rewrites `main`'s fork-point commit into a new commit on `feature`

> **Explanation:** After the reset, `HEAD` is the merge base, a commit that belongs to `main`. `--amend` replaces it with a new commit that also contains its changes. A plain `git commit` is what you want.

#### Q. You run `git merge --squash feat` on `main`, then `git commit`. Afterwards `git branch -d feat` fails with "not fully merged". Why?

* [ ] The squash merge left unresolved conflicts behind on `feat`
* [x] The squash commit has a single parent, so `feat`'s commits are not reachable from `main`
* [ ] `git branch -d` never deletes a branch that has been pushed
* [ ] `merge --squash` locks the source branch until you run `git merge --continue`

> **Explanation:** A squash merge copies the combined changes into one ordinary commit and records no merge, so Git cannot see that `feat` is merged. Use `git branch -D feat` once you are sure.

#### Q. You run `git merge --squash feat` on `main`. What state are you in before you commit?

* [ ] A new commit exists on `main` with a message listing every `feat` commit
* [ ] A merge commit with two parents exists, holding one combined diff
* [x] The combined changes from `feat` are staged, and no commit has been created yet
* [ ] Only the changes from `feat`'s last commit have been applied

> **Explanation:** `--squash` updates the index and working tree but stops before committing and does not set `MERGE_HEAD`. Your next `git commit` creates one regular commit.

#### Q. `README` has uncommitted edits and `untracked.txt` is untracked. You run `git archive -o out.zip HEAD`. What does `out.zip` contain?

* [x] The committed `README` and no `untracked.txt`
* [ ] Your edited `README` and `untracked.txt`
* [ ] Your edited `README` but no `untracked.txt`
* [ ] The committed `README` plus `untracked.txt`, since it is in the same folder

> **Explanation:** `git archive` packs the tree of the commit you name, not the working directory, so uncommitted edits and untracked files are never included.

#### Q. You add `secret.txt export-ignore` to `.gitattributes` but do not commit it. `git archive HEAD` still includes `secret.txt`. Why?

* [ ] `export-ignore` only works with `--format=zip`
* [x] `git archive` reads attributes from the tree being archived, and `HEAD` has no such rule
* [ ] Archive exclusions must be listed in `.gitignore` instead
* [ ] `secret.txt` must also be untracked for the rule to apply

> **Explanation:** Commit the `.gitattributes` change, or pass `--worktree-attributes` to use the working copy's attributes. `.gitignore` has no effect on files that are already tracked.

#### Q. What paths does `git archive --prefix=proj HEAD README | tar -t` list?

* [ ] `proj/README`
* [ ] `README`, because `--prefix` only applies with `-o`
* [x] `projREADME`
* [ ] `proj/` and `proj/README`

> **Explanation:** The prefix is prepended as a literal string, so you need the trailing slash. `--prefix=proj/` produces `proj/` and `proj/README`.

#### Q. Your repository contains a submodule at `libs/sub`. What does `git archive HEAD` put at that path?

* [x] An empty `libs/sub/` directory
* [ ] The submodule's files at the commit recorded in `HEAD`
* [ ] The submodule's files at its latest upstream commit
* [ ] A text file containing the submodule's commit hash

> **Explanation:** The superproject only stores a gitlink (a commit hash) for the submodule, and `git archive` does not recurse into it. Package submodules separately.

#### Q. You committed too early and want to undo the last commit but keep its changes staged, ready to recommit. Which command?

* [ ] `git reset --hard HEAD~1`
* [ ] `git reset HEAD~1`
* [ ] `git revert HEAD`
* [x] `git reset --soft HEAD~1`

> **Explanation:** `--soft` moves the branch back and leaves the index alone. The default `--mixed` would also unstage the changes, `--hard` would discard them, and `revert` adds a new undo commit.

#### Q. The last commit modified tracked file `f`. You run `git reset HEAD~1` with no mode flag. What does `git status --short` show?

* [ ] `M  f`
* [x] ` M f`
* [ ] `?? f`
* [ ] Nothing; the change is gone

> **Explanation:** The default `--mixed` mode resets the branch and the index but not the working tree, so the change is still in `f` but unstaged (`M` in the second column).

#### Q. You have an uncommitted edit to tracked file `f` and a new untracked file `untracked.txt`. You run `git reset --hard HEAD~1`. What does `git status --short` show?

* [ ] Nothing; `--hard` wipes tracked and untracked changes
* [ ] ` M f` and `?? untracked.txt`
* [x] `?? untracked.txt` only
* [ ] `M  f` and `?? untracked.txt`

> **Explanation:** `reset --hard` overwrites the index and tracked files but never touches untracked files. Removing those is the job of `git clean`.

#### Q. The working tree has untracked `untracked.txt`, untracked directory `build/`, and `debug.log`, which matches `.gitignore`. What does `git clean -fd` remove?

* [x] `build/` and `untracked.txt`; the ignored `debug.log` stays
* [ ] Only `untracked.txt`; directories also need `-x`
* [ ] All three, because `-d` includes ignored files
* [ ] Nothing, until you confirm each file interactively

> **Explanation:** `-f` is required to delete anything and `-d` adds untracked directories. Ignored files are removed only with `-x`. Run `git clean -nd` first to preview.

#### Q. You force-deleted an unmerged branch and Git printed `Deleted branch exp (was 5e14eed).` How do you get the branch back?

* [ ] `git reset --hard exp@{1}`
* [x] `git branch exp 5e14eed`
* [ ] `git revert 5e14eed`
* [ ] `git switch exp`, since deletion only hides the branch

> **Explanation:** Deleting a branch removes only the ref (and its own reflog); the commits remain until garbage collection. Recreate the ref at the printed hash, or find it in `git reflog` if it has scrolled away.

#### Q. You meant to undo one commit but ran `git reset --hard HEAD~3`. Nothing else has happened since. What restores the branch to where it was?

* [ ] `git revert HEAD~3..HEAD`
* [ ] `git restore --source=HEAD~3 .`
* [x] `git reset --hard HEAD@{1}`
* [ ] `git pull`, which re-downloads the missing commits

> **Explanation:** `HEAD@{1}` is where `HEAD` was before its last move, so resetting to it undoes the reset (`ORIG_HEAD` works too). The commits were never deleted, just unreferenced.

#### Q. You clone a repository fresh and immediately run `git reflog`. What does it show?

* [ ] The `HEAD` history of every developer who has pushed to the repository
* [ ] The server's reflog for the default branch
* [ ] Nothing, because reflogs must first be enabled in config
* [x] A single `clone: from ...` entry

> **Explanation:** Reflogs are local and are never pushed or cloned. Your reflog only records movements in your own repository, so it cannot recover a teammate's unpushed work.

#### Q. History is `A-B-C-D-E`, and `secret.txt` was added in C along with other changes. You run `git filter-repo --invert-paths --path secret.txt`. Which commits get new hashes?

* [ ] Every commit, A through E
* [x] Commits from C onward; A and B keep their hashes
* [ ] Only C; D and E keep theirs
* [ ] None; only the `secret.txt` blob is removed from storage

> **Explanation:** A commit's hash covers its tree and its parent, so changing C changes every descendant. Commits before C have identical content and keep their IDs. Rotate the secret anyway, since old clones still contain it.

#### Q. You rebased the shared branch `feature` and force-pushed. A teammate who had the old commits and nothing new runs `git fetch` then `git status`. What do they see?

* [x] Their `feature` has diverged from `origin/feature`, with the old commits counted as ahead
* [ ] `Your branch is up to date with 'origin/feature'`
* [ ] `Your branch is behind 'origin/feature'` and can be fast-forwarded
* [ ] An error saying `origin/feature` was rewritten and must be recloned

> **Explanation:** The old commits are not in the rewritten history, so Git reports something like `ahead 2, behind 3`. A plain pull would merge both copies and duplicate the commits, which is why published history should not be rewritten.

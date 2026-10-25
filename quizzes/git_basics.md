# Git Basics Quiz

Notes: [Introduction to version control](../notes/01_introduction_to_version_control.md), [Creating a repository](../notes/02_create_repository.md), [Making changes](../notes/03_making_changes.md), [Observing the repository](../notes/08_observing_repository.md), [Points of confusion](../notes/13_points_of_confusion.md)

Covers the three areas (working tree, index, repository), init, clone and config scopes, `.gitignore` rules, staging, committing, status, diff, restore, amend, basic undo with revert and reset, and removing a file from history.

#### Q. The central server hosting your team's repository goes offline for a day. What can you still do in your full local clone?

* [x] Browse the whole history and make new commits, then push them later
* [ ] View only the latest snapshot, because older history is fetched on demand
* [ ] Make new commits, but `git log` shows only commits created since the outage
* [ ] Read files only, because commits must be registered with the server first

> **Explanation:** Git is distributed: a full clone holds every commit locally, so history browsing and committing work offline. Only operations that talk to the remote, such as fetch and push, need the server.

#### Q. You run the commands below. Which content of `notes.txt` ends up in the new commit?

```bash
echo "draft 1" > notes.txt
git add notes.txt
echo "draft 2" > notes.txt
git commit -m "Add notes"
```

* [ ] `draft 2`, because `git commit` reads files from the working tree
* [x] `draft 1`, because `git commit` records what is in the index
* [ ] Nothing, because the file changed after it was staged
* [ ] Both versions, stored as two separate commits

> **Explanation:** A commit is built from the index. The second edit exists only in the working tree, so after the commit `git status --short` still shows ` M notes.txt`.

#### Q. What does a Git commit object actually point to?

* [ ] A diff of the lines that changed since its parent commit
* [ ] A copy of the whole working directory, including untracked files
* [x] A tree holding a full snapshot of the tracked files, plus parents and metadata
* [ ] Only the files that were staged for that particular commit

> **Explanation:** `git cat-file -p HEAD` shows a `tree` line (the complete snapshot), `parent` lines, author, committer and message. Diffs are computed on the fly when you compare two snapshots.

#### Q. `report.txt` has a change that is staged and a different change that is not. You run `git restore --staged report.txt`. What happens?

* [ ] Both changes are discarded and the file matches the last commit
* [ ] The unstaged change is discarded and the staged one is kept
* [ ] The staged change is committed and the unstaged one stays
* [x] The index entry is reset to `HEAD`; the working tree file is untouched

> **Explanation:** `--staged` restores only the index, from `HEAD` by default. The working tree keeps its latest content, so both edits now appear as one unstaged modification (` M`).

#### Q. In a project that has never been pushed anywhere, you run `rm -rf .git`. What is the result?

* [ ] Git rebuilds `.git` from the working tree on the next `git status`
* [x] The current files stay, but all commits, branches and staged state are gone
* [ ] Only the configuration is lost; history is stored with each file
* [ ] The files are deleted too, because Git owns the working tree

> **Explanation:** All history and metadata live inside `.git`. Removing it leaves plain files in a normal folder, and `git status` then fails with `fatal: not a git repository`.

#### Q. You run `git clone https://github.com/djeada/git.git` with no other arguments. Where does the working copy go?

* [ ] Into the current directory, mixed in with any existing files
* [ ] Into a new directory named `git.git`
* [ ] Into a new directory named `djeada/git`
* [x] Into a new directory named `git`

> **Explanation:** Without a destination, `git clone` uses the last path component of the URL and strips a trailing `.git`, so `git.git` becomes `git`.

#### Q. You run `git clone <url> project`, but `project/` already exists and contains a file. What happens?

* [x] Clone aborts with `fatal: destination path 'project' already exists and is not an empty directory.`
* [ ] Clone merges the remote files into the existing ones and keeps local copies on conflict
* [ ] Clone overwrites the existing files with the versions from the remote
* [ ] Clone creates `project/project` and places the repository inside it

> **Explanation:** `git clone` accepts a destination only if it does not exist or is an empty directory. Anything else is refused with exit code 128.

#### Q. The remote has branches `main` and `dev`. Right after `git clone`, what does `git branch` (no options) list?

* [ ] `dev` and `main`
* [x] Only `* main`
* [ ] `origin/dev` and `origin/main`
* [ ] Nothing until you run `git fetch`

> **Explanation:** Clone creates remote-tracking branches for every remote branch (`git branch -r` shows `origin/dev`), but only one local branch: the remote's default. `git switch dev` creates a local `dev` from `origin/dev`.

#### Q. You are setting up a shared repository on a server that people will only push to and pull from. Which clone option fits?

* [ ] `--depth 1`
* [ ] `--single-branch`
* [x] `--bare`
* [ ] `--recurse-submodules`

> **Explanation:** A bare repository has no working tree, so there are no checked-out files to get out of sync when others push. Commands like `git status` in it fail with `this operation must be run in a work tree`.

#### Q. Your `~/.gitconfig` sets `user.email` to `me@personal.dev` and the repository's `.git/config` sets it to `me@work.com`. Inside that repository, what does `git config user.email` print?

* [ ] `me@personal.dev`, because global settings override local ones
* [ ] Both addresses, one per line
* [ ] An error, because the key is defined in two scopes
* [x] `me@work.com`, because local settings override global ones

> **Explanation:** Git reads system, then global, then local config, and the last value read wins. Local therefore beats global, which beats system. `git config --show-scope user.email` confirms the value comes from `local`.

#### Q. Inside a repository you run `git config user.email "ci@example.com"` with no scope flag. Where is the value written?

* [x] `.git/config` of the current repository
* [ ] `~/.gitconfig`
* [ ] `/etc/gitconfig`
* [ ] Nowhere, because a scope flag is required when writing

> **Explanation:** Writing without a scope flag defaults to `--local`. Outside a repository the same command fails with `fatal: not in a git directory`.

#### Q. Commits in one repository keep getting an unexpected author email. Which command shows which config file the value comes from?

* [ ] `git config --get-all user.email`
* [ ] `git config --global --list`
* [x] `git config --show-origin user.email`
* [ ] `git log --format=%ae -1`

> **Explanation:** `--show-origin` prints the file path next to the value, for example `file:.git/config`. `--get-all` lists values without their source, and `--global --list` shows only one scope.

#### Q. A directory contains `app.log`, `important.log` and `notes.txt`. With the `.gitignore` below, which files does `git status --short` list as untracked (besides `.gitignore`)?

```text
*.log
!important.log
```

* [ ] Only `notes.txt`
* [ ] All three files
* [x] `important.log` and `notes.txt`
* [ ] Only `important.log`

> **Explanation:** `*.log` ignores every log file, then the later `!important.log` re-includes that one. Later matching patterns override earlier ones, so only `app.log` stays ignored.

#### Q. With the `.gitignore` below, does `build/keep.txt` show up in `git status`?

```text
build/
!build/keep.txt
```

* [ ] Yes, because the negation comes after the directory rule
* [x] No, because Git cannot re-include a file whose parent directory is excluded
* [ ] Yes, but only after running `git add build/`
* [ ] No, because negation patterns are not allowed to contain a slash

> **Explanation:** Once `build/` is excluded, Git never looks inside it, so the negation has no effect. Write `build/*` instead to ignore the contents but still allow `!build/keep.txt`.

#### Q. The repository has a file `TODO` at the root and another at `src/TODO`. Which `.gitignore` line ignores only the root one?

* [ ] `TODO`
* [ ] `TODO/`
* [ ] `**/TODO`
* [x] `/TODO`

> **Explanation:** A leading slash anchors the pattern to the directory containing the `.gitignore`. A bare `TODO` or `**/TODO` matches at any depth, and `TODO/` would match only directories.

#### Q. A root `.gitignore` contains `src/*.txt`. Which of these files is ignored?

* [x] `src/c.txt`
* [ ] `src/sub/c.txt`
* [ ] `c.txt`
* [ ] `docs/src/c.txt`

> **Explanation:** A pattern with a slash is relative to the `.gitignore` location, and `*` does not cross `/`. It matches only files directly in the top-level `src/`; `src/**/*.txt` would match nested ones too.

#### Q. `config.env` was committed weeks ago. You add `config.env` to `.gitignore` and then edit the file. What does `git status --short` show for it?

* [ ] Nothing, because the file is now ignored
* [ ] `!! config.env`
* [x] ` M config.env`
* [ ] `D  config.env`

> **Explanation:** `.gitignore` only affects untracked files. A file that is already in the index stays tracked, so its edits keep showing as modified until you remove it from the index.

#### Q. You want Git to stop tracking the already-committed `config.env` but keep the file on your disk. Which command do you run before committing?

* [ ] `git rm config.env`
* [ ] `git restore --staged config.env`
* [ ] `git reset --hard config.env`
* [x] `git rm --cached config.env`

> **Explanation:** `--cached` removes the file from the index only, so the next commit deletes it from the tree while the local copy remains. Plain `git rm` would also delete it from the working tree.

#### Q. `app.log` is matched by `*.log` in `.gitignore`. What happens when you run `git add app.log`?

* [x] Git refuses with exit code 1 and suggests `-f` to add it anyway
* [ ] Git stages it silently, because naming a file overrides `.gitignore`
* [ ] Git stages it and prints a warning, exiting with code 0
* [ ] Git removes the matching line from `.gitignore` and stages the file

> **Explanation:** Git prints `The following paths are ignored by one of your .gitignore files` and a hint to use `-f`. `git add -f app.log` stages it, after which the file is tracked like any other.

#### Q. A `.gitignore` contains the single line below, and the directory has `c.txt`. Is `c.txt` ignored?

```text
*.txt # text files
```

* [ ] Yes, because everything after `#` is treated as a comment
* [x] No, because `#` starts a comment only at the beginning of a line
* [ ] Yes, but only in the root directory
* [ ] No, because patterns cannot contain spaces

> **Explanation:** The whole line, including ` # text files`, is one pattern, so it does not match `c.txt`. Put comments on their own lines.

#### Q. You want to ignore your editor's `*.swp` files in one repository without changing any file your teammates receive. Where do you add the pattern?

* [ ] The repository's root `.gitignore`
* [ ] A new `.gitignore` inside `.git/`
* [x] `.git/info/exclude`
* [ ] `.git/config` under a `[core]` section

> **Explanation:** `.git/info/exclude` uses `.gitignore` syntax but lives inside `.git`, so it is never committed or shared. A committed `.gitignore` would affect everyone.

#### Q. A file you expected to see in `git status` never appears. Which command tells you exactly which ignore file and line is hiding it?

* [ ] `git status --ignored`
* [x] `git check-ignore -v path/to/file`
* [ ] `git ls-files --others`
* [ ] `git config --get core.excludesFile`

> **Explanation:** `git check-ignore -v` prints the source file, line number and pattern, such as `.gitignore:3:build/`. `git status --ignored` shows ignored files (`!!`) but not which rule matched.

#### Q. In a repository where `a.txt` is committed, you run the commands below. What does the last command print?

```bash
echo b >> a.txt
git add a.txt
echo c >> a.txt
git status --short
```

* [ ] `M  a.txt`
* [ ] ` M a.txt`
* [ ] `AM a.txt`
* [x] `MM a.txt`

> **Explanation:** The first column is index versus `HEAD`, the second is working tree versus index. The staged edit gives `M` in the first column and the later unstaged edit gives `M` in the second.

#### Q. `a.txt` is tracked and unchanged. What does `git status --short` print after `git rm --cached a.txt`?

```bash
git rm --cached a.txt
git status --short
```

* [ ] ` D a.txt`
* [x] `D  a.txt` followed by `?? a.txt`
* [ ] `D  a.txt` only
* [ ] Nothing, because the file is still on disk

> **Explanation:** The index no longer has the file, so a deletion is staged (`D` in the first column). The file is still on disk and not ignored, so it also shows as untracked.

#### Q. You rename a tracked file with the shell instead of Git. What does `git status --short` print?

```bash
mv old.txt new.txt
git status --short
```

* [x] ` D old.txt` and `?? new.txt`
* [ ] `R  old.txt -> new.txt`
* [ ] ` R old.txt -> new.txt`
* [ ] `D  old.txt` and `A  new.txt`

> **Explanation:** Nothing is staged yet, so Git sees an unstaged deletion and an untracked file. After `git add -A` the status becomes `R  old.txt -> new.txt`, which is what `git mv` produces in one step.

#### Q. You edited the tracked `app.py` and created a new file `util.py`. What does `git commit -am "Update"` commit?

* [ ] Both files, because `-a` means all files in the working tree
* [ ] Nothing, because `-a` requires a prior `git add`
* [x] Only the change to `app.py`; `util.py` stays untracked
* [ ] Only `util.py`, because new files take priority

> **Explanation:** `-a` stages modifications and deletions of files Git already tracks. Untracked files must be added explicitly, so `util.py` still shows as `??` afterwards.

#### Q. You modified `src/a`, deleted `top`, and created `src/new`. Which describes the effect of `git add -u` at the repository root?

* [ ] It stages all three changes
* [ ] It stages only the new file `src/new`
* [ ] It stages the modification only; deletions need `git rm`
* [x] It stages the modification and the deletion, but not `src/new`

> **Explanation:** `-u` (update) works only on tracked files, so it records modifications and deletions. New files need `git add <path>`, `git add .`, or `git add -A`.

#### Q. You changed files in both `src/` and `docs/`, then ran `git add .` from inside `src/`. What is staged?

* [x] Only the changes under `src/`
* [ ] All changes in the repository
* [ ] Only files directly in `src/`, not its subdirectories
* [ ] Nothing, because `git add .` must be run at the root

> **Explanation:** `.` is a pathspec for the current directory and everything below it. Changes in `docs/` remain unstaged; `git add -A` without a path would stage the whole tree.

#### Q. You create a new directory `assets/` containing two new files. What does `git status --short` show by default?

* [ ] `?? assets/a.png` and `?? assets/b.png`
* [ ] `A  assets/`
* [x] `?? assets/`
* [ ] Nothing, until the files are added

> **Explanation:** By default status collapses a wholly untracked directory into one `??` line. `git status --short -uall` lists each file individually.

#### Q. You create an empty directory `logs/` in a clean repository. What does `git status --short` print?

* [ ] `?? logs/`
* [x] Nothing
* [ ] `A  logs/`
* [ ] `!! logs/`

> **Explanation:** Git tracks file content, not directories, so an empty directory is invisible to it. Projects commonly add a placeholder file, such as `.gitkeep`, to keep the directory in the repository.

#### Q. `f.txt` was committed containing `one`. You run the commands below. What does `git diff --staged` show as added?

```bash
echo two >> f.txt
git add f.txt
echo three >> f.txt
```

* [ ] `+three` only
* [ ] `+two` and `+three`
* [ ] Nothing, because the file has unstaged changes
* [x] `+two` only

> **Explanation:** `git diff --staged` (also `--cached`) compares the index with `HEAD`, which is exactly what the next commit will contain. Plain `git diff` compares the working tree with the index and would show only `+three`.

#### Q. In the same situation (`two` staged, `three` unstaged), which command shows both added lines in one diff?

* [ ] `git diff`
* [x] `git diff HEAD`
* [ ] `git diff --staged`
* [ ] `git status -v`

> **Explanation:** `git diff HEAD` compares the working tree with the last commit, so it includes staged and unstaged changes. `git status -v` shows only the staged diff unless given `-v` twice.

#### Q. You create a new file `new.txt` and edit the tracked `f.txt`, staging neither. What does `git diff` show?

* [x] The change to `f.txt` only
* [ ] The changes to both files
* [ ] The full content of `new.txt` only
* [ ] Nothing, because nothing is staged

> **Explanation:** `git diff` compares the working tree with the index, and an untracked file has no index entry to compare against. Once you `git add new.txt`, it appears in `git diff --staged`.

#### Q. `f.txt` is committed as `v1`. What does `cat f.txt` print at the end?

```bash
echo v2 > f.txt
git add f.txt
echo v3 > f.txt
git restore f.txt
cat f.txt
```

* [ ] `v1`
* [ ] `v3`
* [x] `v2`
* [ ] Nothing, because the file is deleted

> **Explanation:** Without `--source`, `git restore` fills the working tree from the index, not from `HEAD`. The staged `v2` comes back and `v3` is lost; `git status --short` then shows `M  f.txt`.

#### Q. `f.txt` is tracked and has unstaged edits. What happens when you run `git rm f.txt`?

* [x] Git refuses, reports local modifications, and suggests `--cached` or `-f`
* [ ] Git deletes the file and stages the deletion, discarding the edits
* [ ] Git stages the deletion but leaves the edited file on disk
* [ ] Git commits the edits first and then removes the file

> **Explanation:** `git rm` protects uncommitted work: it errors with `the following file has local modifications` and exits with 1. `--cached` keeps the file on disk and `-f` forces removal.

#### Q. You renamed `top` to `top2` with `git mv` and committed. Which command lists the commits from before the rename as well?

* [ ] `git log -- top2`
* [ ] `git log --all -- top2`
* [ ] `git log --stat top2`
* [x] `git log --follow -- top2`

> **Explanation:** Git stores no rename record; it detects renames by comparing snapshots. `--follow` makes `git log` track a single file across renames, while a plain path filter stops at the rename commit.

#### Q. `f.txt` is clean. You run `git restore --source=HEAD~1 f.txt`. What does `git status --short` show next?

* [ ] `M  f.txt`, because the old version is staged
* [x] ` M f.txt`, because only the working tree changed
* [ ] Nothing, because `HEAD` moved back one commit
* [ ] `MM f.txt`, because both index and working tree changed

> **Explanation:** By default `git restore` writes only to the working tree, even with `--source`. `HEAD` and the index are untouched; add `--staged` to update the index as well.

#### Q. You commit, then notice you forgot to stage `b.txt`. You run `git add b.txt` and `git commit --amend --no-edit`. What is the result?

* [x] The last commit is replaced by a new commit, with a new hash, containing `b.txt`
* [ ] A second commit containing only `b.txt` is added on top
* [ ] The existing commit is modified in place and keeps its hash
* [ ] The commit message is cleared because `--no-edit` skips the editor

> **Explanation:** Commits are immutable, so amend creates a new commit with the same parent and replaces the branch tip. The commit count does not change, but the hash does, which matters if the old commit was pushed.

#### Q. You already pushed a commit to `origin/main`, then amended it locally. What does a plain `git push` do?

* [ ] It replaces the remote commit, because the branch names match
* [ ] It pushes the amended commit as a new commit after the old one
* [ ] It succeeds and the remote keeps both versions on separate branches
* [x] It is rejected as non-fast-forward, because the histories have diverged

> **Explanation:** The amended commit is not a descendant of the pushed one, and `git status` reports the branches have diverged. Replacing the remote commit requires a force push, ideally `--force-with-lease`.

#### Q. After amending a pushed commit, you push with `--force-with-lease`, but a teammate pushed a new commit since your last fetch. What happens?

* [ ] The push succeeds and the teammate's commit is overwritten
* [x] The push is rejected as `stale info`, protecting the teammate's commit
* [ ] Git automatically rebases your commit on top of the teammate's
* [ ] The push succeeds and Git creates a merge commit on the remote

> **Explanation:** `--force-with-lease` overwrites the remote branch only if it still points where your `origin/main` says it does. A plain `--force` would discard the teammate's commit.

#### Q. A bad commit is already on the shared `main` branch that teammates have pulled. What is the safest way to undo it?

* [ ] `git reset --hard HEAD~1` followed by `git push --force`
* [ ] `git commit --amend` to empty the commit, then push
* [ ] `git restore --source=HEAD~1 .` and push without committing
* [x] `git revert <commit>` and push the new commit

> **Explanation:** `git revert` adds a new commit, such as `Revert "Add c"`, that undoes the changes while keeping history intact. Resetting and force-pushing rewrites history that others already have.

#### Q. Your last commit modified `a.txt` and added `b.txt`. What does `git status --short` show after `git reset --soft HEAD~1`?

* [x] `M  a.txt` and `A  b.txt`
* [ ] ` M a.txt` and `?? b.txt`
* [ ] Nothing, because the changes are discarded
* [ ] `MM a.txt` and `AM b.txt`

> **Explanation:** `--soft` moves only the branch pointer; the index and working tree still hold the commit's content, so the changes appear staged and ready to recommit.

#### Q. With the same last commit (modified `a.txt`, added `b.txt`), what does `git status --short` show after `git reset HEAD~1`?

* [ ] `M  a.txt` and `A  b.txt`
* [ ] Nothing, because the changes are discarded
* [x] ` M a.txt` and `?? b.txt`
* [ ] ` M a.txt` and ` A b.txt`

> **Explanation:** With no mode flag, reset uses `--mixed`: it moves the branch and resets the index, but keeps the working tree. The changes become unstaged, and `b.txt` becomes untracked again.

#### Q. You have an edited tracked file and a new untracked file `scratch.tmp`. What remains after `git reset --hard`?

* [ ] Both changes are gone and the directory matches `HEAD` exactly
* [ ] Both changes remain, because no commit was given
* [ ] The edit remains, but `scratch.tmp` is deleted
* [x] The edit is gone, but `scratch.tmp` is still there

> **Explanation:** `reset --hard` resets the index and tracked files to `HEAD`, but never touches untracked files. Use `git clean` to remove those.

#### Q. You committed `secret.txt`, then ran `git rm secret.txt` and committed again. Can someone with a clone still read the secret?

* [ ] No, because `git rm` removes the file from every commit
* [ ] Only if they cloned before the second commit
* [x] Yes, for example with `git show HEAD~1:secret.txt`
* [ ] No, because Git garbage-collects deleted files on the next push

> **Explanation:** Deleting a file only affects the new snapshot; earlier commits still contain it. Removing it requires rewriting history with `git filter-repo` or `git filter-branch`, then rotating the leaked secret anyway.

#### Q. You strip `secret.txt` from `main` with `git filter-branch --index-filter 'git rm -r --cached --ignore-unmatch secret.txt' HEAD`. Why does `git log --all --oneline -- secret.txt` still list commits?

* [x] `filter-branch` saved the old history under `refs/original/`
* [ ] `--index-filter` only rewrites the most recent commit
* [ ] `git log --all` reads from the remote rather than local refs
* [ ] The rewrite only takes effect after the next `git commit`

> **Explanation:** `filter-branch` keeps a backup ref such as `refs/original/refs/heads/main`. Deleting it, expiring the reflog and running `git gc --prune=now` are needed before the old objects are really gone.

#### Q. Which command reliably reports whether any commit reachable from any ref still contains a file at the path `secret.txt`?

* [ ] `git grep "secret.txt" $(git rev-list --all)`
* [ ] `git log --stat`
* [ ] `git ls-files secret.txt`
* [x] `git log --all --oneline -- secret.txt`

> **Explanation:** A path filter with `--all` lists every commit, across all refs, that touched that path. `git grep` searches file contents for the text, not file names, and `git log --stat` only walks the current branch.

#### Q. A merge stops on a conflict and the file contains the block below. Which text came from the branch you were on when you ran `git merge feature`?

```text
<<<<<<< HEAD
mainv
=======
feat
>>>>>>> feature
```

* [ ] `feat`, because the incoming branch is shown first
* [x] `mainv`, the part between `<<<<<<< HEAD` and `=======`
* [ ] Neither; the block shows the common ancestor's versions
* [ ] Both lines came from `feature`; `HEAD` marks the target position

> **Explanation:** The section after `<<<<<<< HEAD` is your current branch's version, and the section before `>>>>>>> feature` is the incoming one. Edit the file, remove the markers, then `git add` it and commit.

#### Q. During that conflicted merge, what does `git status --short` show for the conflicted file `f`?

* [x] `UU f`
* [ ] `MM f`
* [ ] `C  f`
* [ ] `!! f`

> **Explanation:** `U` marks an unmerged path, and `UU` means both sides modified it. After resolving, `git add f` clears the conflict state so the merge can be committed.

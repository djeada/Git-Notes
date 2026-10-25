# Workflows, Servers and Internals Quiz

Notes: [Git server](../notes/12_git_server.md), [Branching strategies](../notes/14_branching_strategies.md), [Mono and multirepo](../notes/15_mono_and_multi_repo.md), [Git internals](../notes/17_git_internals.md)

Covers hosting bare repositories and server hooks, team branching strategies, monorepo and multirepo layout tools, and how Git stores objects and refs under the hood.

#### Q. The shared repository on the server was created with plain `git init` and has `main` checked out. What happens when a developer runs the push below?

```bash
git push origin main
```

* [ ] The push succeeds and the server's working tree is updated to the new commit
* [x] The push is refused with `[remote rejected] main -> main (branch is currently checked out)`
* [ ] The push succeeds, but the server's checked-out files silently go stale
* [ ] The push is refused as `non-fast-forward` until someone runs `git pull` on the server

> **Explanation:** By default `receive.denyCurrentBranch` refuses updates to the branch checked out in a non-bare repository, because its index and working tree would no longer match. Shared remotes should be bare (`git init --bare`).

#### Q. The server repo was created with `sudo git init --bare /opt/git/myrepo.git`, and root has no `init.defaultBranch` set. A developer whose config sets `init.defaultBranch=main` runs the commands below (Git 2.43 on both sides). What happens at the push?

```bash
git clone git@yourserver:/opt/git/myrepo.git && cd myrepo
echo hello > README.md && git add README.md && git commit -m "Initial commit"
git push origin main
```

* [ ] The push creates `main` on the server and the server's `HEAD` switches to it
* [ ] The push creates `main`, and later clones check it out without any warning
* [ ] The push is rejected because an empty bare repository refuses its first push
* [x] It fails with `error: src refspec main does not match any`, because the clone's branch is `master`

> **Explanation:** Cloning an empty repository adopts the server's unborn `HEAD` (`refs/heads/master` here), so the commit lands on local `master`. Creating the server repo with `git init --bare --initial-branch=main` avoids the mismatch.

#### Q. Which remote URL makes Git connect as `git@yourserver` and open the same repository as `ssh://git@yourserver/opt/git/myrepo.git`?

* [x] `git@yourserver:/opt/git/myrepo.git`
* [ ] `git@yourserver:opt/git/myrepo.git`
* [ ] `git@yourserver/opt/git/myrepo.git`
* [ ] `ssh://yourserver:git/opt/git/myrepo.git`

> **Explanation:** In scp-like syntax the path after the colon is passed as-is, so `:opt/...` is relative to the git user's home directory. Without any colon, Git treats the string as a local path and never uses SSH.

#### Q. The `git` user's login shell is `/usr/bin/git-shell`. A developer with an authorised key runs the command below. What is the result?

```bash
ssh git@yourserver 'ls -la'
```

* [ ] A listing of the git user's home directory, after which the connection closes
* [ ] `Permission denied (publickey)`, because git-shell rejects every SSH login
* [x] `fatal: unrecognized command 'ls -la'`, while `git clone` and `git push` still work
* [ ] An interactive restricted shell that allows only `cd` and `ls`

> **Explanation:** git-shell accepts only Git's server-side commands such as `git-upload-pack` and `git-receive-pack`, so ordinary commands are refused while fetching and pushing keep working.

#### Q. The server's `pre-receive` hook exits with status 1. A developer pushes two branches at once. What ends up on the server?

```bash
git push origin main feat
```

* [ ] `feat` is created and only `main` is rejected, because it is listed first
* [ ] Both refs are updated, and the hook's exit status is reported as a warning
* [ ] `main` is updated and `feat` is rejected, because it is a new branch
* [x] Neither ref is updated; both are shown as `(pre-receive hook declined)`

> **Explanation:** `pre-receive` runs once for the whole push, reading every `<old> <new> <ref>` line on stdin; a non-zero exit rejects all of them. Use the `update` hook for per-ref decisions.

#### Q. The server has an `update` hook that exits 1 when its first argument is `refs/heads/main` and 0 otherwise. What happens when a developer runs `git push origin main feat`, where `feat` is a new branch?

* [ ] Both refs are rejected, because pushes are all-or-nothing by default
* [ ] Both refs are updated, because `update` hooks can only print warnings
* [x] `feat` is created and `main` is rejected with `(hook declined)`
* [ ] `main` is rejected and `feat` is skipped without being sent

> **Explanation:** `update` runs once per ref, with the ref name and old and new IDs as arguments, so it can reject individual refs. A push is only all-or-nothing when you use `git push --atomic`.

#### Q. A `post-receive` hook runs a deploy script that fails, and the hook exits with status 1. What does the developer who pushed see?

* [ ] `! [remote rejected] main -> main (post-receive hook declined)`, and the ref stays put
* [x] The hook's output prefixed with `remote:`, and a successful `main -> main` update
* [ ] A rejected push that Git retries automatically once the hook succeeds
* [ ] The ref update is rolled back on the server and `git push` exits non-zero

> **Explanation:** `post-receive` runs after all refs have been updated, so it cannot reject anything; its output is relayed with a `remote:` prefix and `git push` still exits 0. Validation belongs in `pre-receive` or `update`.

#### Q. After an admin runs `find /opt/git -type f -exec chmod 644 {} \;` on the server, pushes to `main` stop triggering the `post-receive` deploy. Why?

* [ ] Hooks must be owned by root, and the command changed their owner
* [ ] The hook now fails with `Permission denied`, so every push is rejected
* [x] Hooks without the execute bit are skipped, so the push succeeds but nothing runs
* [ ] Hooks only run from `.git/hooks`, which a bare repository does not have

> **Explanation:** Git runs a hook only if the file is executable; otherwise it ignores the hook and the push proceeds normally. Restore the bit with `chmod +x /opt/git/myrepo.git/hooks/post-receive`.

#### Q. You serve repositories with `git daemon --base-path=/opt/git /opt/git` and add a `git-daemon-export-ok` file to `myrepo.git`, so clones over `git://` work. What happens when someone pushes to that `git://` URL?

* [ ] The push succeeds without authentication, because `git://` has no access control
* [ ] The push succeeds only if the user's SSH key is in `authorized_keys`
* [x] It fails with `access denied or repository not exported`, because only fetching is enabled
* [ ] It fails until the repository also contains a `git-daemon-push-ok` file

> **Explanation:** `git daemon` enables only the `upload-pack` (fetch) service by default; `receive-pack` must be enabled explicitly, which is rarely wise on an unauthenticated protocol. Push over SSH or authenticated HTTPS instead.

#### Q. Nginx forwards `/git/` to `git-http-backend` with `GIT_HTTP_EXPORT_ALL` set and no authentication, and clones work. What happens when a developer pushes to that HTTP URL?

* [x] The server answers `403 Forbidden`, because pushes need an authenticated user by default
* [ ] The push succeeds, because `GIT_HTTP_EXPORT_ALL` enables both fetch and push
* [ ] Git retries the push over SSH using the same host name
* [ ] The push succeeds, but server-side hooks are skipped for HTTP pushes

> **Explanation:** `git-http-backend` enables `receive-pack` only when the web server sets `REMOTE_USER` (an authenticated request) or `http.receivepack` is turned on, so anonymous pushes are refused. Put authentication in front of the location to allow pushes.

#### Q. The `mirror` remote has branches `main` and `stale`. Locally you have branches `main` and `b1` and a tag `v1`. What does `git push --mirror mirror` do?

* [ ] Pushes `main`, `b1` and `v1`, and leaves `stale` untouched
* [x] Pushes `main`, `b1` and `v1`, and deletes `stale` on the remote
* [ ] Pushes only branches, because tags need a separate `git push --tags`
* [ ] Refuses to run, because `stale` would be lost without `--force`

> **Explanation:** `--mirror` makes the remote's refs match the local ones exactly: new and changed refs are pushed (force-updated if needed) and refs that no longer exist locally are deleted, shown as `- [deleted] stale`.

#### Q. Your team uses Git Flow. Production runs `v2.3.0` from `main`, and `develop` already contains unreleased features. A critical bug is reported in production. What is the Git Flow way to fix it?

* [ ] Commit the fix on `develop` and cut `release/2.3.1` from it
* [ ] Commit the fix directly on `main`, tag it, and leave `develop` alone
* [ ] Branch `feature/fix-bug` from `develop` and ship it with the next release
* [x] Branch `hotfix/2.3.1` from `main`, then merge it into `main` (tagged) and `develop`

> **Explanation:** Hotfix branches start from the production line so unreleased work on `develop` is not shipped by accident; merging back into `develop` (or an active release branch) keeps the bug from returning in the next release.

#### Q. In Git Flow, `release/1.4` has been cut from `develop` and QA is testing it. A developer finishes a new feature that was not planned for 1.4. Where should it be merged?

* [x] Into `develop`, so it ships with the next release
* [ ] Into `release/1.4`, as long as its tests pass
* [ ] Directly into `main`, so it bypasses the frozen release
* [ ] Into a `hotfix/*` branch that is merged into the release

> **Explanation:** Once a release branch is cut, only stabilisation work (bug fixes, docs, version bumps) goes on it. New features keep landing on `develop` and wait for the next release branch.

#### Q. Your team practises trunk-based development. A feature will take two weeks, but work must merge to `main` at least daily and `main` must stay releasable. What is the standard approach?

* [ ] Keep a long-lived `feature/*` branch and merge it once at the end
* [x] Merge small increments daily behind a feature flag that is off by default
* [ ] Push daily to a shared `develop` branch and merge that into `main` monthly
* [ ] Merge to `main` daily but skip CI on commits that are unfinished

> **Explanation:** Trunk-based development keeps branches short and hides incomplete behaviour behind flags (or branch by abstraction), so every commit on trunk builds, passes tests and is safe to deploy.

#### Q. Which description matches GitHub flow rather than Git Flow?

* [ ] Permanent `main` and `develop` branches, plus `release/*` and `hotfix/*` branches
* [ ] One long-lived branch per environment, promoted by merging upward
* [ ] Everyone commits straight to `main`, with no branches and no reviews
* [x] A single always-deployable `main`, with short-lived branches merged via pull request

> **Explanation:** GitHub flow uses only `main` plus short-lived topic branches that are reviewed in a pull request, merged and deployed. It has no `develop`, `release/*` or `hotfix/*` branches.

#### Q. You maintain `release/1.x` alongside `main`. A bug fix was committed on `main` as `412cc26`, and you run the commands below. What does the new commit's message look like?

```bash
git switch release/1.x
git cherry-pick -x 412cc26
```

* [ ] The original message, and the new commit keeps the hash `412cc26`
* [x] The original message followed by `(cherry picked from commit 412cc26...)`
* [ ] `Merge commit '412cc26' into release/1.x`
* [ ] The original message followed by a `Signed-off-by:` line naming you

> **Explanation:** `-x` appends a line recording the source commit, which makes a fix traceable across release branches. The new commit has a different parent, so its hash differs from `412cc26`.

#### Q. A team keeps `qa`, `staging` and `production` branches and promotes code by merging, cherry-picking hotfixes between them. What is the main criticism of this setup?

* [ ] Git cannot merge between more than two long-lived branches
* [ ] It forces every change to be hidden behind a feature flag
* [x] Environments end up running different commits, breaking "build once, promote the same artifact"
* [ ] Production must be marked with lightweight tags, which cannot be signed

> **Explanation:** Environment branches drift apart through merges and cherry-picks, so the commit tested in staging is not the one that reaches production. Promote a single build and keep environment differences in configuration.

#### Q. In your clone of a fork, `origin` is your fork and `upstream` is the canonical project. Your `feat` branch is behind upstream's `main`. Which commands bring it up to date?

* [ ] `git pull origin main`, then `git push upstream feat`
* [x] `git fetch upstream`, then `git rebase upstream/main`
* [ ] `git fetch origin`, then `git rebase origin/main`
* [ ] `git remote set-url origin <upstream-url>`, then `git pull`

> **Explanation:** New upstream commits only reach your clone by fetching from `upstream`; your fork's `origin/main` stays stale until you sync it. Rebasing onto `upstream/main` replays your work on the latest canonical history.

#### Q. A teammate pushed to `develop` a minute ago. You run `git fetch origin develop` before rebasing your feature branch. Where is the new tip of `develop` recorded?

* [ ] Only in `FETCH_HEAD`; `origin/develop` is updated only by a plain `git fetch`
* [ ] In your local `develop` branch, which fetch fast-forwards automatically
* [x] In both `FETCH_HEAD` and `origin/develop`, so `git rebase origin/develop` sees it
* [ ] Nowhere yet, because fetch with a branch name only checks for updates

> **Explanation:** Fetching a named branch writes `FETCH_HEAD` and also updates the matching remote-tracking ref, shown as `develop -> origin/develop`. Fetch never moves your local branches.

#### Q. A function in a shared library is renamed, and five services call it. Which property of a monorepo helps most with this change?

* [ ] Each service can keep the old name until it opts into a new library version
* [x] The rename and every call-site update can land in one atomic commit
* [ ] Clones are smaller, because each service is downloaded separately
* [ ] Each service can be released on a fully independent schedule

> **Explanation:** With everything in one repository, a cross-cutting refactor is one reviewed, revertible commit. In a multirepo the library is published first and each consumer is updated separately; the other options describe multirepo strengths.

#### Q. You add a library as a submodule at `libs/lib` and commit. What does the command below print for that path?

```bash
git ls-tree HEAD libs/
```

* [x] `160000 commit <hash>  libs/lib`, a pointer to one commit of the library
* [ ] `040000 tree <hash>  libs/lib`, listing all of the library's files
* [ ] `100644 blob <hash>  libs/lib`, a file containing the library's URL
* [ ] `120000 blob <hash>  libs/lib`, a symlink to `.git/modules/lib`

> **Explanation:** A submodule is stored as a gitlink: a tree entry with mode `160000` naming one commit in the other repository. The URL lives separately in `.gitmodules`.

#### Q. A teammate clones that superproject with a plain `git clone`. What do they find at `libs/lib`, and what fixes it?

* [ ] The library's latest default-branch files, so nothing needs fixing
* [ ] A broken symlink, which `git submodule sync` repairs
* [x] An empty directory; `git submodule update --init` checks out the recorded commit
* [ ] A file holding the URL; `git submodule add` must be run again

> **Explanation:** A plain clone does not fetch submodules, and `git submodule status` shows them with a leading `-`. Run `git submodule update --init` (or clone with `--recurse-submodules`) to clone them at the commit the superproject records.

#### Q. After `git submodule update --init`, a developer runs `git status` inside `libs/lib`. What is the first line of output?

* [ ] `On branch main`
* [ ] `On branch libs/lib`
* [ ] `fatal: not a git repository (or any of the parent directories): .git`
* [x] `HEAD detached at 53ed6ef`

> **Explanation:** `git submodule update` checks out the exact commit recorded in the superproject's gitlink, not a branch, so the submodule is in detached HEAD. Switch to a branch before committing inside it.

#### Q. A library was added with `git subtree add --prefix=libs/lib <url> main --squash`. What does a teammate get from a plain `git clone` of the app repository?

* [ ] An empty `libs/lib` until they run `git subtree pull`
* [ ] A gitlink at `libs/lib` that needs `git submodule update --init`
* [x] The library's files in `libs/lib` as ordinary tracked files
* [ ] A `.gitsubtree` file that the clone uses to fetch the library

> **Explanation:** A subtree merges the library's content into the repository's own history (here as one squashed commit), so clones get ordinary files with no extra steps. Only maintainers pulling upstream updates need `git subtree`.

#### Q. A monorepo has `README.md`, `apps/`, `libs/` and `services/` at the top level. You run the command below. Which files remain in the working tree?

```bash
git sparse-checkout set services/payments
```

* [ ] Only the files under `services/payments/`
* [x] `README.md` and other top-level files, plus everything under `services/payments/`
* [ ] Everything under `services/`, but nothing from the top level
* [ ] Every file, but only those under `services/payments/` are writable

> **Explanation:** `set` uses cone mode by default, which always keeps files directly in the root (and in each parent directory) plus the chosen directories recursively. The other files stay tracked in the index but are not written to disk.

#### Q. You want a quick one-commit copy of a large repository on your own disk and run the command below. What do you get?

```bash
git clone --depth 1 mono mono-shallow
```

* [x] A full clone, with the warning `--depth is ignored in local clones; use file:// instead.`
* [ ] A shallow clone with one commit, created with hard links
* [ ] An error, because `--depth` only works with an `ssh://` or `https://` URL
* [ ] A clone with one commit but no working tree

> **Explanation:** Cloning from a plain local path copies or hard-links the object store directly, which ignores `--depth`. Use `file:///path/to/mono` so the clone goes through the transport and the shallow clone works.

#### Q. What is the key difference between `git clone --filter=blob:none <url>` and `git clone --depth 1 <url>`?

* [ ] The filtered clone has only the newest commit; the shallow clone has every commit but no blobs
* [ ] Both have full history; the filter only skips files above a size limit
* [ ] The filtered clone never has file contents, so `git checkout` fails
* [x] The filtered clone has all commits and trees and fetches blobs on demand; the shallow clone truncates history

> **Explanation:** A blob-less partial clone keeps complete history, so `git log` works, and downloads file contents when checkout or diff needs them. A depth-1 clone has a single grafted commit until you run `git fetch --unshallow`.

#### Q. `echo 'hello world' | git hash-object --stdin` prints `3b18e512dba79e4c8300dd08aeb37f8e728b8dad`. Which command prints the same hash?

* [ ] `printf 'hello world\n' | sha1sum`
* [ ] `printf 'blob 11\0hello world' | sha1sum`
* [x] `printf 'blob 12\0hello world\n' | sha1sum`
* [ ] `printf 'hello.txt\0hello world\n' | sha1sum`

> **Explanation:** An object ID is the SHA-1 of a header (type, space, byte length, NUL byte) followed by the content. `echo` adds a newline, making the content 12 bytes, and file names are never part of a blob.

#### Q. In an empty repository you run the commands below. What does the last command print?

```bash
echo 'hello world' > hello.txt
git hash-object hello.txt
git cat-file -t 3b18e512
```

* [ ] `blob`
* [ ] `file`
* [ ] `3b18e512dba79e4c8300dd08aeb37f8e728b8dad`
* [x] `fatal: Not a valid object name 3b18e512`

> **Explanation:** Without `-w`, `git hash-object` only computes the ID and writes nothing to `.git/objects`. After `git hash-object -w hello.txt` (or `git add`), `git cat-file -t` prints `blob`.

#### Q. You run `git add notes.txt` in a new repository but do not commit. Which statement is true?

* [ ] Nothing is written to `.git/objects` until `git commit` runs
* [x] A blob for the content already exists in `.git/objects`, and the index points to it
* [ ] The index itself stores a full copy of the file's contents
* [ ] A blob, a tree and a commit are written, but no branch points to them yet

> **Explanation:** Staging writes (or reuses) the blob immediately and records the path, mode and blob ID in the index. `git commit` later writes the tree and commit objects and moves the branch.

#### Q. `greeting.txt` is already committed. You run the commands below. How many new objects does `git add` write?

```bash
cp greeting.txt copy.txt
git add copy.txt
```

* [x] None; the index entry for `copy.txt` points at the existing blob
* [ ] One blob for the contents of `copy.txt`
* [ ] One blob for `copy.txt` and one tree for the directory
* [ ] One blob, one tree and one commit

> **Explanation:** Identical bytes produce an identical blob ID, so both index entries share one blob. The new name only appears in a tree object when you commit.

#### Q. `hello.txt` sits in the repository root. You run `git mv hello.txt greeting.txt` and commit. How many new objects does the commit add?

* [ ] One: a rename record pointing at the old blob
* [ ] Three: a new blob, a new root tree and the commit
* [x] Two: a new root tree and the commit, reusing the existing blob
* [ ] Four: a new blob, two new trees and the commit

> **Explanation:** Names live in trees, so a rename changes only the root tree while the content blob is reused. Git stores no rename objects; rename detection is computed later by `git diff` or `git log`.

#### Q. You change one line in `src/main.py`, the only change, and commit. Which new objects are created?

* [ ] A delta object for `main.py` and a commit
* [ ] A new blob and a commit, with the existing trees updated in place
* [ ] A new blob, a new `src` tree and a commit
* [x] A new blob, a new `src` tree, a new root tree and a commit

> **Explanation:** Objects are immutable and a tree's ID depends on its entries, so a new blob forces new trees all the way up to the root. Unchanged sibling blobs and trees are reused.

#### Q. You run `chmod +x deploy.sh` without editing its contents and commit. What changes in the object database?

* [ ] A new blob, because the file mode is part of the blob header
* [x] A new tree listing the same blob with mode `100755`, plus a new commit
* [ ] Nothing, because Git ignores permission changes unless you use `--chmod`
* [ ] Only the index entry changes, so there is nothing to commit

> **Explanation:** A blob holds only bytes, while the tree entry records the mode, so the same blob ID reappears under mode `100755`. Git only ignores the change if `core.fileMode` is `false`.

#### Q. You run the commands below in a clean repository. What does `git status --short` print?

```bash
mkdir empty
git add empty
git status --short
```

* [ ] `A  empty/`
* [ ] `?? empty/`
* [ ] `fatal: pathspec 'empty' did not match any files`
* [x] Nothing

> **Explanation:** The index and trees record files (and gitlinks), and a directory exists in a tree only when it contains something. An empty directory is invisible to Git, which is why projects add a placeholder such as `.gitkeep`.

#### Q. A loose blob has the ID `3b18e512dba79e4c8300dd08aeb37f8e728b8dad`. Where is it stored?

* [x] `.git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad`
* [ ] `.git/objects/3b18e512dba79e4c8300dd08aeb37f8e728b8dad`
* [ ] `.git/objects/blob/3b18e512dba79e4c8300dd08aeb37f8e728b8dad`
* [ ] `.git/refs/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad`

> **Explanation:** Loose objects are fanned out into subdirectories named after the first two hex characters of the ID, with the remaining 38 characters as the file name. The object type is stored inside the object, not in the path.

#### Q. You run `cat` on that loose object file. Why is the output unreadable?

* [ ] It is encrypted with a key stored in `.git/config`
* [ ] It holds a delta against another version of the file
* [x] It is the zlib-compressed bytes of `blob 12\0hello world\n`, header included
* [ ] It is stored in the same binary format as `.git/index`

> **Explanation:** Each loose object is its header plus content, compressed with zlib. Read it with `git cat-file -p` (content) or `-t` and `-s` (type and size); deltas are only used inside packfiles.

#### Q. You create two tags on the same commit as shown. What do the two `cat-file` commands print?

```bash
git tag light
git tag -a v1 -m "first release"
git cat-file -t light
git cat-file -t v1
```

* [ ] `tag`, then `tag`
* [ ] `commit`, then `commit`
* [ ] `tag`, then `commit`
* [x] `commit`, then `tag`

> **Explanation:** A lightweight tag is only a ref pointing at the commit, while an annotated tag's ref points at a tag object that stores the tagger, date, message and target commit.

#### Q. For the annotated tag `v1` from the previous question, which statement about `git rev-parse v1` and `git rev-parse v1^{commit}` is true?

* [x] They print different IDs: first the tag object's, then the tagged commit's
* [ ] They print the same ID, because a tag is only an alias for a commit
* [ ] The first fails, because annotated tags must be peeled with `^{}`
* [ ] The second prints the ID of the tag's own tree

> **Explanation:** `refs/tags/v1` stores the tag object's ID; `^{commit}` (or `^{}`) peels the tag to the commit it points at, the same ID shown on the `object` line of `git cat-file -p v1`.

#### Q. You commit, then run `git commit --amend -m "two"` without changing any files. Which statement is true?

* [ ] The commit ID stays the same, because the snapshot is unchanged
* [ ] The tree ID changes, because the message is stored in the tree
* [ ] The existing commit object is rewritten in place with the new message
* [x] The tree ID is unchanged, but a new commit object with a new ID is created

> **Explanation:** A commit's ID covers its tree, parents, author, committer (with timestamps) and message, so a new message means a new commit. The old commit is untouched and stays reachable from the reflog as `HEAD@{1}`.

#### Q. You run `git checkout --detach` and then the commands below. What do they print?

```bash
cat .git/HEAD
git symbolic-ref HEAD
```

* [ ] `ref: refs/heads/main`, then `refs/heads/main`
* [x] A raw commit ID, then `fatal: ref HEAD is not a symbolic ref`
* [ ] `ref: HEAD`, then `HEAD`
* [ ] A raw commit ID, then `refs/heads/main`

> **Explanation:** Normally `.git/HEAD` is a symbolic ref (`ref: refs/heads/main`). When detached it holds a commit ID directly, so `git symbolic-ref HEAD` fails; `git symbolic-ref -q HEAD` fails silently, which is handy in scripts.

#### Q. In a fresh clone whose remote default branch is `main`, what does `git symbolic-ref refs/remotes/origin/HEAD` print?

* [x] `refs/remotes/origin/main`
* [ ] `origin/main`
* [ ] `ref: refs/heads/main`
* [ ] The commit ID at the tip of `origin/main`

> **Explanation:** `origin/HEAD` is a symbolic ref recording the remote's default branch; its file contains `ref: refs/remotes/origin/main` and `symbolic-ref` prints the full target. `git rev-parse --abbrev-ref origin/HEAD` would print `origin/main`.

#### Q. A deploy script runs `cat .git/refs/heads/main` to get the current commit. After `git gc` it fails with `No such file or directory`, yet `git log main` still works. What is the robust fix?

* [ ] Run `git checkout main` before the script so the file is recreated
* [ ] Read `.git/HEAD` instead, because it always contains a commit ID
* [x] Use `git rev-parse refs/heads/main`, which also reads `.git/packed-refs`
* [ ] Turn off ref packing by running `git gc --no-prune` instead

> **Explanation:** `git gc` packs refs into `.git/packed-refs` and removes the loose ref files. Commands such as `git rev-parse` resolve both loose and packed refs, so scripts should not read ref files directly.

#### Q. A repository contains only loose objects, all reachable. You run `git gc` and then `git count-objects -v`. What do you expect?

* [x] `count: 0`, with every object counted under `in-pack` in a `.pack` file indexed by an `.idx`
* [ ] An unchanged `count`, because `gc` only deletes unreachable objects
* [ ] `count: 0` and `in-pack: 0`, because `gc` deletes objects that were already pushed
* [ ] `packs: 0`, because packfiles are only created by `git push` and `git fetch`

> **Explanation:** `gc` repacks reachable objects into a packfile, which uses delta compression to save space, plus an `.idx` index for fast lookup. Objects are never deleted just because a remote also has them.

#### Q. You amended a commit a minute ago and then ran `git gc`. Is the pre-amend commit still in the repository?

* [x] Yes; the reflog still references it, so `gc` keeps it
* [ ] No; `gc` deletes every commit not reachable from a branch or tag
* [ ] No; `--amend` deleted the old commit immediately
* [ ] Yes, but only as a loose file that `git cat-file` cannot read

> **Explanation:** Reflog entries count as reachability roots, and `gc` prunes unreachable objects only after a grace period (two weeks by default), so recent rewrites can be recovered with `git reflog`.

#### Q. You stage a file, edit it, and stage it again without committing. What does `git fsck` report?

```bash
echo v1 > d && git add d
echo v2 > d && git add d
git fsck
```

* [ ] Nothing, because the second `git add` overwrote the first blob in place
* [x] `dangling blob <hash>`, for the blob holding `v1`
* [ ] `missing blob <hash>`, because the index lost a reference
* [ ] `dangling commit <hash>`, because each `git add` creates a commit

> **Explanation:** The first `git add` wrote a blob that no index entry, tree or ref references any more, so `fsck` lists it as dangling. `gc` prunes such objects once they are older than the grace period.

#### Q. You committed a secret, ran `git reset --hard HEAD~1`, and have not pushed. Which command sequence removes the commit from the local object database right away?

* [ ] `git gc`
* [ ] `git prune`
* [x] `git reflog expire --expire=now --all && git gc --prune=now`
* [ ] `git reset --hard HEAD~1 && git gc --aggressive`

> **Explanation:** The reflog still references the commit, so `gc` and `prune` keep it. Expiring the reflog makes it unreachable, and `--prune=now` deletes unreachable objects without waiting for the grace period.

#### Q. You build a commit with plumbing commands as shown. Why does `git log` not list it?

```bash
tree=$(git write-tree)
git commit-tree "$tree" -p HEAD -m "manual"
git log --oneline
```

* [ ] `commit-tree` only prints a preview and then discards the object
* [ ] `write-tree` needs `git add -A` first, so the commit was empty
* [ ] The commit becomes visible only after `git gc` packs it
* [x] The commit exists, but no ref points to it until `git update-ref` moves one

> **Explanation:** `write-tree` writes a tree from the index and `commit-tree` writes a commit and prints its ID, but neither moves a branch. `git update-ref refs/heads/main <id>` is the step that makes it part of `main`.

#### Q. On `main`, `git merge x` stops with a conflict in `a`. What does `git cat-file -p :2:a` print?

* [x] The version of `a` from `main` (ours)
* [ ] The common ancestor's version of `a`
* [ ] The version of `a` from `x` (theirs)
* [ ] The working-tree file with conflict markers

> **Explanation:** During a conflict the index holds stage 1 (base), stage 2 (ours, the branch you are on) and stage 3 (theirs), all visible with `git ls-files -s`. The file with markers exists only in the working tree.

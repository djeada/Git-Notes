## Dangerous Commands

Some git commands are powerful but risky because they can rewrite history, move branch tips, or discard work. Used carelessly, they break open reviews, hide teammates’ changes, and make it hard to trace what actually shipped. Treat history edits as exceptional: prefer additive fixes for shared code, keep risky changes local until you’re sure, and coordinate before publishing them. Always back up your branch, read prompts carefully, and verify the target you’re changing. In short, these tools are essential for cleanup—just use them sparingly, with safeguards and team alignment.

### Rebase (`git rebase`)

**Rebase** takes the commits on your branch and **replays** them on top of a new base (usually the updated `main`), creating **new commit IDs** for each replayed commit. It’s commonly used to keep a feature branch up to date or to clean up history (e.g., squash, reorder, edit commits).

```
Visual — before → after (rebasing feature onto main)

BEFORE
main:    A──B──C
               \
feature:        D──E

AFTER
main:    A──B──C
                 └──D'──E'   (D,E replayed; new hashes)
```

**Does this rewrite history?**
Yes. Rebased commits get **new hashes**. If others already pulled the old history, you’ll create divergence.

**When is it safe?**
Safe on **local/unshared** branches. If you must rebase a published branch, coordinate and use `git push --force-with-lease`.

#### Advantages

* Produces a **linear, easy-to-read** history.
* Lets you **resolve conflicts once** against the latest base.
* Interactive mode (`git rebase -i`) enables **squash/reword/reorder**.

#### Disadvantages

* **History rewrite** can confuse collaborators and break open PRs.
* Risk of **lost work** if conflicts are mishandled.
* Requires a **force push** to publish rewritten commits.

#### When to Use

* Before opening a PR, to refresh a feature branch onto `main`.
* To curate commits with `rebase -i`.
* Avoid on **shared/protected** branches; prefer merge if others already built on your branch.

### Reset (`git reset`)

**Reset** moves the current branch (and possibly your index and working tree) to another commit. Mode determines what happens to your files: `--soft` keeps changes staged, `--mixed` (default) keeps changes unstaged, `--hard` discards them.

```
Visual — before → after (moving HEAD from C to B)

BEFORE
branch: A──B──C   (HEAD)

AFTER --soft
branch: A──B   (HEAD)
        └─ changes from C kept **staged**

AFTER --mixed (default)
branch: A──B   (HEAD)
        └─ changes from C kept **unstaged**

AFTER --hard
branch: A──B   (HEAD)
        (changes from C **discarded**)
```

**Does this rewrite history?**
Yes, the branch pointer moves. On shared branches this **rewrites public history**.

**How is this different from revert?**
`reset` **moves** your branch pointer; `revert` **adds a new commit** that undoes prior changes. Use `revert` on **published** history.

#### Advantages

* Quick way to **undo** the last commit(s) locally.
* Adjust what’s **staged vs. unstaged** without editing files.

#### Disadvantages

* `--hard` can **destroy work**.
* Using reset on shared branches **breaks collaborators**.

#### When to Use

* Local cleanups (e.g., “oops, wrong commit”) — `--soft` or `--mixed`.
* Reserve `--hard` for disposable work or after backing up.
* Use `git revert` to undo commits that are already public.

### Force Push (`git push --force` / `--force-with-lease`)

**Force push** publishes a rewritten local history by **overwriting** the remote branch. It’s typically needed after a rebase or amend.

```
Visual — before → after (overwriting remote with rebased local)

BEFORE (diverged)
origin/feature: A──B──C──D
local/feature : A──B──X──Y   (HEAD)

AFTER
origin/feature: A──B──X──Y
local/feature : A──B──X──Y   (HEAD)
```

**Is it safe?**
Use `--force-with-lease` to refuse overwriting new remote work you haven’t seen. **Coordinate** with teammates first.

**Where is it allowed?**
Generally fine on **your own feature branches**. Avoid on `main` or protected branches.

#### Advantages

* Lets you publish a **clean, linear** history after rebase/amend.
* Removes accidental commits from the remote branch.

#### Disadvantages

* Can **erase others’ work** if they pushed meanwhile.
* Breaks PR comment threads anchored to old commits.

#### When to Use

* After `rebase -i` or `commit --amend` on **your branch**.
* With `--force-with-lease`, after confirming no one else pushed.

### History Rewrite (`git filter-branch` / alternatives)

**filter-branch** rewrites repository **history commit-by-commit** (e.g., to remove secrets, large files, or change authors). Modern practice prefers **`git filter-repo`** (faster/safer) or **BFG Repo-Cleaner** for common cases.

```
Visual — before → after (remove a secret added in C)

BEFORE
A──B──C[secret]──D──E

AFTER (new hashes)
A'──B'──C'──D'──E'   (secret purged from all affected commits)
```

**Will this change commit IDs?**
Yes—potentially **across most of history**. All clones/forks become incompatible until updated.

**What else must I do?**
Rotate any **exposed credentials**, and **force-push** rewritten branches. Inform downstreams to reclone or run `git fetch --all && git reset --hard origin/main` (replace `origin/main` with the correct branch as needed) to realign their local history.

#### Advantages

* Removes **sensitive data** or bloat everywhere in history.
* Can significantly **shrink** repository size.

#### Disadvantages

* **Disruptive** to collaborators and forks; breaks old SHAs.
* Easy to make mistakes; recovery can be tedious.

#### When to Use

* Security/legal incidents; purging large binaries.
* On repos you control and after thorough backups.
* Prefer `git filter-repo` or **BFG** over raw `filter-branch` for safety/speed.

> **Note:** `git filter-repo` and **BFG Repo-Cleaner** are not included with Git by default.  
> - To install `git filter-repo`, see [https://github.com/newren/git-filter-repo](https://github.com/newren/git-filter-repo) or install via `pip install git-filter-repo`.  
> - To use BFG, download the jar from [https://rtyley.github.io/bfg-repo-cleaner/](https://rtyley.github.io/bfg-repo-cleaner/).
### Amend (`git commit --amend`)

**Amend** replaces the **most recent commit** with a new one (e.g., fix message, add a forgotten file). The result is a **new commit ID**.

```
Visual — before → after (replace last commit)

BEFORE
A──B   (HEAD)

AFTER
A──B'  (HEAD)   (B replaced with B')
```

**Does this rewrite history?**
Yes. If the old commit was pushed, you’ll need a **force push** and must coordinate.

**What’s a safe workflow?**
Amend **before** pushing, or amend your branch and publish with `--force-with-lease`.

#### Advantages

* Quick fix for **typos, messages, or small omissions**.
* Keeps history **tidy** by avoiding extra “fixup” commits.

#### Disadvantages

* Rewrites history, which can **disrupt collaborators**.
* Amending repeatedly can confuse review threads.

#### When to Use

* Local changes not yet pushed.
* Minor corrections to your own branch (with coordinated force push).
* Avoid on shared/protected branches; add a new commit instead.

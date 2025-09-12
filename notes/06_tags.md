## Git Tags

Tags mark *exact* commits. They’re perfect for releases, rollbacks, changelogs, and CI/CD triggers. Unlike branches, tags don’t move—ever—so you can always point to the exact build you shipped.

Think of a branch as a **bookmark you keep moving**, and a tag as a **sticky note glued to a page**.

```
A -- B -- C -- D -- E  (main)
          ^ 
          |
        v1.2.0  ← frozen pointer to commit C
```

### Mental model

* A *branch* allows new features to be developed safely, while skipping branches risks breaking the main code; for example, a login feature can be built in a branch while production stays stable.
* A *tag* marks a fixed commit for easy reference, but without tags it is harder to find exact release points; for instance, version 1.0 can be recalled instantly with a tag.
* A lightweight tag gives a simple pointer to a commit, while omitting it removes a quick way to bookmark work; for example, the project’s first setup can be tagged this way.
* An annotated tag adds a message, tagger, and date, but without it, release details are lost; for example, version 2.0 can store who created the release and when.

### Creating Tags

Tags are markers you can place on commits — often used for releases, milestones, or just to remember an important point in your project’s history. Git supports three main flavors: lightweight, annotated, and signed.

#### 1. Lightweight Tag (quick and simple)

A lightweight tag is basically a named pointer to a commit. It doesn’t store extra info (no tagger, no message).

Tag the latest commit (HEAD):

```bash
git tag v2.0.0
```

Tag a specific commit (using its short hash):

```bash
git tag v2.0.0 b4d373a
```

Check what the tag points to:

```bash
git show --no-patch v2.0.0
```

Example output:

```
tag v2.0.0
Tagger: (none - lightweight tag)
commit b4d373k8990g2...
Author: Jane Dev <jane@example.com>
Date:   Mon Sep 1 12:34:56 2025 +0000

    Merge feature: new billing flows
```

Notice how there’s no tagger or message section — Git just jumps straight to the commit info. Lightweight tags are fine for personal bookmarks, but not recommended for formal releases.

#### 2. Annotated Tag (the usual choice for releases)

An annotated tag creates a real tag object that stores metadata: who created it, when, and an optional message.

At HEAD:

```bash
git tag -a v2.0.0 -m "v2.0.0: usage-based billing, new invoices"
```

At a specific commit:

```bash
git tag -a v2.0.0 -m "v2.0.0: ..." b4d373k
```

Inspect it:

```bash
git show v2.0.0
```

Sample output:

```
tag v2.0.0
Tagger: Jane Dev <jane@example.com>
Date:   Mon Sep 1 12:45:00 2025 +0000

v2.0.0: usage-based billing, new invoices

commit b4d373k8990g2...
Author: Jane Dev <jane@example.com>
Date:   Mon Sep 1 12:34:56 2025 +0000

    Merge feature: new billing flows
```

Here you see two parts:

* The **tag object** at the top (who created the tag, when, and why).
* The **commit** the tag points to, shown below.

This extra metadata is super handy for tracking releases and automation.

#### 3. Signed Tag (for authenticity and trust)

If you have GPG or SSH signing set up, you can cryptographically sign tags. This proves the tag really came from you.

```bash
git tag -s v2.0.0 -m "Release v2.0.0"
git tag -v v2.0.0   # verify the signature
```

* `-s` creates a signed tag.
* `-v` verifies the signature and tells you if it matches a known key.

Signed tags are especially useful in open-source or enterprise projects where you need strong guarantees about who made a release.

👉 One last note: tags don’t get pushed automatically. To share them, run:

```bash
git push origin v2.0.0      # push a single tag
git push origin --tags      # push all tags
```

### Seeing and Finding Tags

Tags pile up quickly in a project, so Git gives you ways to list, filter, and sort them.

#### List all tags

```bash
git tag
```

This just prints every tag name in your repo:

```
v1.0.0
v1.1.0
v2.0.0
v2.1.0-rc.1
```

#### Filter tags by pattern

```bash
git tag -l "v2.*"
```

Only tags starting with `v2.` show up:

```
v2.0.0
v2.1.0-rc.1
```

You can also match other patterns, like release candidates:

```bash
git tag -l "*-rc.*"
```

#### Show messages next to tags

Annotated tags can have messages, and `-n` lets you preview them:

```bash
git tag -n
```

Output:

```
v1.0.0   First stable release
v2.0.0   Usage-based billing, new invoices
```

You can also limit or filter:

```bash
git tag -n9 -l "v2.*"
```

This shows the first 9 lines of each tag message, but only for tags starting with `v2.`. Great when you want quick release notes.

#### Sort tags semantically

Normal sorting treats `v1.10.0` as *before* `v1.9.9` (because text sorting sees `1` vs `9`). Version-aware sort fixes that:

```bash
git tag --sort=-v:refname
```

Now the latest version appears first:

```
v2.1.0-rc.1
v2.0.0
v1.10.0
v1.9.9
```

#### Find the nearest tag to the current commit

```bash
git describe --tags
```

Example output:

```
v2.0.0-5-gdeadbeef
```

That means:

* the latest tag reachable is `v2.0.0`
* you’re 5 commits ahead of it
* the current commit starts with `deadbeef`

This is super useful in CI/CD pipelines for naming builds like `2.0.0+5`.


### Using Tags in Real Workflows

Tags are the backbone of releases, hotfixes, and rollbacks. Here are the most common ways you’ll use them in practice.

#### Cutting a Release + Triggering CI

The usual flow:

1. Merge your release PR into `main`.
2. Create a tag at that commit.
3. Push the tag — CI/CD kicks off.

Example:

```
A -- B -- C -- D -- E -- F  (main)
                ^           ^
               v1.1.0      v1.2.0   ← each tag starts a release build
```

Commands:

```bash
git tag -a v1.2.0 -m "v1.2.0: new billing"
git push origin v1.2.0
```

What happens next: most CI platforms (GitHub Actions, GitLab CI, CircleCI, etc.) can listen for “tag push” events. A tag like `v1.2.0` often triggers a pipeline that builds artifacts, signs them, and publishes a release.

#### Hotfix from a Release Tag

Say a bug slipped into production. You want to patch *exactly* what was shipped.

Instead of checking out the tag directly (which gives you a detached HEAD), make a branch starting from it:

```bash
git switch -c hotfix/invoice-rounding v1.2.0
```

Now you can fix, commit, and PR back into `main` or your release branch.

Interpretation: you’re working from the exact code you shipped last time — no surprise commits from `main` sneaking in.

#### Rolling Back to a Known-Good Version

Sometimes the latest release misbehaves. Tags let you roll back safely.

See what commit a tag points to:

```bash
git show --no-patch v1.2.0
```

Deploy tooling (Heroku, Kubernetes, GitHub Actions, etc.) often accepts a tag name directly:

```bash
deploy --ref v1.2.0
```

That redeploys the *exact same bytes* as last time. No guessing.

#### Generating a Changelog Between Releases

Need release notes or just want to see what changed? Compare two tags:

```bash
git log --oneline v1.1.0..v1.2.0
```

Shows commit subjects:

```
abc123  Fix invoice rounding error
def456  Add usage-based billing
```

Or see file-level stats:

```bash
git diff --stat v1.1.0..v1.2.0
```

Output:

```
billing.js    | 20 +++++++++++---------
invoice.test  | 12 ++++++++--
2 files changed, 22 insertions(+), 10 deletions(-)
```

Great for writing changelogs.

#### Checking Out a Tag (and Avoiding Detached HEAD Confusion)

To browse code at a tag:

```bash
git switch --detach v1.2.0
# or: git checkout v1.2.0
```

Now you’re in a detached HEAD state — safe for read-only browsing, not great for new commits.

ASCII reminder:

```
(main) ──●──●──●──●
              ↑
            v1.2.0  (HEAD is detached here)
```

If you do want to commit, make a branch first:

```bash
git switch -c fix-from-v1.2.0 v1.2.0
```

That way your work doesn’t get “lost in limbo.”

### Moving, renaming, and deleting tags

#### Move a tag to a different commit (local)

```bash
git tag -f v2.0.0 NEWCOMMIT
```

If it’s already on the remote, you must **replace** it there too:

```bash
git push origin -f v2.0.0   # force updates tag ref on remote
```

⚠️ Caution:

* Force-moving public release tags can break consumers and automation. Prefer a **new tag** (e.g., `v2.0.1`) and deprecate the old.

#### Rename a tag (no direct rename; recreate)

```bash
git tag new-name old-name
git tag -d old-name
git push origin new-name
git push origin :refs/tags/old-name   # or: git push origin --delete old-name
```

#### Delete tags

Local:

```bash
git tag -d v2.0.0
```

Remote:

```bash
git push origin --delete v2.0.0
# or:
git push origin :refs/tags/v2.0.0
```

Interpretation:

* Deleting remote tags is sensitive. Communicate widely if it’s a release tag.

### Power viewing

See where tags sit in history:

```bash
git log --graph --decorate --oneline --all
```

You’ll see tags in-line:

```
* 3fa1c2d (tag: v1.2.0, origin/main, main) Merge ...
* 0b7a3ff Add invoices API
* 9e0a1a1 (tag: v1.1.0) Release v1.1.0
```

Find tags with notes:

```bash
git tag -n | grep "billing"
```

Show only tags on the current branch:

```bash
git tag --merged
```

### Practical Naming for Tags

Pick a naming style that both humans and automation tools will love.

* Using *semantic versions* like `vMAJOR.MINOR.PATCH` makes it clear what kind of change occurred, while skipping this practice leaves version meaning ambiguous; for example, `v2.0.1` signals a patch to version 2.0.
* Adding *pre-release* labels such as `-rc.1` or `-beta.2` lets pipelines handle them differently, while without them it is harder to separate testing builds from production releases; for example, `v2.1.0-rc.1` can be built but not shipped live.
* Choosing a *date-based* scheme like `release-2025-09-12` gives simple ordering for frequent releases, while omitting structure can make it unclear when versions were cut; for example, daily builds can be tagged by date.
* Protecting the *public namespace* for tags ensures only automated systems create official ones, while leaving it open risks personal tags being mistaken for releases; for example, locking down all `v*` tags prevents accidental production deployments.

Why bother with all this? Because automation thrives on predictable names — and so do humans reading the history months later.

### Common “Why Isn’t This Working?” Pitfalls

**“I created the tag but CI didn’t run.”**

You probably forgot to push the tag. Remember:

```bash
git push origin v2.0.0
```

Also double-check your CI config — many pipelines only listen for certain patterns (`v*`, `release-*`, etc.).

**“I pushed but my teammate still can’t see it.”**

They need to fetch tags explicitly:

```bash
git fetch --tags
```

By default, `git fetch` doesn’t grab new tags unless they’re tied to commits being fetched.

**“`git show` doesn’t display my tag message.”**

That means you made a **lightweight tag**. Those don’t have messages. Use an annotated tag next time:

```bash
git tag -a v2.0.0 -m "Release v2.0.0"
```

**“I checked out a tag and my commits disappeared.”**

That’s the classic **detached HEAD** trap. Tags don’t move, so commits made in this state get “orphaned.” The fix: create a branch at the tag before committing:

```bash
git switch -c fix-from-v2.0.0 v2.0.0
```

**“We moved a release tag and broke everything.”**

Never rewrite or move public tags. Once something like `v2.0.0` is out in the world, it’s immutable. If you need to patch, bump the version (`v2.0.1`) instead.

### Real-world mini recipes (copy/paste)

Create an annotated release tag at HEAD and push:

```bash
git tag -a v2.3.0 -m "v2.3.0: SSO + revamped audit logs"
git push origin v2.3.0
```

Push only release tags automatically when pushing commits:

```bash
git push --follow-tags
```

Generate a changelog between releases:

```bash
git log v2.2.0..v2.3.0 --pretty=format:"- %s (%h)"
```

Diff code between releases:

```bash
git diff --stat v2.2.0..v2.3.0
```

Find latest version tag (nearest reachable):

```bash
git describe --tags --abbrev=0
```

Clean up a mistaken remote tag:

```bash
git tag -d v2.3.0
git push origin --delete v2.3.0
```

Retag correctly (same name, new commit), and push forcefully:

```bash
git tag -a -f v2.3.0 -m "v2.3.0: hotfix included" NEWCOMMIT
git push -f origin v2.3.0
```

(Again: **avoid** for public releases; prefer `v2.3.1`.)

### Why teams use tags in practice?

* A *tag* makes releases reproducible by serving as the single reference for what was shipped, while without tags it can be unclear which commit actually represents a release; for example, version 1.2.3 can always be rebuilt exactly as delivered.
* Using tags enables *CI/CD* systems to trigger builds and deployments consistently, but without them automation may behave unpredictably; for instance, a pipeline can sign and publish artifacts every time a new tag is pushed.
* Deploying by tag provides instant and precise *rollbacks*, while skipping tags forces guesswork about which commit is safe; for example, reverting to a “v1.1” tag restores the prior release immediately.
* Generating changelogs and audits from tags gives easy diffs and logs between versions, while omitting tags makes history harder to track; for example, comparing “v1.0” and “v1.1” highlights all changes since the last release.
* Signed tags enhance *security* by proving provenance, while unsigned references cannot guarantee authenticity; for example, a GPG-signed “v2.0” tag confirms it came from a trusted maintainer.


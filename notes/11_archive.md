## Git Archive

`git archive` is your clean-room packager. It snapshots exactly what Git tracks at a commit—no `.git` folder, no stray build junk, no temp files. This means you can hand someone a tidy source bundle or ship code to a server without dragging history along.

Think of it as a camera: you point it at a commit, click, and get a tar/zip of only the files that belong in that picture. It’s great for releases, vendor handoffs, monorepo subtrees, and “just give me the code” moments.

Because the archive is created from the repository’s tree, not your working directory, it’s consistent and repeatable, which makes ops and audits much happier.
```
+------------------------+                   +------------------------+
|  YOUR GIT REPOSITORY   |                   |      project.tar       |
|                        |   git archive     |                        |
|  .git/    src/         |    ========>      |  src/    README        |
|  README   docs/        |                   |  docs/                 |
|                        |                   |                        |
+------------------------+                   +------------------------+
```

The left box is your repo with history + working files; the right box is the clean export—no `.git`, only tracked content as of the chosen commit.

### Quick snapshots (branch, tag, or commit)

You pick a point in time (HEAD, a branch, a tag, a SHA), and you get a bundle. This is the “just give me today’s code” path.

```bash
# Zip of whatever you're on now
git archive -o project.zip HEAD

# Tar.gz from a release tag, with a friendly top-level folder name
git archive --format=tar --prefix=project-1.4.2/ v1.4.2 | gzip > project-1.4.2.tar.gz

# Peek inside without extracting
tar -tzf project-1.4.2.tar.gz | head
# output (example):
# project-1.4.2/
# project-1.4.2/README.md
# project-1.4.2/src/main.py
# project-1.4.2/src/utils/__init__.py
```

A tiny mental model:

```
commit (tree) ──► reproducible source package
      ^
      └── HEAD / tag / SHA you chose
```

### Partial exports

You don’t have to ship the whole repo. Target a file, a folder, or a pattern and keep the archive lean.

```bash
# One file from HEAD
git archive -o config.zip HEAD config.yml

# One directory
git archive -o api.zip HEAD services/api/

# Glob patterns (quote so your shell doesn’t expand it)
git archive -o py-src.zip HEAD ':(glob)src/**/*.py'
```

Visualization:

```
+------------------------+     git archive     +-----------------------+
|      BRANCH HEAD       |    ============>    |   config.zip          |
| config.yml             |                     | config.yml            |
| src/                   |                     |                       |
+------------------------+                     +-----------------------+
```

### Prefixes that keep things organized

A prefix puts everything under a single folder inside the archive—super handy for releases and avoiding file collisions when someone extracts.

```bash
git archive --prefix=project-name/ -o project.zip HEAD
```

```
+------------------------+                   +------------------------------+
|  YOUR GIT REPOSITORY   |                   |        project.zip           |
|                        |   git archive     |                              |
|  src/    README        |    ========>      |  project-name/src/           |
|  docs/                 |                   |  project-name/README         |
|                        |                   |                              |
+------------------------+                   +------------------------------+
```

### Stream it (no temp files, even to another box)

Because `git archive` writes to stdout by default, you can compress and ship in one go. Great for deployments or quick backups.

```bash
archive_name="project-$(date +%F).tar.gz"
git archive main | gzip | ssh user@host "cat > /deploy/$archive_name"
# output (local):
# (no local file created; archive landed on remote host)
```

Remote server view:

```
[repo @ dev box] --stdout--> [gzip] --ssh--> [/deploy/project-2025-09-13.tar.gz]
```

### Real-world flows that come up all the time

**1) Release tarball from a tag, nice folder name inside**

```bash
git archive --format=tar --prefix=myapp-2.0.0/ v2.0.0 | gzip > myapp-2.0.0.tar.gz
```

**2) Monorepo: ship just one service**

```bash
git archive -o payments.zip HEAD services/payments/
```

**3) Vendor handoff: exclude dev-only stuff using `.gitattributes`**
Create rules once, every archive stays clean.

```bash
# .gitattributes (commit this)
docs/private/**    export-ignore
*.psd              export-ignore
scripts/dev/**     export-ignore
VERSION            export-subst
```

And if `VERSION` contains a placeholder, it gets filled when you archive:

```text
# VERSION file in repo
MyApp $Format:%h$ ($Format:%ci$)
```

```bash
git archive -o clean.zip HEAD
# After extracting clean.zip, VERSION might read:
# MyApp 9f3a1c2 (2025-09-13 09:45:12 +0000)
```

**4) Zero-clone archive from a remote (server does the packing)**
Useful when you don’t want a local checkout.

```bash
git archive --remote=ssh://git@yourserver/opt/git/myrepo.git v1.4.2 | gzip > myrepo-1.4.2.tar.gz
# note: the remote must allow git-upload-archive over SSH
```

### Formats you’ll actually use

* `tar` → `project.tar` (uncompressed, fast on server-to-server)
* `tar.gz` → `project.tar.gz` (common for releases)
* `zip` → `project.zip` (nice for Windows users)

Pick explicitly, or let `-o`’s extension do the talking:

```bash
git archive --format=zip -o src.zip HEAD
git archive --format=tar HEAD > project.tar
```

### Deployment example, end-to-end

You want the latest `main` on a server, clean and quick.

```bash
# I. create tar from main and compress
git archive --format=tar main | gzip > project.tar.gz

# II. ship it
scp project.tar.gz user@server:/deployments

# III. unpack on the server
ssh user@server 'mkdir -p /apps/project && tar -xzf /deployments/project.tar.gz -C /apps/project'
```

Result: server gets only the tracked sources, with the exact tree from `main`.

### Sanity checks and small gotchas

```bash
# List contents of a tar.gz without extracting
tar -tzf project.tar.gz | head -10
```

* Untracked or ignored files are not included. If you rely on built assets, either track them (not ideal) or build on the target machine after extracting.
* Submodules aren’t pulled in automatically; they appear as empty directories in archives. Package them separately or run a build step that fetches their sources.
* Remote archiving depends on the server allowing `git-upload-archive` over SSH; most self-hosted setups do, some hosted platforms disable it.
* Executable bits are preserved; extended attributes aren’t. If permissions matter in production, set them in your deploy scripts.

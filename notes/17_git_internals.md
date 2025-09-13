## Git internals

Git stores your project as a graph of immutable objects. At the leaves are blobs: raw file contents with no filenames attached. Trees sit above blobs and act like directories; a tree is just a list that maps a filename and a mode to either another tree (subfolder) or a blob (file). Commits point to exactly one top-level tree—the snapshot of the entire project at that moment—plus zero or more parents (for merges), and some who/when/why metadata. Refs like `main` or `v1.2.0` are just human-friendly names that point at a commit’s ID. HEAD is a tiny text file telling Git which ref you have checked out. The magic is that object IDs are content-hashes, so if two files have identical bytes, they share the same blob; if a folder hasn’t changed, its tree can be reused; if nothing changed, the commit points at the same tree as before. That’s how Git is both space-efficient and lightning fast: reuse everywhere, append-only history, and lookups by hash.

Inside `.git/`, everything supports that model. You’ll see `objects/` holding the database (loose files at first, later packed into `.pack` files), `refs/` holding branch and tag pointers, `HEAD` pointing to the current branch, an `index` file acting as a staging area, and logs that remember where refs used to point (your safety net). Git never edits a blob or tree in place; it writes a brand-new object, then flips a pointer. That’s why operations feel atomic and why you can always walk backward. Think of it like Lego: blobs are bricks, trees are plates arranging bricks by name, commits are photos of the model with notes about who built it—Git keeps every photo and never scribbles on an old one.

```
Objects (content-addressed)
        ┌────────────┐
file →  │   blob     │  (bytes only)
        └─────┬──────┘
              │ referenced by name+mode
        ┌─────▼──────┐
folder→ │   tree     │  (entries: mode, type, hash, name)
        └─────┬──────┘
              │ root tree of snapshot
        ┌─────▼──────┐
history │  commit    │  (tree, parents, author, message)
        └────────────┘

names:
HEAD → refs/heads/main → <commit>

storage:
.git/
  HEAD
  index
  objects/
    12/34abcd...   (loose objects)
    pack/pack-xxxx.{pack,idx}
  refs/
    heads/main
    tags/v1.2.0
  logs/refs/heads/main  (reflog)
```

### Walk the objects by hand (tiny repo lab)

```bash
# start clean
mkdir toy && cd toy
git init
# output:
# Initialized empty Git repository in .../toy/.git/
```

Create a file and inspect what gets stored.

```bash
echo "hello world" > hello.txt
git status
# output:
# Untracked files:
#   hello.txt
```

Stage it; staging writes a blob object and records its hash in the index.

```bash
git add hello.txt

# See what’s staged and the blob ID behind it
git ls-files -s
# output (mode | stage | blob-hash | path):
# 100644 0000000000000000000000000000000000000000 <hash>  hello.txt
# (your hash will differ)
```

Peek at the blob (raw file content, no filename inside):

```bash
git cat-file -t <blob-hash>
# output:
# blob

git cat-file -p <blob-hash>
# output:
# hello world
```

Write the first tree from the index and look inside it.

```bash
git write-tree
# output:
# <tree-hash>

git cat-file -p <tree-hash>
# output (mode type   hash                                   name):
# 100644 blob  <blob-hash>    hello.txt
```

Make a commit that points at that tree.

```bash
echo "first" | git commit-tree <tree-hash> -m "initial snapshot"
# output:
# <commit-hash>

# Move the branch name to that commit
git update-ref refs/heads/main <commit-hash>
echo "ref: refs/heads/main" > .git/HEAD
```

Read the commit object:

```bash
git cat-file -p <commit-hash>
# output (example):
# tree <tree-hash>
# author You <you@example.com> 1699999999 +0000
# committer You <you@example.com> 1699999999 +0000
#
# initial snapshot
```

Now use the regular porcelain to keep going.

```bash
# Add another file and commit it the normal way
echo "bye" > bye.txt
git add bye.txt
git commit -m "add bye"
# output:
# [main 9f3a1c2] add bye
#  1 file changed, 1 insertion(+)
#  create mode 100644 bye.txt
```

See the new snapshot’s tree and all files’ blob hashes.

```bash
git ls-tree -r HEAD
# output:
# 100644 blob <hash1>    bye.txt
# 100644 blob <hash2>    hello.txt

git cat-file -p HEAD^{tree}
# output:
# 100644 blob <hash1>    bye.txt
# 100644 blob <hash2>    hello.txt
```

Show how commits stitch together.

```bash
git log --oneline --graph --decorate
# output:
# * 9f3a1c2 (HEAD -> main) add bye
# * a1b2c3d initial snapshot
```

Peek at pointers on disk.

```bash
cat .git/HEAD
# output:
# ref: refs/heads/main

cat .git/refs/heads/main
# output:
# 9f3a1c2...   (the commit ID for main)
```

A picture of what you just built:

```
refs/heads/main ──► (commit 9f3a1c2)
                      │
                      ├─ tree T2 (root)
                      │    ├─ 100644 blob <hash1>  bye.txt
                      │    └─ 100644 blob <hash2>  hello.txt
                      │
                      └─ parent (commit a1b2c3d)
                           └─ tree T1 with only hello.txt
```

### What the index (staging area) really holds

The index is a compact table: for each path, it stores mode, blob hash, and some metadata. That’s why “staged vs unstaged” is so fast—Git compares your working tree to the index, not to HEAD, and it can build a tree object directly from the index during commit.

```bash
git ls-files --stage
# output (mode stage hash path):
# 100644 0 <hash1> bye.txt
# 100644 0 <hash2> hello.txt
```

Stages other than `0` appear during merges to hold multiple versions of the same path (that’s how Git presents conflict hunks).

### Loose objects vs packfiles (storage evolution)

New objects start life as compressed files under `.git/objects/aa/bb...` where `aa` is the first two hex digits of the ID. Over time Git packs many objects into a single `.pack` file for space and speed; an accompanying `.idx` lets it locate an object quickly.

```bash
# Count loose objects
find .git/objects -type f | wc -l
# output (small number initially)

# Pack them
git gc
# output:
# Counting objects: ...
# Writing objects: ...
# Total ..., reused ...

# See the packs
ls .git/objects/pack
# output:
# pack-1234abcd.pack
# pack-1234abcd.idx
```

ASCII of that progression:

```
loose:
.git/objects/
  ab/cdef...   (one file per object)
  12/3456...

packed:
.git/objects/pack/
  pack-xxxx.pack   (many objects)
  pack-xxxx.idx    (index for lookups)
```

### Names are in trees, not blobs (why dedup works)

Two files with identical bytes anywhere in history share the same blob ID. Filenames and execute bits live in trees, so renames don’t rewrite blobs; Git just updates the tree entry to point the same blob under a new name.

```bash
cp hello.txt copy.txt
git add copy.txt
git ls-tree -r HEAD
# output (notice identical blob hash for both paths):
# 100644 blob <hash2> copy.txt
# 100644 blob <hash2> hello.txt
```

### Tags anchor names to objects (annotated vs lightweight)

Tags are refs too; annotated tags are objects with their own metadata and a pointer to another object (usually a commit). Lightweight tags are just a name pointing straight to a commit.

```bash
git tag -a v1.0 -m "first release"
git cat-file -p refs/tags/v1.0
# output:
# object <commit-hash>
# type commit
# tag v1.0
# tagger You <you@example.com> ...
#
# first release
```

### Quick “x-ray” recipes you’ll use

```bash
# Show the type/size/pretty of any object by hash
git cat-file -t <id>
git cat-file -s <id>
git cat-file -p <id>

# Show exactly which tree/parent(s) a commit has
git show --no-patch --pretty=raw HEAD

# Map paths to blob hashes at a commit (great for checksums)
git ls-tree -r --long HEAD

# Compare two trees without touching the working dir
git diff --name-status <commitA>^{tree} <commitB>^{tree}
```

### Mental model to keep

Blobs are bytes. Trees give names to blobs (and trees). Commits name a tree and connect it to parents. Refs name commits so humans don’t have to memorize hashes. The `.git/` folder is the whole universe: a content-addressed object store, a few tiny pointers, and logs. Everything else is a view on those pieces.

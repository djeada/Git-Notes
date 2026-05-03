## Git Internals

Git stores your project as a graph of immutable objects. Instead of storing changes as a sequence of file diffs, Git stores snapshots of your project. Each snapshot is built from content-addressed objects, meaning each object is identified by a hash of its contents.

At the bottom are **blobs**. A blob stores raw file contents only. It does not store the file name, path, permissions, or history.

Above blobs are **trees**. A tree is like a directory. It maps names to objects. Each tree entry contains a file mode, object type, object hash, and name. A tree can point to blobs, which represent files, or to other trees, which represent subdirectories.

Above trees are **commits**. A commit points to one top-level tree, which represents the full project snapshot at that moment. A commit also stores parent commit IDs, author and committer information, timestamps, and the commit message.

Refs such as `main`, `feature/login`, or `v1.2.0` are human-friendly names that point to object IDs, usually commit IDs. `HEAD` tells Git what you currently have checked out. Most of the time, `HEAD` points to a branch ref such as `refs/heads/main`.

The important idea is this:

```text
file bytes → blob
directory listing → tree
snapshot + history → commit
human name → ref
current checkout → HEAD
```

Git objects are immutable. Git does not edit an existing blob, tree, or commit in place. When something changes, Git writes new objects and then moves a pointer, such as a branch ref, to the new commit.

This is one reason Git is efficient:

* identical file contents reuse the same blob,
* unchanged directories can reuse the same tree,
* history is append-only,
* object lookup is fast because objects are addressed by hash.

A simple mental model:

```text
Blobs are file contents.
Trees are directories.
Commits are snapshots with history.
Refs are names for commits.
HEAD is where you are now.
```

### The `.git/` Directory

The `.git/` directory is the database and control center of a Git repository. Your working directory contains the checked-out files, but `.git/` contains the history, references, staging area, and recovery logs.

Common important files and directories:

```text
.git/
  HEAD
  index
  objects/
  refs/
  logs/
  config
```

### Important Parts

| Path            | Purpose                                              |
| --------------- | ---------------------------------------------------- |
| `.git/objects/` | Stores Git objects: blobs, trees, commits, and tags  |
| `.git/refs/`    | Stores branch and tag references                     |
| `.git/HEAD`     | Points to the current branch or directly to a commit |
| `.git/index`    | The staging area                                     |
| `.git/logs/`    | Reflogs that remember where refs used to point       |
| `.git/config`   | Repository-specific configuration                    |

New objects usually start as **loose objects** under `.git/objects/`. Later, Git may compress many objects into **packfiles** for better storage efficiency and faster transfer.

Example layout:

```text
.git/
  HEAD
  index
  objects/
    12/34abcd...
    pack/
      pack-xxxx.pack
      pack-xxxx.idx
  refs/
    heads/main
    tags/v1.2.0
  logs/
    HEAD
    refs/heads/main
```

The design is simple: Git stores immutable objects, then moves small pointers around.

That is why many Git operations feel atomic. Creating a commit writes new objects, then updates a ref. The old commit is still there. If a branch moves unexpectedly, the reflog often lets you recover it.

### Object Model

Git has four main object types:

```text
blob
tree
commit
tag
```

#### Blob

A blob stores file contents.

It does not know:

* the file name,
* the file path,
* whether the file was renamed,
* which commit it belongs to.

Example:

```text
blob = "hello world\n"
```

If two files have exactly the same bytes, they point to the same blob.

#### Tree

A tree represents a directory. It stores entries that map names to object IDs.

Each tree entry contains:

```text
mode type hash name
```

Example:

```text
100644 blob  <hash>  hello.txt
040000 tree  <hash>  src
```

The tree is where file names and modes live. That is why a blob can be reused under multiple names.

#### Commit

A commit represents a project snapshot plus history metadata.

A commit stores:

* the root tree,
* zero or more parent commits,
* author metadata,
* committer metadata,
* commit message.

A normal commit has one parent. The first commit has no parent. A merge commit usually has two or more parents.

Example commit content:

```text
tree <tree-hash>
parent <parent-commit-hash>
author You <you@example.com> 1699999999 +0000
committer You <you@example.com> 1699999999 +0000

Add hello.txt
```

The commit does not store a full copy of every file directly. It points to a tree, and that tree points to blobs and subtrees.

#### Tag Object

Git has two common types of tags:

* **lightweight tag**
* **annotated tag**

A lightweight tag is just a ref that points directly to an object, usually a commit.

An annotated tag is a real Git object. It has its own metadata, message, tagger, and a pointer to another object.

Annotated tags are useful for releases because they can be signed, inspected, and treated as first-class objects.

### High-Level Diagram

```text
Objects

        ┌────────────┐
file →  │    blob    │   raw file bytes only
        └─────┬──────┘
              │ referenced by name and mode
              ▼
        ┌────────────┐
dir  →  │    tree    │   entries: mode, type, hash, name
        └─────┬──────┘
              │ root tree of snapshot
              ▼
        ┌────────────┐
hist →  │   commit   │   tree, parent(s), author, message
        └────────────┘

Names

HEAD → refs/heads/main → commit
```

Another view:

```text
refs/heads/main
      │
      ▼
  commit C2
      │
      ├── root tree T2
      │     ├── blob B1  hello.txt
      │     └── blob B2  bye.txt
      │
      └── parent commit C1
            │
            └── root tree T1
                  └── blob B1  hello.txt
```

### Walk the Objects by Hand

This lab builds a tiny repository and inspects the objects Git creates.

#### 1. Create a Repository

```bash
mkdir toy
cd toy
git init
```

Example output:

```text
Initialized empty Git repository in .../toy/.git/
```

At this point, Git has created a `.git/` directory, but there are no commits yet.

#### 2. Create a File

```bash
echo "hello world" > hello.txt
git status
```

Example output:

```text
Untracked files:
  hello.txt
```

The file exists in your working directory, but Git has not stored it as part of a snapshot yet.

#### 3. Stage the File

```bash
git add hello.txt
```

Staging does two important things:

1. Git writes a blob object for the file contents.
2. Git records the file path, mode, and blob hash in the index.

Now inspect the index:

```bash
git ls-files -s
```

Example output:

```text
100644 <blob-hash> 0	hello.txt
```

Meaning:

```text
100644       file mode
<blob-hash>  blob object ID
0            stage number
hello.txt    path
```

Stage `0` means the normal, resolved version of the file. Other stage numbers appear during merge conflicts.

#### 4. Inspect the Blob

Use `git cat-file` to inspect the object.

```bash
git cat-file -t <blob-hash>
```

Output:

```text
blob
```

Print the blob contents:

```bash
git cat-file -p <blob-hash>
```

Output:

```text
hello world
```

Notice that the blob contains only the file content. It does not contain the name `hello.txt`.

#### 5. Write a Tree from the Index

The index can be turned into a tree object.

```bash
git write-tree
```

Example output:

```text
<tree-hash>
```

Inspect the tree:

```bash
git cat-file -p <tree-hash>
```

Example output:

```text
100644 blob <blob-hash>	hello.txt
```

Now the name `hello.txt` appears. That name is stored in the tree, not in the blob.

#### 6. Create a Commit Manually

Use `git commit-tree` to create a commit object that points to the tree.

```bash
git commit-tree <tree-hash> -m "initial snapshot"
```

Example output:

```text
<commit-hash>
```

At this point, the commit object exists, but no branch necessarily points to it yet. A commit without a ref can become hard to find later.

Move `main` to the commit:

```bash
git update-ref refs/heads/main <commit-hash>
```

Make `HEAD` point to `main`:

```bash
echo "ref: refs/heads/main" > .git/HEAD
```

Now inspect the commit:

```bash
git cat-file -p <commit-hash>
```

Example output:

```text
tree <tree-hash>
author You <you@example.com> 1699999999 +0000
committer You <you@example.com> 1699999999 +0000

initial snapshot
```

The first commit has no parent.

#### 7. Continue with Normal Git Commands

Create another file:

```bash
echo "bye" > bye.txt
git add bye.txt
git commit -m "add bye"
```

Example output:

```text
[main 9f3a1c2] add bye
 1 file changed, 1 insertion(+)
 create mode 100644 bye.txt
```

Now inspect the tree for `HEAD`:

```bash
git ls-tree -r HEAD
```

Example output:

```text
100644 blob <hash1>	bye.txt
100644 blob <hash2>	hello.txt
```

You can also inspect the root tree directly:

```bash
git cat-file -p HEAD^{tree}
```

Example output:

```text
100644 blob <hash1>	bye.txt
100644 blob <hash2>	hello.txt
```

#### 8. See How Commits Link Together

```bash
git log --oneline --graph --decorate
```

Example output:

```text
* 9f3a1c2 (HEAD -> main) add bye
* a1b2c3d initial snapshot
```

The second commit points to the first commit as its parent.

Diagram:

```text
refs/heads/main ──► commit C2
                     │
                     ├── tree T2
                     │    ├── blob B1  bye.txt
                     │    └── blob B2  hello.txt
                     │
                     └── parent commit C1
                          │
                          └── tree T1
                               └── blob B2  hello.txt
```

#### 9. Inspect Pointers on Disk

Look at `HEAD`:

```bash
cat .git/HEAD
```

Output:

```text
ref: refs/heads/main
```

Look at the branch ref:

```bash
cat .git/refs/heads/main
```

Example output:

```text
9f3a1c2...
```

This file contains the commit ID that `main` currently points to.

In some repositories, refs may be packed into `.git/packed-refs`, so you may not always see every ref as a loose file under `.git/refs/`.

### The Index / Staging Area

The index is Git’s staging area. It sits between the working directory and the next commit.

A useful model:

```text
working directory  →  index  →  commit
     files            staged     snapshot
```

The index stores a compact table of paths and blob IDs, plus file mode and metadata. It does not store file contents directly. The contents are stored as blob objects.

Inspect it with:

```bash
git ls-files --stage
```

Example output:

```text
100644 <hash1> 0	bye.txt
100644 <hash2> 0	hello.txt
```

The index is why Git can quickly answer questions such as:

```bash
git status
git diff
git diff --staged
git commit
```

Git compares:

```text
working directory vs index       → unstaged changes
index vs HEAD                    → staged changes
HEAD vs another commit/tree       → committed differences
```

During a merge conflict, the index can store multiple versions of the same path:

```text
stage 1 = common ancestor
stage 2 = ours
stage 3 = theirs
stage 0 = resolved version
```

That is how Git keeps track of conflict information before you resolve it.

### Loose Objects and Packfiles

New objects usually begin as loose objects.

A loose object is stored under `.git/objects/` using the first two hex characters of its object ID as a directory name.

Example:

```text
.git/objects/ab/cdef1234...
```

Here:

```text
ab        first two hex characters
cdef...   rest of the object ID
```

Loose objects are individually compressed. This is simple and fast for creating new objects.

Over time, Git may pack many loose objects into a packfile. A packfile stores many objects together and can use delta compression to reduce space.

Packfiles live here:

```text
.git/objects/pack/
```

Example:

```text
pack-1234abcd.pack
pack-1234abcd.idx
```

The `.pack` file stores the objects. The `.idx` file lets Git quickly find objects inside the pack.

Commands:

```bash
find .git/objects -type f | wc -l
git gc
ls .git/objects/pack
```

Example progression:

```text
Loose objects:

.git/objects/
  ab/cdef...
  12/3456...

Packed objects:

.git/objects/pack/
  pack-xxxx.pack
  pack-xxxx.idx
```

`git gc` means garbage collection. It cleans up and optimizes the repository by packing objects, pruning unreachable objects when safe, and improving storage efficiency.

### Names Are in Trees, Not Blobs

A blob only stores bytes. File names live in tree objects.

This explains Git’s deduplication behavior.

If two files have identical contents, they use the same blob:

```bash
cp hello.txt copy.txt
git add copy.txt
git ls-files -s
```

Example output:

```text
100644 <same-blob-hash> 0	copy.txt
100644 <same-blob-hash> 0	hello.txt
```

After committing, the tree has two names pointing to the same blob.

```text
tree
 ├── hello.txt → blob B1
 └── copy.txt  → blob B1
```

This is also why a rename does not rewrite the file contents. Git can represent the new name in a new tree while reusing the same blob.

Important detail: Git does not store “rename objects.” Rename detection is usually computed later by commands like `git diff` or `git log --follow`, based on similarity between deleted and added paths.

### Refs, Branches, and HEAD

A ref is a name that points to an object ID.

Common refs:

```text
refs/heads/main
refs/heads/feature/login
refs/tags/v1.0
refs/remotes/origin/main
```

A branch is simply a movable ref that usually points to a commit.

Example:

```text
refs/heads/main → commit C3
```

When you create a new commit on `main`, Git writes the commit object and then moves `refs/heads/main` to the new commit.

```text
before:

main → C2

after commit:

main → C3 → C2
```

`HEAD` usually points to the current branch:

```text
HEAD → refs/heads/main → commit C3
```

If you check out a specific commit instead of a branch, Git enters a detached HEAD state:

```text
HEAD → commit C2
```

Detached HEAD is not dangerous by itself, but new commits made there are not attached to a branch unless you create one.

### Reflog: Git’s Safety Net

The reflog records where refs used to point. It is local to your repository and is extremely useful for recovery.

For example, when a branch moves due to commit, reset, rebase, or merge, Git records the previous position.

Inspect the reflog:

```bash
git reflog
```

Example output:

```text
9f3a1c2 HEAD@{0}: commit: add bye
a1b2c3d HEAD@{1}: commit: initial snapshot
```

You can use reflog entries to recover lost commits:

```bash
git checkout -b recovered HEAD@{1}
```

or:

```bash
git reset --hard HEAD@{1}
```

Use `reset --hard` carefully because it changes the working directory and index.

The reflog is local. It is not normally pushed to remotes.

### Tags

Tags give names to important points in history, often releases.

There are two main tag types.

## Lightweight Tag

A lightweight tag is just a ref pointing directly to a commit.

```bash
git tag v1.0
```

Conceptually:

```text
refs/tags/v1.0 → commit C3
```

#### Annotated Tag

An annotated tag creates a tag object with metadata and a message.

```bash
git tag -a v1.0 -m "first release"
```

Inspect it:

```bash
git cat-file -p refs/tags/v1.0
```

Example output:

```text
object <commit-hash>
type commit
tag v1.0
tagger You <you@example.com> ...

first release
```

Annotated tags are generally better for releases because they preserve tagger information, can be signed, and have their own object ID.

#### Quick Commands

Use these commands when you want to inspect Git directly.

These commands are useful when you want to look under Git’s normal user-facing commands and inspect the actual objects Git stores internally. Git stores repository data as objects, mainly commits, trees, blobs, and tags. These commands help you identify object types, inspect commit metadata, map filenames to blob hashes, compare stored snapshots, and check how much object storage the repository is using.

#### Inspect Any Object

```bash
git cat-file -t <id>   # object type
git cat-file -s <id>   # object size
git cat-file -p <id>   # pretty-print object
```

The `git cat-file` command lets you inspect Git objects directly by object ID, branch name, tag name, or other revision expressions. It is useful when you already have a hash and want to know what it represents.

Example output:

```text
$ git cat-file -t HEAD
commit

$ git cat-file -s HEAD
245

$ git cat-file -p HEAD
tree 7b3f9a1c5d3e8f6a2b0c9d4e1f8a6b7c9d0e1f2a
parent 2a4c6e8f1b3d5a7c9e0f2a4b6d8e1c3f5a7b9d0
author Alex Example <alex@example.com> 1777808400 +0200
committer Alex Example <alex@example.com> 1777808400 +0200

Add hello.txt
```

Explanation:

* `git cat-file -t <id>` shows the object type, such as `commit`, `tree`, `blob`, or `tag`.
* `git cat-file -s <id>` shows the object size in bytes.
* `git cat-file -p <id>` pretty-prints the object in a readable form.
* `<id>` can be a full hash, short hash, branch name, tag name, or expression like `HEAD`.

#### Inspect a Commit

```bash
git show --no-patch --pretty=raw HEAD
```

This shows the commit’s tree, parent commits, author, committer, and message.

This command displays the raw metadata for the commit pointed to by `HEAD`, without showing the file diff. It is useful when you want to inspect exactly what commit object Git has stored and which tree snapshot that commit points to.

Example output:

```text
commit 9fceb02d0ae598e95dc970b74767f19372d61af8
tree 7b3f9a1c5d3e8f6a2b0c9d4e1f8a6b7c9d0e1f2a
parent 2a4c6e8f1b3d5a7c9e0f2a4b6d8e1c3f5a7b9d0
author Alex Example <alex@example.com> 1777808400 +0200
committer Alex Example <alex@example.com> 1777808400 +0200

    Add hello.txt
```

Explanation:

* `commit` is the hash of the commit object itself.
* `tree` is the root tree object for the project snapshot.
* `parent` points to the previous commit.
* `author` records who originally wrote the change.
* `committer` records who created or applied the commit.
* `--no-patch` hides the diff so only commit metadata is shown.
* `--pretty=raw` shows Git’s raw commit fields with minimal formatting.

#### Inspect a Tree

```bash
git ls-tree HEAD
git ls-tree -r HEAD
git ls-tree -r --long HEAD
```

Useful for mapping file paths to blob hashes.

A tree object represents a directory snapshot. It stores filenames, file modes, object types, and object IDs. The `git ls-tree` command shows the contents of a tree, commit, branch, or tag without checking anything out into the working directory.

Example output:

```text
$ git ls-tree HEAD
100644 blob e965047ad7c57865823c7d992b1d046ea66edf78    hello.txt
040000 tree 3b18e512dba79e4c8300dd08aeb37f8e728b8dad    src

$ git ls-tree -r HEAD
100644 blob e965047ad7c57865823c7d992b1d046ea66edf78    hello.txt
100644 blob a3f5c7d9e1b2a4c6e8f0d3b5a7c9e1f2d4b6a8c0    src/main.py

$ git ls-tree -r --long HEAD
100644 blob e965047ad7c57865823c7d992b1d046ea66edf78       14    hello.txt
100644 blob a3f5c7d9e1b2a4c6e8f0d3b5a7c9e1f2d4b6a8c0      128    src/main.py
```

Explanation:

* `git ls-tree HEAD` shows the top-level files and directories in `HEAD`.
* `git ls-tree -r HEAD` recursively lists files inside subdirectories.
* `git ls-tree -r --long HEAD` also shows blob sizes.
* `100644` means a normal file.
* `040000` means a directory tree.
* `blob` means file content.
* `tree` means directory content.
* The hash beside each file is the object ID Git stores for that path.

#### Compare Two Trees

```bash
git diff --name-status <commitA>^{tree} <commitB>^{tree}
```

This compares snapshots without needing to check them out.

This command compares the tree snapshots from two commits. It focuses on stored project content rather than the working directory. It is useful when you want to compare two repository states directly at the object level.

Example output:

```text
M       README.md
A       hello.txt
D       old-config.yml
R100    app.py   src/app.py
```

Explanation:

* `M` means the file was modified.
* `A` means the file was added.
* `D` means the file was deleted.
* `R100` means the file was renamed with 100% similarity.
* `<commitA>^{tree}` resolves commit A to its tree object.
* `<commitB>^{tree}` resolves commit B to its tree object.
* This compares committed snapshots, not uncommitted working directory changes.

#### Find Which Object a Path Uses

```bash
git rev-parse HEAD:hello.txt
```

This prints the blob ID for `hello.txt` at `HEAD`.

This command asks Git to resolve a specific path inside a specific commit. It is useful when you want to know exactly which blob object stores the file content for a path at a particular commit.

Example output:

```text
e965047ad7c57865823c7d992b1d046ea66edf78
```

Explanation:

* `HEAD:hello.txt` means the file `hello.txt` as stored in `HEAD`.
* The output is the blob object ID for that file version.
* If the file content changes, the blob ID changes.
* If the file does not exist at that commit, Git returns an error.
* This is useful for connecting a file path to the object stored in `.git/objects` or packfiles.

#### Read a File from a Commit

```bash
git show HEAD:hello.txt
```

This prints the version of `hello.txt` stored in `HEAD`.

This command reads a file directly from a commit without checking out that commit. It is useful when you want to inspect a historical version of a file, compare content mentally, or recover a file’s contents from another revision.

Example output:

```text
Hello, Git!
```

Explanation:

* `HEAD:hello.txt` selects `hello.txt` from the `HEAD` commit.
* The file content is printed directly to the terminal.
* It does not modify the working directory.
* You can replace `HEAD` with another commit, branch, or tag.
* Example: `git show main:hello.txt` reads `hello.txt` from the `main` branch.

##### Show Object Storage Statistics

```bash
git count-objects -v
```

This shows loose object count, pack count, and storage size.

This command displays statistics about Git’s object database. It helps you understand how many loose objects exist, how many packfiles are present, and how much disk space Git objects are using.

Example output:

```text
count: 24
size: 96
in-pack: 1520
packs: 2
size-pack: 384
prune-packable: 0
garbage: 0
size-garbage: 0
```

Explanation:

* `count` is the number of loose objects.
* `size` is the disk space used by loose objects, usually in KiB.
* `in-pack` is the number of objects stored inside packfiles.
* `packs` is the number of packfiles.
* `size-pack` is the disk space used by packfiles.
* `prune-packable` shows loose objects that also exist in packs and may be removable.
* `garbage` shows invalid or leftover object files.
* This is useful before or after cleanup commands such as `git gc`.

### A More Complete Mental Model

Git is made of three main layers:

```text
1. Object database
   blobs, trees, commits, tags

2. References
   branches, tags, remote-tracking refs, HEAD

3. Working state
   working directory, index, current checkout
```

The working directory is what you edit.

The index is what you plan to commit.

The commit is the saved snapshot.

```text
working directory
      │ git add
      ▼
index
      │ git commit
      ▼
commit
      │ branch ref moves
      ▼
history
```

When you run:

```bash
git add hello.txt
```

Git writes or reuses a blob and updates the index.

When you run:

```bash
git commit
```

Git writes a tree from the index, writes a commit pointing to that tree, and moves the current branch ref.

When you run:

```bash
git checkout main
```

Git updates `HEAD`, updates the index, and writes files into the working directory to match the commit pointed to by `main`.

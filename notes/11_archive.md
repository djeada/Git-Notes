## Git Archive

`git archive` is a command for creating compressed archives of a repository’s tracked files at a specific point in time. It’s a quick way to package your project’s current state without including Git’s version history or untracked files. The result is a clean, self-contained snapshot you can share, back up, or deploy.

Unlike copying a project folder manually, `git archive` ensures the archive contains only what Git tracks—no `.git` directory, no local build artifacts, and no accidental temporary files.

For example, say you’re working on a software project and need to send the latest stable version to a collaborator. You don’t want to send your full Git history or other unrelated files—just the working files from a specific commit. `git archive` does exactly that, packaging them neatly into formats like `tar` or `zip`.

```
+------------------------+                   +------------------------+
|  YOUR GIT REPOSITORY   |                   |      project.tar       |
|                        |   git archive     |                        |
|  .git/    src/         |    ========>      |  src/    README        |
|  README   docs/        |                   |  docs/                 |
|                        |                   |                        |
+------------------------+                   +------------------------+
```

The left box is your Git repository, including `.git` (history/metadata) and working files. The right box is the resulting archive—`.git` is gone, leaving only the project files you want.

### Creating Archives for a Snapshot of the Repository

You can archive files from a specific commit, branch, or tag. The `.git` directory and untracked files are excluded automatically, so the archive stays clean.

Example: create a `zip` archive of the latest commit on the current branch:

```bash
git archive -o archive_name.zip HEAD
```

Here:

* `HEAD` means “current commit of the current branch.”
* You can replace `HEAD` with a branch name, tag, or commit hash to target a different point in history.

By default, `git archive` writes to standard output (stdout). This lets you pipe it directly to another process—like compressing with `gzip` or sending over SSH—without creating a file first:

```bash
git archive branch_name | gzip | ssh user@host "cat > project.tar.gz"
```

This:

1. Creates an archive of `branch_name`.
2. Compresses it with `gzip`.
3. Streams it straight to a remote server.

---

### Exporting Specific Files or Directories

If you only need part of a repository, `git archive` can target individual files or folders.

Example: archive only `config.yml` from the current commit:

```bash
git archive -o config.zip HEAD config.yml
```

Visualization:

```
+------------------------+     git archive     +-----------------------+
|      BRANCH HEAD       |    ============>    |   config.zip          |
| config.yml             |                     | config.yml            |
| src/                   |                     |                       |
+------------------------+                     +-----------------------+
```

The resulting archive contains only `config.yml` as it exists in that commit. The same works for directories—just provide the path.

---

### Customizing the Archive with Prefixes and Filters

You can add a path prefix to every file in the archive. This is useful for avoiding filename collisions or preparing files for deployment into a subfolder.

```bash
git archive --prefix=project-name/ -o project.zip HEAD
```

Visualization:

```
+------------------------+                   +------------------------------+
|  YOUR GIT REPOSITORY   |                   |        project.zip           |
|                        |   git archive     |                              |
|  src/    README        |    ========>      |  project-name/src/           |
|  docs/                 |                   |  project-name/README         |
|                        |                   |                              |
+------------------------+                   +------------------------------+
```

You can also filter which files are included using patterns. For example, to archive only Python files from a specific folder:

```bash
git archive -o source_code.zip HEAD -- path/to/code/*.py
```

Visualization:

```
+------------------------+     git archive     +-----------------------+
|      BRANCH HEAD       |    ============>    | python_files.zip      |
| src/   file1.py        |                     | src/   file1.py       |
| lib/   helper.py       |                     | lib/   helper.py      |
| docs/  guide.txt       |                     |                       |
+------------------------+                     +-----------------------+
```

### Supported Formats for Archiving

By default, `git archive` produces `tar` files, but you can request other formats:

| Format   | Description             | Example Output   |
| -------- | ----------------------- | ---------------- |
| `tar`    | Uncompressed tarball    | `project.tar`    |
| `tar.gz` | Gzip-compressed tarball | `project.tar.gz` |
| `zip`    | Zip-compressed archive  | `project.zip`    |

The format can be set explicitly with `--format` or inferred from the file extension in `-o`.

### Example Workflow: Deployment

You want to deploy the latest `main` branch to a server:

I. Create a tar archive of `main`:

```bash
git archive --format=tar main > project.tar
```

II. Compress it:

```bash
gzip project.tar
```

III. Transfer it to the server:

```bash
scp project.tar.gz user@server:/deployments
```

Result: the server gets a compact package with only the files it needs, ready to extract and use.

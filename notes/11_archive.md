# Git Archive

Git archive is a handy tool for creating compressed archives of a repository’s content. It’s designed to generate snapshots of your project at a specific state, which can then be shared, backed up, or used in deployment scenarios. Unlike simply copying files, this command ensures that only the tracked contents of your repository are included, leaving behind metadata like the `.git` folder.

For instance, imagine working on a software project and needing to deliver the latest stable version to a collaborator without sharing your entire repository history. Git archive allows you to extract just the files you need, neatly packaged in an archive format such as `tar` or `zip`.

Here's a visual representation of how it works:

```
+------------------------+                   +------------------------+
|  YOUR GIT REPOSITORY   |                   |      project.tar       |
|                        |   git archive     |                        |
|  .git/    src/         |    ========>      |  src/    README        |
|  README   docs/        |                   |  docs/                 |
|                        |                   |                        |
+------------------------+                   +------------------------+
```

In this diagram, the left box represents your Git repository, including the `.git` directory (which contains your version history and metadata) and all your project files. The right box shows the resulting archive after running `git archive`. Notice how the `.git` directory is excluded, and only the working files are included in the archive.

### Creating Archives for a Snapshot of the Repository

When using Git archive, you can extract files from a specific commit, branch, or tag. It’s worth noting that the command excludes the `.git` directory and any untracked files, ensuring the archive is clean and concise. For example, if you want to create a `zip` file of the current branch’s latest commit, you could run:

```bash
git archive -o archive_name.zip HEAD
```

Here, `HEAD` refers to the most recent commit on the current branch. You could replace `HEAD` with a branch name, tag, or commit hash to target a specific state.

By default, Git archive outputs the archive content to `stdout`, meaning it prints to the terminal or can be piped into another process. For instance, you could compress the archive with `gzip` or send it over SSH:

```bash
git archive branch_name | gzip | ssh user@host "cat > project.tar.gz"
```

This command generates an archive of `branch_name`, compresses it with `gzip`, and sends it directly to a remote machine.

### Exporting Specific Files or Directories

There may be times when you don’t need the entire repository but just a specific file or folder. Git archive makes this easy. If you need a file named `config.yml` from the current branch, you can isolate it with:

```bash
git archive -o config.zip HEAD config.yml
```

The visual flow:

```
+------------------------+     git archive     +-----------------------+
|      BRANCH HEAD       |    ============>    |   config.zip          |
| config.yml             |                     | config.yml            |
| src/                   |                     |                       |
+------------------------+                     +-----------------------+
```

The resulting `zip` file will contain only `config.yml` as it exists in the latest commit. Similarly, directories can be targeted by specifying their path in the command.

### Customizing the Archive with Prefixes and Filters

Sometimes, it’s useful to include a prefix for all the paths in the archive. This is particularly helpful when deploying files to a directory structure or when avoiding filename collisions. Adding a prefix can be done with the `--prefix` option:

```bash
git archive --prefix=project-name/ -o project.zip HEAD
```

Here's a visual:

```
+------------------------+                   +------------------------------+
|  YOUR GIT REPOSITORY   |                   |        project.zip           |
|                        |   git archive     |                              |
|  src/    README        |    ========>      |  project-name/src/           |
|  docs/                 |                   |  project-name/README         |
|                        |                   |                              |
+------------------------+                   +------------------------------+
```

In this example, every file in the archive will appear under a `my_project/` directory.


To fine-tune the archive contents, you can filter which files or directories are included. Using glob patterns, you can include files with specific extensions or in specific folders:

```bash
git archive -o source_code.zip HEAD -- path/to/code/*.py
```

The visual flow:

```
+------------------------+     git archive     +-----------------------+
|      BRANCH HEAD       |    ============>    | python_files.zip      |
| src/   file1.py        |                     | src/   file1.py       |
| lib/   helper.py       |                     | lib/   helper.py      |
| docs/  guide.txt       |                     |                       |
+------------------------+                     +-----------------------+
```

This creates an archive containing only Python files from the specified directory.

### Supported Formats for Archiving

Git archive supports various output formats to suit different needs. By default, it generates `tar` files, but you can specify formats such as `zip` or `tar.gz` by using the `--format` option. The table below summarizes the available formats:

| Format       | Description                                   | Example Output       |
|--------------|-----------------------------------------------|----------------------|
| `tar`        | Uncompressed tarball                         | `project.tar`        |
| `tar.gz`     | Compressed tarball using gzip                | `project.tar.gz`     |
| `zip`        | Zip-compressed archive                       | `project.zip`        |

The file type can also be determined automatically based on the file extension you specify with the `-o` flag.

### Example Workflow: Deployment

Imagine you’ve been tasked with deploying the latest version of your project to a server. You want to package the files from the `main` branch and transfer them. Using Git archive, you can streamline this process:

I. Create a tar archive of the `main` branch:

```bash
git archive --format=tar main > project.tar
```

II. Compress it to save space:

```bash
gzip project.tar
```

III. Transfer it to the server:

```bash
scp project.tar.gz user@server:/deployments
```

This ensures that the server receives only the files it needs, neatly compressed and ready for extraction.

### Tips and Considerations

- To ensure that unwanted files or folders are not included in the archive, a `.gitattributes` file can be used effectively. Files marked with `export-ignore` in this file will be omitted from the archive. For instance, you can exclude directories like `docs/` or `tests/` by adding specific entries.
- Git archive is an excellent choice for automating deployments or backups since it integrates seamlessly with scripts. This makes it easy to include archive creation as part of your regular workflows or CI/CD pipelines.
- The archives created using Git are based on widely accepted standards, ensuring compatibility with common tools. This allows users to extract these archives using tools like `tar` or `unzip` without requiring any special configurations.

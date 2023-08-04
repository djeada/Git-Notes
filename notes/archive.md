# Git Archive

Git archive is a powerful command that allows you to create compressed archives of your Git repository. These archives can be in one of several formats, including tar, gzip, and zip. Git archive is particularly useful when you want to create a snapshot of your repository at a specific point in time.

## Creating an archive of a Git repository

The `git archive` tool allows you to export a copy of your repository's code from a specific commit, tag, or branch. It's important to note that the `.git` directory will not be included in the archive.

To create a zip archive of the most recent commit in the current branch, use the following command:

```bash
git archive -o archive_name.zip HEAD
```

The file name (which can end in .tar, .tgz, .tar.gz, or .zip) determines the file type of the archive.

By default, `git archive` outputs to `stdout`, which allows you to easily pipe the output to another command or destination. For example, you can create a bzip2-compressed archive of the `branch_name` branch and send it over SSH to the specified destination using the following command:

```bash
git archive branch_name | bzip2 | ssh user_name@192.168.2.0 "cat > archive_name.bz"
```

## Exporting a single file or directory

You can also use `git archive` to export a single file or directory from a Git repository. To export a single file named `file.txt` from the current branch, use the following command:

```bash
git archive -o file.zip HEAD file.txt
```

This command will create a zip archive containing only the `file.txt` file from the most recent commit in the current branch.

## Including or excluding specific files or directories

When creating an archive, you can include or exclude specific files or directories using command-line options. To include only files with a `.py` extension in the archive, use the following command:

```bash
git archive -o archive_name.zip HEAD --prefix=my_project/ --format=zip -- path/to/directory --include=*.py
```

This command will create a zip archive containing only the Python files in the specified directory, with the prefix `my_project/` added to their path.

## Creating an archive of a Git repository

The `git archive` tool allows you to export a copy of your repository's code from a specific commit, tag, or branch. It is important to note that the `.git` directory will not be included in the archive.

To create a zip archive of the most recent commit in the current branch, use the following command:

```bash
git archive -o archive_name.zip HEAD
```

The file name (which can end in .tar, .tgz, .tar.gz, or .zip) determines the file type of the archive.

By default, git archive outputs to `stdout`, which allows you to easily pipe the output to another command or destination:

```bash
git archive branch_name | bzip2 | ssh user_name@192.168.2.0 "cat > archive_name.bz"
```

This command creates a bzip2-compressed archive of the `branch_name` branch and sends it over SSH to the specified destination.

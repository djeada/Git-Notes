To export your repository, use git archive. This tool may be used to extract a copy of code from an existing commit, tag, or branch. It should be noted that the.git directory will not be included.

Make a zip archive of the most recent commit in the current branch:

```bash
git archive -o archive_name.zip HEAD
```

The file name, which might be end with .tar, .tgz, .tar.gz or.zip, determines the file type.

Git outputs to stdout by default, making it easy to pipe it further:

```bash
git archive branch_name | bzip2 | ssh user_name@192.168.2.0 "cat > archive_name.bz"
```

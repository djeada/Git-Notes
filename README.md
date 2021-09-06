# git

TODO:
* merge tool
* change commit message
* squash last 5 commits
* remove commit

<h1> How to merge to master </h1>

```bash
git push origin feature_branch
git checkout master
git pull origin master
git merge feature_branch
git push origin master
```

<h1> How to see what branches are merged/not merged? </h1>

```bash
git checkout master 
git branch --merged
git branch --no-merged
```

<h1>How can I edit the commit message after a git push?</h1>

Last commit:

```bash
git commit --amend -m "new message"
git push --force
```

Old commit:

```bash
git rebase -i HEAD~N
# N is the commit number, beginning with the most recent.
git commit --amend
git rebase --continue
git push --force
```

<h1>Fetch a file from another branch without changing the current branch.</h1>

```bash
git checkout other_branch -- path/to/file 
```

<h1> How to remove branch feature?</h1>

```bash
git brach -d feature
git push origin --delete feature
```

<h1> Own git server </h1>

```bash
sudo apt install git-core
mkdir -p ...path_to_git
mkdir -p ...path_to_git/repo.git
cd ..path_to_git/repo.git
git init --bare
```

<h1>How To Checkout Git Tags</h1>

History of commits.

```bash
git log --oneline --graph

```

Last 10 tags.
```bash
git describe --tags `git rev-list --tags --max-count=10`
```

Checkout
```bash
tag="v2.0"
git checkout $tag
```

<h1>Tag vs Commit</h1>
Both branches and tags refer to a commit. Their behavior, on the other hand, is different. The tag is associated with a single commit, whereas the branch changes from one commit to the next as new code is added.

The most significant distinction is in how they are used. When working on a feature, you use branches to organize code over time. Tagging, in general, refers to metadata linked with a build or deployment that is automatically or manually updated.

<h1>Git stash</h1>
Git stash may be used to take current changes from one branch and move them to another.

To extract unstaged changes, use stash:

```bash
git stash
```

Switch to the chosen branch:

```bash
git checkout another_branch
```

Restore your changes:

```bash
git stash pop
```

<h1>Git archive</h1>
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

<h1>Show word changes in diff</h1>

```bash
git diff --word-diff 
```

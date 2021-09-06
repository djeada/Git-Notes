# git

TODO:
* merge tool
* change commit message
* squash last 5 commits
* remove commit
* change message of a commit

<h1> How to merge to master </h1>

```bash
git push origin feature_branch
git checkout master
git pull origin master
git merge feature_branch
git push origin master
```

<h1> How to change commit message after push</h1>

```bash
git commit --amend -m "new message"
git push --force
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

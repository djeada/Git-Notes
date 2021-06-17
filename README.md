# git

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

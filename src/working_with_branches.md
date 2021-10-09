<h1>Showing branches</h1>

List all local branches:

```bash
git branch
```

List all branches, local and remote:

```bash
git branch -av
```

<h1>Switching between branches</h1>

Switch to a branch_name and update working directory:

```bash
git checkout branch_name
```

<h1>Creating branches</h1>

Create a new branch called branch_name:

```bash
git checkout -b branch_name
```

<h1>Merging branches</h1>
Merge branch_a into branch_b:

```bash
git checkout branch_b
git merge branch_a
```

<h1>Fetch a file from another branch without changing the current branch.</h1>

```bash
git checkout other_branch -- path/to/file 
```

Add a specific commit to your branch:

```bash
git cherry-pick commit_number
```

<h1>Removing branches</h1>

```bash
git brach -d branch_name
git push origin --delete branch_name
```


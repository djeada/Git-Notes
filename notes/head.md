## What is HEAD?
<code>HEAD</code> is by default a pointer to the current branch.
It does not have to be the most recent commit in any branch; it may also refer to tags or any other commits.
When you checkout a new branch, <code>HEAD</code> is updated to point to the new one.
The current <code>HEAD</code> is unique to each local repository and hence unique to each developer. 
 
You can see what <code>HEAD</code> is pointing using:

```bash
cat .git/HEAD
```

Normally, you'll get anything along the lines of: 

```
ref: refs/heads/master
```

You could receive something like this: 

```
b4d373k8990g2b5de30a37bn843b2f51fks2b40
```

<code>HEAD</code> in the first case refers to the master branch. In the second example, it refers to a commit that is not the last commit in any branch. The second case is referred to as the detached head state. 

## Detached HEAD

When <code>HEAD</code> points to a commit that is not the latest commit in any branch, this is referred to as a "detached <code>HEAD</code>."
If you commit in this situation, no branch will be updated.
It doesn't matter where that commit is. 

Operations that can cause <code>HEAD</code> to get "detached <code>HEAD</code>": 

1. Checking out a specific commit:

```bash
git checkout b4d373k8990g2b5de30a37bn843b2f51fks2b40
```

2. Explicitly checking out a remote branch:

```bash 
git checkout origin/master
```

3. Checking out a branch using the detached flag:

```bash
git switch master --detached
```

4. Checking out a tag:

```bash
git checkout v1.0.0
```

## HEAD is used to refer to N most recent commits in a branch. 

We may use <code>HEAD</code> to refer to the most recent commit, and <code>HEAD</code>~</code> to refer to the commit before to the tip, and <code>HEAD</code>~2</code> to refer to the commit even earlier, and so on. 

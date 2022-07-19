## What is HEAD?
HEAD is by default a pointer to the current branch.
It does not have to be the most recent commit in any branch; it may also refer to tags or any other commits.
When you checkout a new branch, HEAD is updated to point to the new one.
The current HEAD is unique to each local repository and hence unique to each developer. 
 
You can see what HEAD is pointing using:

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

HEAD in the first case refers to the master branch. In the second example, it refers to a commit that is not the last commit in any branch. The second case is referred to as the detached head state. 

## Detached head

A "detached HEAD" is when HEAD points to a commit that is not the last commit in any branch.
No branch will be updated if you commit in this state.
It makes no difference where that commit is.

Operations that can cause HEAD to get "detached HEAD": 

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

We may use HEAD to refer to the most recent commit, and HEAD~ to refer to the commit before to the tip, and HEAD~2 to refer to the commit even earlier, and so on. 

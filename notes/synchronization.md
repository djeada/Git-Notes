TODO: 
- mergetool

## Updating the local repository with updates from the remote repository

Get the latest changes from origin without merging:

```bash
git fetch
```

Fetch the latest changes from origin and merge:

```bash
git pull
```

Fetch the latest changes from origin and rebase: 

```bash
git pull --rebase
```

## Differences between fetch and pull

Fetch will not change your files, but pull may.
You may update your remote-tracking branches in refs/remotes/remote at any time by running git fetch (i.e. it will just bring the local copy of the remote repository up to date).
This operation is safe to do since it never affects any of your own local branches under refs/heads.


Pull is a very dangerous tool.
In principle, it's simply a fetch followed by a merge.
It will function properly if the same files are not changed both locally and remotely.
However, you may make a big mess by just pulling in new modifications on top of existing changes in the same file. 

## What to do after fetch?

Fetch retrieves the updates from the remote without modifying your files. It is your responsibility to integrate the updates into your files.

One method is to simply use <code>merge</code> command after fetching:

```bash
git fetch
git merge
```
  
Another method is to discard any uncommitted local modifications and replace your files with the remote version of the current branch: 

```bash
git fetch
git reset --hard origin/branch_name 
```

Make sure to first commit any work-in-progress! 

## Merging vs rebasing

* Merge creates a merge commit with all changes (easy & safe to use).
* Rebase moves all commits on the tip of the other branch (good to keep git history linear & clean)

## Merge conflicts

Merge conflicts arise when two changes are present in the same location in the same file. Then you have to solve the conflict manually.
Furthermore, you may consider to always merge commits manually, since even though two edits are physically separated in the file, they might still be conceptually at odds.

If merge conflicts occur, read the super-helpful terminal advice and continue as follows: 

1. <code>git diff</code> to resolve the merge conflicts
2. <code>git rebase --continue</code>
3. repeat untill all issues are resolved 

## Sending changes from the local to the remote

Push local changes to the origin:

```bash
git push origin branch_name
```
  
It is important to note that it will only function if there are no new commits on the remote branch.
  
## What if the remote was updated by someone else in the meantime?
  
One common scenario is that while one developer was gathering his commits and deciding whether or not to push them to the remote, another developer pushed his updates in the meanwhile.
What will happen next? Since pushing never results in merging, the developer will be informed that his commits were rejected by the remote after pushing them.
The developer must now pull, handle any merge issues locally, and push to the remote again. 

## How to merge with master?
Suppose you spent some time working on a new branch named <code>feature_branch</code> before deciding to merge your updates with the master.
You may do it by following these steps: 

```bash
git push origin feature_branch
git checkout master
git pull origin master
git merge feature_branch
git push origin master
```

## How to find out which branches have been merged and which have not been merged?

To see which branches have previously been merged with master, use the <code>checkout</code> command: 

```bash
git checkout master 
git branch --merged
git branch --no-merged
```

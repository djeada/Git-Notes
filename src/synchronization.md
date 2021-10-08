TODO: 
- mergetool

<h1>Updating the local repository with updates from the remote repository</h1>

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

<h1>Differences between fetch and pull</h1>

Fetch will not change your files, but pull may.
You may update your remote-tracking branches in refs/remotes/remote at any time by running git fetch.
This operation is safe to do since it never affects any of your own local branches under refs/heads.


Pull is a very dangerous tool.
In principle, it's simply a fetch followed by a merge.
It will function properly if the same files are not changed both locally and remotely.
However, you may make a big mess by just pulling in new modifications on top of existing changes in the same file. 

<h1>What to do after fetch?</h1>

Fetch retrieves the updates from the remote without modifying your files. It is your responsibility to integrate the updates into your files.

One method is to simply use <i>merge</i> command after fetching:

  ```bash
git fetch
git merge
```
  
Another option is to blow away any uncommitted local changes and use the remote version of your branch to bring your files up to date:

```bash
git fetch
git reset --hard origin/branch_name 
```

Make sure to first commit any work-in-progress! 

<h1>Merge conflicts</h1>

Merge conflicts arise when two changes are present in the same location in the same file. Then you have to solve the conflict manually.
Furthermore, you may consider to always merge commits manually, since even though two edits are physically separated in the file, they might still be conceptually at odds.

If merge conflicts occur, read the super-helpful terminal advice and continue as follows: 

1. git diff to resolve the merge conflicts
2. git rebase --continue
3. repeat untill all issues are resolved 

<h1>How to merge with master?</h1>

```bash
git push origin feature_branch
git checkout master
git pull origin master
git merge feature_branch
git push origin master
```

<h1>How to find out which branches have been merged and which have not been merged?</h1>

The <i>checkout</i> command can be used to make check which branches were merged:

```bash
git checkout master 
git branch --merged
git branch --no-merged
```

<h1>Sending changes from the local to the remote</h1>

Push local changes to the origin:

```bash
git push origin branch_name
```
  
It is important to note that it will only function if there are no new commits on the remote branch.
  
<h1>What if the remote was updated by someone else in the meantime?</h1>
  
Pushing never results in a merging.
The user should pull, resolve any merge issues locally, and then push to the remote. 

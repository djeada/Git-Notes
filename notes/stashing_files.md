## What is stashing?

In Git terminology, "stashing" refers to temporarily saving changes that are not ready to be committed. This allows you to switch branches or make other changes without losing your work.

For example, suppose you are working on a feature in a branch called feature, and you need to switch to the master branch to fix a bug. However, you don't want to commit the changes you've made in the feature branch, because they are not ready to be merged into the master branch yet. In this case, you can use stashing to save your changes and switch branches without losing your work.

```
Before Stashing
----------------
... -> [C3] -> [C4] (HEAD)
          \
           [WIP]


After git stash
---------------
... -> [C3] -> [C4] (HEAD)


After git stash pop/apply
-------------------------
... -> [C3] -> [C4] (HEAD)
          \
           [WIP]
```

- `[C3]` and `[C4]`: These are past commits, with `[C4]` being the most recent commit.
- `(HEAD)`: This is the current pointer, indicating where you are in the commit tree.
- `[WIP]`: This represents your current working directory with changes that haven't been committed.
    
## Stashing changes

To stash your changes, use the git stash command. This will save your changes to a temporary storage area and revert your working directory to the most recent commit.

For example:

```bash
$ git stash
Saved working directory and index state WIP on feature: abcdefg Add new feature
HEAD is now at abcdefg Add new feature
```

This will save your changes and leave your working directory clean, allowing you to switch branches or make other changes without losing your work.

## Stashing with a message

You can also provide a message when stashing your changes, which can be helpful for keeping track of what you were working on. To do this, use the git stash save command followed by the message. For example:

```bash
$ git stash save "Work in progress on new feature"
Saved working directory and index state WIP on feature: abcdefg Work in progress on new feature
HEAD is now at abcdefg Add new feature
```

## Viewing stashed changes

To see a list of all the stashed changes in your repository, you can use the git stash list command. For example:

```bash
$ git stash list
stash@{0}: WIP on feature: abcdefg Work in progress on new feature
```

You can also view the details of a specific stash by specifying its index or reference. For example:

```bash
git stash show stash@{0}
```

This will show the changes that were stashed, as well as the message if one was provided.

## Applying stashed changes

To apply the changes from a stash, use the git stash apply command followed by the stash reference. For example:

```bash
git stash apply stash@{0}
```

This will apply the changes from the stash to your working directory, but the stash will remain in the stash list.

## Dropping stashed changes

If you no longer need a stash, you can drop it using the git stash drop command followed by the stash reference. For example:

```bash
git stash drop stash@{0}
```

This will remove the stash from the stash list.

## Clearing the stash list

To remove all stashes from the stash list, use the git stash clear command. For example:

```bash
git stash clear
```

This will permanently remove all stashes from the stash list and cannot be undone.

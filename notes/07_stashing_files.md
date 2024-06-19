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

Sure, here’s a more comprehensive expansion of those sections on Git:

---

## Viewing Stashed Changes

Stashing in Git allows you to save your uncommitted changes temporarily without committing them to your repository. This is useful when you need to switch branches or work on something else without losing your current progress. To manage and view your stashed changes, Git provides several commands.

### Listing All Stashes

To see a list of all the stashed changes in your repository, you can use the `git stash list` command. This command will display a list of all the stashes that you have created in your repository. Each stash entry will include an index, a description of the changes, and a message if one was provided. For example:

```bash
$ git stash list
stash@{0}: WIP on feature: abcdefg Work in progress on new feature
stash@{1}: WIP on bugfix: 1234567 Temporary stash before bugfix
stash@{2}: WIP on refactor: 89abcde Started refactoring codebase
```

In the output above:
- `stash@{0}`, `stash@{1}`, `stash@{2}` are references to specific stashes.
- The description after the reference provides context about the stashed changes.

### Viewing Details of a Specific Stash

If you want to view the details of a specific stash, you can use the `git stash show` command followed by the stash reference. This command will show a summary of the changes included in the stash. For example:

```bash
$ git stash show stash@{0}
```

This will provide an overview of the files that were modified, added, or deleted in the stashed changes. To get a more detailed view, you can use the `-p` option, which will show the actual diff of the changes:

```bash
$ git stash show -p stash@{0}
```

This detailed view includes the exact changes made to each file, similar to the output of `git diff`. This can be particularly useful if you need to review what was stashed before deciding to apply or drop the stash.

### Displaying Stash Statistics

Additionally, you can use the `--stat` option with `git stash show` to see a concise summary of changes including the number of insertions and deletions:

```bash
$ git stash show --stat stash@{0}
```

This provides a quick overview of the scope of the changes without showing the full diff.

## Applying ("Unstashing") Stashed Changes

Once you have reviewed your stashed changes, you may want to apply them back to your working directory. This process is often referred to as "unstashing." Git provides several commands to apply stashed changes in different ways.

### Applying a Stash

To apply the changes from a specific stash, you use the `git stash apply` command followed by the stash reference. This command applies the stashed changes to your working directory but does not remove the stash from the stash list. For example:

```bash
$ git stash apply stash@{0}
```

After running this command, the changes from `stash@{0}` will be applied to your working directory. However, the stash itself will still be available in the stash list. This is useful if you want to apply the same stash multiple times or keep it for future reference.

### Applying the Latest Stash

If you want to apply the most recent stash, you can omit the stash reference:

```bash
$ git stash apply
```

This command will apply the changes from the most recent stash (i.e., `stash@{0}`).

### Resolving Conflicts

When applying a stash, you may encounter conflicts if the changes in the stash conflict with the changes in your working directory. Git will notify you of any conflicts, and you will need to resolve them manually, similar to resolving merge conflicts. Once resolved, you can continue working with the applied changes.

### Removing a Stash After Applying

If you want to apply a stash and remove it from the stash list in one step, you can use the `git stash pop` command. This command applies the stash and then deletes it from the stash list:

```bash
$ git stash pop stash@{0}
```

This is useful when you are sure you no longer need the stash after applying it. The stash is applied to your working directory, and `stash@{0}` is removed from the list.

### Applying Stashes with Index Changes

By default, `git stash apply` only applies changes to the working directory. If your stash includes changes to the index (staged files), you can include those changes by using the `--index` option:

```bash
$ git stash apply --index stash@{0}
```

This ensures that both the working directory changes and the index changes are applied, making your working directory and index exactly as they were when the stash was created.

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

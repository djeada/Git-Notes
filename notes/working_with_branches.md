## What are branches?
Suppose you have a working  version of your program. You want to add a new feature, but it necessitates some modifications to the logic of current parts of the app. There are several options available to you. You might just begin working on your current source code, being extra cautious and hope not to break anything that is already working. You could alternatively create a backup copy of your project directory, but this would waste valuable disk space and require you to remember which files and where exactly were modified by you. Another way is to utilize git branches. The last approach solves the issues of the second one. Branches allow you to keep various versions of your code distinct and clean. And commits maintain track of every modification you make. 

## How do branches work techincally?
Branches are used to abstract the edit and commit procedure. Consider them a method to request a clean working directory, staging area, and project history. New commits are added to the current branch's history. A branch is just a pointer to the most recent commit in a given context. When you create a branch, git just needs to create a new pointer. This operation makes no additional changes to the repository. 

A history is a directed acyclic graph (DAG) of snapshots in Git. This simply implies that each snapshot in Git refers to a collection of "parents," or snapshots that came before it. Because a snapshot may descend from many parents, for example, as a result of joining (merging) two concurrent branches of development, it is a collection of parents rather than a single parent (as would be the case in a linear history).

A commit history can be represented as follows: 

```bash
o <-- o <-- o <-- o 
                  ^
                  |
                  o <-- o
```

The letter o represents a commit. Each commit's parent is indicated with an arrow. After the third commit, the history splits into two distinct branches. This might imply that two distinct characteristics are being created in tandem, independently of one another. In the future, both branches may be combined to generate a new snapshot that includes both features, resulting in a new history that looks like this, with the newly formed merging commit: 

```bash
o <-- o <-- o <-- o <-------------- o
                  ^                 ^
                  |                 |
                  o <-- o <-- o <-- o
```

## When to create branches?

There are various approaches to this subject. We will briefly mention some of them.

<h2>Bug fixes</h2>
A branch may be created when implementing a fix for a bug discovered in the software. You never know how long it will take to figure out a solution to the problem. Create a safe space for experimentation is really useful in this case.

<h2>New features</h2>
When putting a new idea into action, you may want to consider creating a branch. You will not make any modifications to the working version of your application until your changes are considered ready for integration with the rest of the program. If the new idea doesn't work out, you may simply delete the branch and move on. 

<h2>Long-lived branches</h2>

Typical long-lived branches include: 
* master branch for the project's mainline or production code 
* dev branch for integrating various short-lived branches 

<h2>Continous integration</h2>
Short-lived branches are considerably easier to merge with the master than long-lived ones. This phenomenon has given rise to the idea of continuous integration (or trunk based development). Developers using this method merge their branches often, at least once per day, and sometimes several times per day. 

Remember: A few lines of code that can be merged into the master branch immediately is preferable than a whole feature that will be held on a separate branch for months only to be forgotten and never merged. 

## Showing branches

List all local branches:

```bash
git branch
```

List all branches, local and remote:

```bash
git branch -av
```

## Switching between branches

To switch to a branch_name and update working directory, use:

```bash
git checkout branch_name
```

This process will do the following:

* The HEAD pointer has been updated to point to branch_name.
* The files in the working directory have been swapped and now represent the status of the branch_name as of the most recent commit. 

## Creating branches

Create a new branch called branch_name:

```bash
git checkout -b branch_name
```

This operation does not alter the repository's history. You just get a new pointer to the most recent commit. 

## Transferring changes between branches

Fetch a file from another branch without changing the current branch:

```bash
git checkout other_branch -- path/to/file 
```

Add a specific commit to your branch:

```bash
git cherry-pick commit_number
```

## Merging branches

Merge branch_a into branch_b:

```bash
git checkout branch_b
git merge branch_a
```

Branch_a's changes will be incorporated into branch_b, but branch_b will remain unaffected.

## Removing branches

To remove a branch both locally and remotely, use:

```bash
git brach -d branch_name
git push origin --delete branch_name
```

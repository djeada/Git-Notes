## What is Git?

Git is a version control system (VCS) created by Linus Torvalds, the same person who developed the Linux kernel. It’s a tool for tracking changes to files over time, mainly used in software development but useful for any project that involves evolving files.

The simplest way to think about Git is like a time machine for your project—it takes snapshots of your files at different points in time. You can revisit any of those snapshots later, compare them, or restore them if something breaks.

Git also helps teams work together without overwriting each other’s work. Two people can edit different parts of the same file at the same time, and Git can merge those changes into a single, consistent version.

In short, Git:

* **Keeps backups** by storing the history of your project so you can recover from mistakes or lost files.
* **Supports simultaneous work** so multiple people can edit without interfering with each other’s changes.
* **Merges different lines of work** in a controlled way, keeping your history organized.
* **Records detailed history** of changes—who made them, when, and why.

Branches in Git let you work on multiple versions of your project in parallel. For example, you might fix a bug on one branch while developing a new feature on another, switching between them as needed.

### What is a repository?

A Git repository (often called a “repo”) is where Git stores your project files along with all the historical data that makes version control possible. This history lives inside a hidden `.git` folder. Without it, the directory is just a regular folder—Git won’t know it’s a repository.

There are two main kinds of repositories:

* **Local repositories** – Stored on your own machine. You do most of your editing, staging, and committing here.
* **Remote repositories** – Stored on a server or a hosting platform like GitHub or GitLab. These are used for sharing your work with others or backing it up.

Git is **distributed**, meaning every local copy of a repository contains the full project history. Even without internet access, you can browse history, create commits, and roll back changes. You could lose the main server entirely and still restore the project from any local copy.

```
+-----------------+   +-----------------+   +-----------------+
|   Alice's Repo  |   |  Central Repo   |   |   Bob's Repo    |
|                 |   |  (e.g., GitHub) |   |                 |
|  o---o---o---o  |   |  o---o---o---o  |   |  o---o---o---o  |
+--------|--------+   +--------|--------+   +--------|--------+
         |                     |                     |
   Push/Pull           Push/Pull             Push/Pull
         |                     |                     |
+--------v--------+   +--------v--------+   +--------v--------+
| Local Workspace |   | Server Storage  |   | Local Workspace |
|                 |   |                 |   |                 |
|   o---o---o---o |   |   o---o---o---o |   |   o---o---o---o |
+-----------------+   +-----------------+   +-----------------+
```

### Initializing a New Repository

To start tracking a new project with Git, you create a repository with:

```bash
git init project_name
```

This makes a new folder called `project_name` and adds an empty `.git` directory inside it. That `.git` directory holds all the metadata and history.

Example:

```bash
$ cd project_name
$ ls -a
.  ..  .git
```

If the `.git` folder is removed, Git no longer treats the folder as a repository.

### What happens behind the scenes?

When you run `git init`, Git sets up the internal database it uses to track files, changes, and relationships between changes.

```
+-------------------------+
|         Git Repo        |
|                         |
|  +----------+           |
|  | .git dir |           |
|  +----------+           |
|                         |
|  +----------+  +------+ |
|  | Code     |  | Docs | |
|  +----------+  +------+ |
|                         |
|  +----------+           |
|  | Config   |           |
|  +----------+           |
|                         |
|  +----------+           |
|  | Hooks    |           |
|  +----------+           |
|                         |
+-------------------------+
```

Inside `.git`, you’ll find:

* **Blobs** – Store the actual content of files at specific points in time.
* **Trees** – Represent directories, storing references to blobs and other trees.
* **Commits** – Record a snapshot of the project, along with metadata like author, date, and commit message, plus a link to previous commits.
* **Refs** – Pointers to commits (e.g., the latest commit on a branch or a tagged release).

These pieces form a chain of snapshots. Git’s history browsing commands simply walk through these links to show you what changed and when.

### Cloning an Existing Repository

To make a full local copy of a repository—its files, commit history, and configuration—you use the **clone** command. This is the standard way to download a project from a hosting service like GitHub, GitLab, or Bitbucket onto your own machine:

```bash
git clone https://github.com/djeada/git.git
```

When you run this, Git will:

1. Create a new folder named `git` (matching the repository name, unless you specify otherwise).
2. Download the entire commit history and all tracked files from the remote repository.
3. Configure the local copy so it’s linked to the original remote, allowing you to pull updates or push your own changes later.

If you want to clone the repository into a specific directory rather than the default name, just provide the target path as the second argument:

```bash
git clone https://github.com/djeada/git.git /opt/projects
```

This places the full repository inside `/opt/projects`. From there, you can work with it like any other folder—but because it’s a Git repo, you can commit changes locally and sync with the remote repository when needed.

### Checking if You’re in a Git Repository

There are two super simple ways to check.

**I. Run `git status`**

Type:

```bash
git status
```

If you’re inside a Git repo, Git will tell you which branch you’re on, whether it’s connected to a remote, and if there are any changes. It might look like this:

```
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

If you’re *not* in a repo, Git will complain instead:

```
fatal: not a git repository (or any of the parent directories): .git
```

**II. Look for the `.git` folder**

Run:

```bash
ls .git
```

If the folder exists, you’ll see files like `HEAD`, `config`, `objects`, and `refs`. That means the directory is a Git repo.

If the folder isn’t there, then Git hasn’t been initialized in that location.

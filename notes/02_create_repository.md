## What is Git?

Git is a version control system (VCS) created by Linus Torvalds, the same person who started the Linux operating system. It’s designed to help developers track changes to their projects over time. Think of it like a camera for your code: it takes snapshots of each change, so you can always revert to an earlier version if something goes wrong. This snapshot approach also simplifies team collaboration, because multiple people can make separate updates without stepping on each other’s toes. In short, Git:

* **Keeps backups** of your work, guarding against accidental file loss.  
* **Allows concurrent edits** on the same file, reducing messy conflicts when working with a team.  
* **Merges diverging changes** smoothly, helping maintain a clean history of all modifications.  
* **Provides detailed history**, so you know who changed what and when, fostering accountability.

When you use Git, it feels like you’re always a few steps ahead. If a file breaks, it’s easy to look back at an earlier snapshot. If you need to split your work into multiple lines of development—maybe you’re testing a new feature while also fixing bugs—you can switch between these lines (known as “branches”) without confusion.

### What is a repository?

A Git repository, often shortened to “repo,” is the space where your project and all its historical changes are stored. Inside a repo, Git tracks every file’s revisions and stores them in a special `.git` folder. This folder is critical because without it, Git has no way to keep tabs on what’s changed or when.

You can have multiple types of repositories:

- **Local repositories**: These live on your own computer and are where you do your day-to-day work.
- **Remote repositories**: These live on a server or a hosted service like GitHub or GitLab. They’re handy for backups, collaboration, or sharing code with others.

Although many teams rely on a central remote repository (like a GitHub repo), Git is actually **distributed**. That means any local copy of the project contains the full history. You could lose the main server, and you’d still be able to recover everything from any local repo.

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

It might look like a fancy diagram, but the takeaway is: every clone of a Git repository is a complete version on its own. If you have a spotty internet connection or you need to pass code around on a USB drive, it’s no problem.

### Initializing a New Repository

When you’re starting a brand-new project, you can create a Git repository by running:

```bash
git init project_name
```

This creates a folder called `project_name` in your current directory, along with an empty `.git` folder inside `project_name`. If you go into that folder and run `ls -a`, you should see the `.git` directory. Here’s an example of what you might see:

```bash
$ cd project_name
$ ls -a
.  ..  .git
```

That `.git` folder is what turns a plain old directory into a Git repo. If you ever delete it, the directory stops being a repository and is just normal files again.

### What happens behind the scenes?

Under the hood, Git is storing not just your files but also the relationships between different states of those files. When you run `git init`, Git creates a `.git` folder to hold all this information.

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

- **Blobs**: These store file contents. Each version of a file (even if it’s just a small change) is kept as a separate blob.  
- **Trees**: Think of these like directories that track which files (or other directories) are present.  
- **Commits**: Each commit has metadata (author name, date, message), plus pointers to the state of the project (the “tree”) at that moment and links to any parent commits.  
- **Refs**: These are pointers to specific commits—like the tip of a branch (e.g., `main` or `dev`) or tags that mark important snapshots.

All these pieces fit together like a chain of snapshots. When you view the project’s history, Git is basically letting you hop between these snapshots to see exactly what changed at each step.

### Cloning an Existing Repository

If you want to download a complete copy of a repo, you can **clone** it. This is how you typically get code from a remote server like GitHub onto your computer:

```bash
git clone https://github.com/djeada/git.git
```

After running this, Git will:

1. Create a new folder named `git` (unless you specify a different folder name).
2. Grab the entire history of that project from the remote repo.
3. Set up everything so you can start working immediately in your local copy.

If you’d like to place the cloned repository in a specific folder, you can do this:

```bash
git clone https://github.com/djeada/git.git /opt/projects
```

This will put a complete copy of the repository in `/opt/projects`. From there, you can open, modify, and manage the files just like any normal directory—except that it’s fully backed by Git, and you can push your changes back to the original repo or any other remote whenever you want.

### Verifying a Git Repository

Sometimes you might wonder, “Am I actually *in* a Git repository right now?” To check, run:

```bash
git status
```

- If you **are** in a Git repo, Git will tell you something like:

```
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

- If you **are not** in a Git repo, you’ll see an error like:

```
fatal: not a git repository (or any of the parent directories): .git
```

Another way to confirm is to look for the `.git` folder:

```bash
ls .git
```

If you see a bunch of subfolders and files (like `HEAD`, `config`, `objects`, `refs`, etc.), that means you’re in a valid Git repo. If you get a “No such file or directory” message, then there’s no `.git` directory there.

```
$ ls .git
HEAD  config  description  hooks  info  objects  refs
```

This indicates a fully functioning Git repository.

<h1>What does git do?</h1>
Version control systems are software applications that keep track of modifications to source code (or other collections of files and folders).
These technologies help in the maintenance of a change history while also promoting developer collaboration.

In a nutshell, git is useful for: 

* Creating backups of your work.
* Working on multiple versions of the same file at the same time.
* Bringing the changes together into a single cohesive version.
* Keeping track of who and when wrote a certain module, file, function, or even line of code. 

<h1>What is a repository?</h1>
A repository is a special directory that tracks and saves all changes to the files contained within it. This data is kept in a subdirectory called .git. Users can delete, copy, or create repositories for their projects. 

There could be only one local repository. There might be a local repository as well as a remote repository. Finally, there might be multiple local repositories as well as multiple remote repositories.

Git was created to accommodate a distributed approach that does not require a central repository.
Git was also built in such a way that the "client" and "server" do not have to be online at the same time.
It was created so that users with unstable connections may share code through email and even work fully unconnected and share code via CD's.

<h1>Create a new repostiory</h1>

Create a new local repository name project_name in the current directory:

```bash
git init project_name
```

<h1>What happens behind the scenes?</h1>
A repository may appear to be nothing more than a directory dedicated to your source code. That is not the case. So, what happens when we create a repository? 

You might have noticed the mysterious .git directory that was created along with the repository. If you delete it, your project directory will revert to a normal directory with files and subdirectories. This directory contains all git objects. One of those objects is a series of snapshots, trough which git stores the entire history of all files and subdirectories in your project directory. In the snapshots files are referred to as "blobs," while directories are referred to as "trees.". 
Git also stores snapshot metadata, such as who created each snapshot, messages connected with each snapshot, and so on.

<h1>Download an existing repository</h1>

Use <i>clone</i> command followed by an url, for example:

```bash
git clone https://github.com/djeada/git.git
```

If you do not want the repository to be downloaded to the current directory, add the destination path after the url:

```bash
git clone https://github.com/djeada/git.git /opt/projects
```

<h1>How to check if a directory is a git repository?</h1>
Use the following command to see if the current path is within a git repository:

```bash
git status 
```

If it isn't a repository, you should get an error message that says "fatal: Not a git repository." 

You may also try listing the .git directory and checking whether it exists: 

```bash
ls .git
```

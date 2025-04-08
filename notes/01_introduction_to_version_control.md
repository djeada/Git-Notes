## Why Learn Git?

Git is a powerful and widely used version control system that helps you manage code changes, work with others, and keep projects safe. Think of it as a digital timeline you can jump back to whenever something goes wrong. Here are some straightforward reasons to learn Git:

- Most software teams rely on Git. It’s a skill that employers actively look for.
- By pushing to an external server (like GitHub), you’ll have a safe backup of your function.
- Every version of your project is saved. You can always rewind to a point where your code functioned correctly.
- If you break something, you can revert to a stable commit and troubleshoot from there.
- For those using multiple machines or virtual environments, Git keeps everything in sync.
- Git lets multiple people function on the same codebase, handling merge conflicts and version tracking.
- GitHub is a great place to show off your projects to potential employers or collaborators.

```
.______________________________________________________________________.
|                                                                      |
|                       GIT / SOURCE CONTROL FLOW                      |
|______________________________________________________________________|

     ( Commit #1 )        ( Commit #2 )        ( Commit #3 )
         │  │                 │  │                 │  │
         v  │                 v  │                 v  │
   .-----------.          .-----------.          .-----------.
   |           |          |           |          |           |
   |  COMMIT   |  ----->  |  COMMIT   |  ----->  |  COMMIT   |
   |    #1     |          |    #2     |          |    #3     |
   '-----------'          '-----------'          '-----------'
          │                     │                     │
          │    <Branch off>     │                     v
          │                     │
          v                     │
     .-----------.              │
     |           |              │
     | COMMIT #2a|              │
     | (Feature) |   < Merge >  │
     '-----------'              │
          │                     │
          v                     v
     .-----------.          .-----------.
     |           |          |           |
     | MERGED    |          |  LATEST   |
     |  CODE     | <------  |  COMMIT   |
     '-----------'          '-----------'

```

- Each **box** (e.g., `COMMIT #1`) represents a snapshot of the project at a certain point in time.
- Lines and arrows show how commits progress linearly—or branch and merge.
- **Branch off**: You create a new line of development to add features or test ideas.
- **Merge**: You bring changes from a branch back into the main line of development.

The main idea is:

1. **Fork** → get your own copy.  
2. **Clone** → edit locally.  
3. **Commit & Push** → updates to your fork.  
4. **Pull Request** → propose changes back to the original repo.

### Who is Using Git?

Git is used in many fields, from open-source communities to government agencies:

- One of the largest, most important open-source projects is maintained in a Git repository.
- Scientists at NASA rely on Git for code management and collaboration.
- Facebook’s popular JavaScript library is developed using Git, underscoring its role in modern front-end development.
- Even organizations like the NSA have GitHub repositories.
- Projects like the “Dive into Deep Learning” book are kept organized with Git, allowing authors and contributors to update content smoothly.

### Git Configuration

Before doing anything else, tell Git who you are by setting your user name and email. This ensures your commits are properly labeled.

I. Open your terminal (or command prompt).

II. Set your user name:

```bash
git config --global user.name "Your Name"
```

III. Set your email:

```bash
git config --global user.email "you@example.com"
```

IV. You can confirm your configuration by running:

```bash
git config --list
```

Expected output (example):

```
user.name=Your Name
user.email=you@example.com
core.editor=vim
...
```

You’ll see Git settings like `user.name` and `user.email`. If they match what you entered, you’re good to go.

If you need different credentials for a single project, omit `--global` to apply the settings only to that repository.

### Setting Up SSH

Using SSH with Git is more secure and saves you from typing your credentials every time you push or pull. Below is a step-by-step guide:

I. Check if you already have an SSH key:

```bash
ls -al ~/.ssh
```

Expected output (example):

```
- 32 id_rsa
- 32 id_rsa.pub
```

- `id_rsa` is your private key.  
- `id_rsa.pub` is your public key.  

If you see these files, you already have an SSH key pair.

II. Generate a new SSH key (if you don’t have one already):

```bash
ssh-keygen -t rsa -C "you@example.com"
```

Expected prompt (simplified):

```
Enter file in which to save the key (/home/user/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

Press Enter to accept the default file location, and optionally type a passphrase for extra security. When complete, you’ll have `id_rsa` (private) and `id_rsa.pub` (public) in `~/.ssh`.

III. View your public key:

```bash
cat ~/.ssh/id_rsa.pub
```

Expected output (example):

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDNKnZ8... you@example.com
```

This is the text you need to copy and paste into GitHub. Don’t share your private key (`id_rsa`) with anyone.

IV. Add your SSH key to GitHub:

- Log in to GitHub and go to Settings > SSH and GPG keys.
- Click New SSH key, give it a title (e.g., “Laptop Key”), paste the public key into the "Key" field, and Save.

V. Test the SSH connection:

```bash
ssh -T git@github.com
```

Expected output (example):

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

If you see that message, you’ve successfully connected to GitHub via SSH. You’re ready to push, pull, and clone without typing a password each time.

## Fork a Repo

When you fork a repository on GitHub, you create your own copy that you can edit freely. Your changes stay separate from the original (“upstream”) project until you decide to share them via a pull request. This is handy if you want to propose edits or new features without risking the original codebase.

```
.____________________________________________________.
|                                                    |
|                      FORK FLOW                     |
|____________________________________________________|

                    ( GitHub )
.---------------------------------------.
|            Original Repo              |
|---------------------------------------|        You click
|  Hosted under author’s account        |  ---->  "Fork"
|  e.g. github.com/author/repo_link     |
'---------------------------------------'
                    |
                    | (Fork creates a copy under your account)
                    v
.---------------------------------------.
|            Your Forked Repo           |
|---------------------------------------|
|  Hosted under your_username account   |
|  e.g. github.com/your_username/repo   |
'---------------------------------------'
```

- **Original Repo**: The main project on GitHub that you don’t directly control.
- **Fork**: You get a full copy under **your** GitHub account. You can modify it freely without affecting the original.
- Later, if you want your changes to be considered for the original repository, you open a Pull Request.

**How to fork a repository**:

1. Go to the GitHub page of the repository you want to fork.
2. Click the *Fork* button (near the top-right of the page).

You’ll now have your own version (fork) of that repository under your GitHub account.

### Cloning a Forked Repository

To work on your new fork locally, clone it to your machine:

```bash
git clone git@github.com:your_username/repo_link
```

Expected output (example):

```
Cloning into 'repo_link'...
remote: Enumerating objects: 107, done.
remote: Counting objects: 100% (107/107), done.
remote: Compressing objects: 100% (79/79), done.
remote: Total 107 (delta 33), reused 89 (delta 16)
Receiving objects: 100% (107/107), 21.47 KiB | 2.15 MiB/s, done.
Resolving deltas: 100% (33/33), done.
```

Git is copying all the files from your forked repository on GitHub to your local machine. Once it’s done, you’ll have a new folder named `repo_link` (or whatever the repository is called).

### Adding the Upstream Repository

To keep your fork up-to-date with the original repository, you need to add the upstream (the source you forked from) as another remote.

I. Move into your local repository:

```bash
cd repo_link
```

There’s no output unless the folder doesn’t exist. If you see an error, double-check the directory name.

II. Add the upstream remote:

```bash
git remote add upstream git://github.com/author/repo_link
```

Usually, there is no output if successful. If you see an error like `fatal: remote upstream already exists.`, it means the remote name “upstream” is already taken or was added before.

*Why do this?* You now have two remotes:

- *origin* (your fork on GitHub)
- *upstream* (the original repository you forked)

### Staying Up-to-Date with Upstream

When the original repository gets new commits, you can fetch and merge them into your fork so you’re always working with the latest changes.

I. Fetch updates from upstream:

```bash
git fetch upstream
```

Expected output (example):

```
remote: Enumerating objects: 10, done.
remote: Counting objects: 100% (10/10), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 10 (delta 4), reused 10 (delta 4)
From github.com:author/repo_link

* [new branch] main -> upstream/main
```

Git sees new branches or commits in the upstream repository and downloads them to your local machine, labeling them under `upstream/`.

II. Merge the upstream branch into your local branch:

```bash
git merge upstream/branch_name
```

Replace `branch_name` with the actual branch you want to update (e.g., `main` or `master`).

Expected output (simplified example):

```
Updating ab1c2d3..ef4g5h6
Fast-forward
README.md         |   5 +++++
src/new_feature.c |  30 +++++++++++++++++++++++++
2 files changed, 35 insertions(+)
```

You’ve successfully merged upstream’s changes into your local branch, and Git shows you which files changed and how many lines were added or removed.

### Creating a Pull Request

When you’re ready to share your changes with the original project, you create a pull request (PR).

```
.________________________________________________________.
|                                                        |
|                    PULL REQUEST FLOW                   |
|________________________________________________________|

(Local Machine)             (Your Fork)                (Original Repo)
      │                          │                            │
      │ git push origin          │                            │
      │------------------------->│ (on GitHub)                │
      │                          │                            │
      │    [Open Pull Request]   │                            │
      │                          │ ----> [Proposed Changes] ->│
      │                          │                            │
      │                          v                            v
      │                   .------------------.          .------------------.
      │                   |    Compare &     |          |                  |
      │                   |   Review Changes |   <----- | Maintainers      |
      │                   '------------------'          |  Merge or Close  |
      │                          │                      '------------------'
      │                          v
      │              [Potential Merge]                    
      │                          │                            
      v                          v                            
.----------------.        .---------------.                      
|  Continue Dev  |        | Original Repo | <--- Updated if PR is merged
'----------------'        '---------------'
```

- **Local Machine**: You do your work here, commit changes, then push them to **your fork**.
- **Your Fork**: On GitHub, it holds your changes. You open a **Pull Request** from here.
- **Original Repo**: Maintainers review the Pull Request. If approved, they merge your changes into the original project.  

Here’s the process:

I. Commit and push your changes to your fork:

```bash
git add .
git commit -m "Describe your changes"
git push origin branch_name
```

Expected output (after `git push`, for example):

```
Counting objects: 5, done.
Compressing objects: 100% (3/3), done.
Total 3 (delta 2), reused 0 (delta 0)
To github.com:your_username/repo_link.git

* [new branch]   branch_name -> branch_name
```

Your local commits are now uploaded to your fork on GitHub in the specified branch.

II. Create the pull request on GitHub:

- Go to your fork on GitHub.
- Click Pull Request.
- Click the New Pull Request or Create Pull Request button.
- Provide a clear title and description so maintainers understand what your changes do.

### Handling Pull Requests (as the Maintainer)

If you maintain the upstream repository and someone else has opened a PR, you can fetch their code, review it locally, and merge it if everything checks out.

I. Go to your local copy of the upstream repository:

```bash
cd path/to/your/local/repo
```

II. Add the contributor’s repo as a remote:

```bash
git remote add other_user git://github.com/other_user/repo_link
```

No output if successful. If you see `fatal: remote other_user already exists.`, pick a different remote name.

III. Fetch and merge the contributor’s changes:

```bash
git fetch other_user
git merge other_user/branch_name
```

Expected output (example):

```
From github.com:other_user/repo_link

* [new branch]   feature   -> other_user/feature
Updating d3a4e5f..a1b2c3d
Fast-forward
file_added_by_contributor.py  | 20 ++++++++++++++++++++
```

You’ve fetched the new branch, and Git shows that it merged contributor changes into your local branch.

IV. Push the merged changes to GitHub:

```bash
git push origin branch_name
```

Expected output:

```
Everything up-to-date
```

or a short log of updated objects.

Your local changes are now synced to GitHub. The pull request will usually show as merged on the GitHub website.

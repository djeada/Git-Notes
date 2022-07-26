## Setting up your own Git server

You don't have to use hosting services like GitHub to use Git. If you have access to a server, you may install Git on it and create your repositories manually.

Installing git on Debian based systems:

```bash
sudo apt install git-core
```

## Creating a repository

Assume you've opted to keep your Git repositories under /opt/git/.
Then, if you want to create a new repository called *new_git_repo*, do the following: 

```bash
mkdir -p /opt/git/new_git_repo.git
cd /opt/git/new_git_repo.git
git init --bare
```

Anyone with SSH access to your server may now clone the repository using the following command (assuming your username is user and the machine's IP address is 192.168.1.10):

```bash
git clone git+ssh://user@192.168.1.10/opt/git/new_git_repo.git
```

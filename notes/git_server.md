## Setting up a Git server

Instead of using hosting services like GitHub, you can set up your own Git server by installing Git on a server and creating your repositories manually.

### Installing Git on a Debian-based system

To install Git on a Debian-based system, use the following command:

```bash
sudo apt install git-core
```

### Creating a repository

To create a new Git repository called new_git_repo, follow these steps:

1. Create a directory for the repository:

```bash
mkdir -p /opt/git/new_git_repo.git
```

2. Navigate to the directory:

```bash
cd /opt/git/new_git_repo.git
```

3. Initialize the repository as a bare repository:

```bash
git init --bare
```

Anyone with SSH access to the server can now clone the repository using the following command (replace user with your username and 192.168.1.10 with the IP address of the machine):

```bash
git clone git+ssh://user@192.168.1.10/opt/git/new_git_repo.git
```

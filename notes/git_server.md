# How to set up your own Git server

If you're a backend developer, you probably already know the importance of having a version control system like Git. While GitHub is a great platform, setting up your own Git server can offer more flexibility and control over your code.

Here's how you can set up your own Git server:

## Installing Git on a Debian-based system

To install Git on a Debian-based system, use the following command:

```bash
sudo apt install git-core
```

## Creating a repository

To create a new Git repository, follow these steps:

Choose a location for your repository. For example, you can create a directory called myrepo in /opt/git/:

```bash
sudo mkdir -p /opt/git/myrepo.git
```
Change to the newly created directory:

```bash
cd /opt/git/myrepo.git
```

Initialize the repository as a bare repository:

```bash
sudo git init --bare
```

## Configuring access to the repository

To allow others to access the repository, you need to configure access to the server. Assuming you're using SSH, here's how you can do it:

Create a new user for Git:

```bash
sudo adduser git
```
Set the password for the user:

```bash
sudo passwd git
```

Allow SSH access for the user:

```bash
sudo su git
mkdir ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Add the public key of the user who will be accessing the repository:

```bash
cat /path/to/public_key >> ~/.ssh/authorized_keys
```

## Cloning the repository

Now that the repository is set up and access is configured, anyone with SSH access to the server can clone the repository using the following command:

```bash
git clone git@yourserver:/opt/git/myrepo.git
```

That's it! You now have your own Git server up and running.

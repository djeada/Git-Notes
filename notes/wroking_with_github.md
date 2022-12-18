## Why learn Git?

Git is a version control system that allows you to track changes in your code over time, revert to previous versions if necessary, and collaborate with others on projects. There are several reasons why learning Git can be beneficial:

* It is a valuable skill in the software industry and is used by many companies as a standard for version control.
* Git can help you make backups of your projects, particularly if you use an external server like GitHub.
* It allows you to keep a record of your progress, from simple programs to large projects that span several years.
* If something goes wrong and you don't know when or where the problem first appeared, you can use Git to check out the commit with the known working version.
* If you use multiple machines (or one machine and many virtual machines), Git can help you synchronize work on a single project between all of those devices.
* You can use Git to collaborate with other developers (such as your friends) on projects.
* You can use Git to show your code to others (including potential employers). This is particularly useful if you use an internet-accessible Git server like GitHub.

## Who is using Git?

Git is used by a wide range of organizations and individuals, including:

* Operating system kernels: https://github.com/torvalds/linux
* Scientists: https://github.com/nasa
* Most popular frameworks: https://github.com/facebook/react
* Government agencies: https://github.com/nationalsecurityagency
* Authors of textbooks: https://github.com/d2l-ai/d2l-en

## Git configuration

Before you start using Git, you should configure your user name and email address. This will ensure that your commits are correctly attributed to you. Use the following commands to set your user name and email address:

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

## Setting up SSH

To set up SSH for Git, you will need to generate a public-private key pair. Check if the files ~/.ssh/id_rsa and ~/.ssh/id_rsa.pub already exist on your system. If they do, then you don't need to take any further action. If they don't exist, then you can generate a new key pair using the following command:

```bash
ssh-keygen -t rsa -C "your@email.com"
```

This will create a new public-private key pair and store it in the ~/.ssh directory. To display the contents of your public key, use the following command:

```bash
cat ~/.ssh/id_rsa.pub
```

Copy and paste the contents of the public key into your GitHub account settings to set up your SSH key.

## Fork a repo
A fork is a copy of a repository that you can modify without affecting the original repository. When you fork a repository, you can make changes and push them to your version of the repository on GitHub, but the changes will not be reflected in the original repository until you submit a pull request.

To fork a repository:

1. Go to the repository on GitHub.
2. Click the "Fork" button in the upper right corner of the page.

You now have your own copy of that repository on your github account.

### Cloning a forked repository

To download a forked repository to your local machine, use the git clone command and specify the URL of the repository on your GitHub account:

```bash
git clone git@github.com:your_username/repo_link
```

This will create a local copy of the repository on your machine.
Adding the upstream repository

It can be useful to keep your fork synchronized with the upstream (original) repository, especially if you plan to submit pull requests. To do this, you can add the upstream repository as a remote in your local repository. Use the following command to add the upstream repository:

```bash
git remote add upstream git://github.com/author/repo_link
```

You can then use git fetch upstream to retrieve updates from the upstream repository and git merge upstream/branch_name to merge the updates into your local branch.

### Creating a pull request

After making changes to a forked repository and pushing them to GitHub, you can submit a pull request to the upstream repository to request that the changes be merged into the upstream branch.

To create a pull request:

1. Go to your fork of the repository on GitHub.
1. Click the "Pull Request" button at the top of the page.
1. Click the green "Create Pull Request" button.
1. Give your pull request a descriptive title and add a brief description of the changes you made.

The maintainer of the upstream repository will then be able to review your changes and decide whether or not to merge them into the upstream branch.
Handling pull requests

If you are the maintainer of an upstream repository and someone else has submitted a pull request, you can review and merge the changes using the following steps:

1. Navigate to the directory containing your local copy of the repository.
2. If you haven't already, add the other user's repository as a remote:

```bash
git remote add other_user git://github.com/other_user/repo_link
```

3. Fetch the changes from the other user's repository and merge them into your local branch:

```bash
git fetch other_user
git merge other_user/branch_name
```

4. Push the changes to your upstream repository on GitHub:

```bash
git push origin branch_name
```

This will merge the changes from the other user's repository into your upstream repository on GitHub.

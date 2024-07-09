## Why Learn Git?

Git is a powerful and widely-used version control system that is essential for managing code changes, collaborating with others, and maintaining the integrity of your projects. Here are several reasons why learning Git can be highly beneficial:

- In the software industry, Git is a valuable skill that is widely adopted by many companies for version control.
- Using Git enables the creation of backups for projects, especially when utilizing external servers like GitHub.
- Maintaining a detailed record of progress, from initial drafts to complex, multi-year projects, is made possible with Git.
- When errors occur, Git provides the ability to revert to a previous, functioning version, which aids in troubleshooting and fixing issues.
- For those working on multiple machines or virtual environments, Git helps synchronize work on a single project across all devices, ensuring consistency.
- Collaboration with other developers is streamlined through Git, allowing for effective management of contributions and merging of changes.
- Showcasing code to others, including potential employers, is facilitated by Git, particularly when using platforms like GitHub, making it an excellent tool for demonstrating skills and projects.

## Who is Using Git?

Git is utilized by a diverse array of organizations and individuals, demonstrating its versatility and importance across various fields. Here are some notable examples:

- The Linux kernel, one of the most critical components of many operating systems, is managed using Git and is available on [GitHub](https://github.com/torvalds/linux).
- Organizations like NASA use Git to manage and collaborate on scientific projects and research, showcasing its application in the scientific community ([GitHub Repository](https://github.com/nasa)).
- Major software frameworks, such as Facebook's React, rely on Git for version control and collaborative development, highlighting its importance in popular software development ([GitHub Repository](https://github.com/facebook/react)).
- Government agencies, including the National Security Agency (NSA), utilize Git for managing and sharing open-source projects, indicating its use in government operations ([GitHub Repository](https://github.com/nationalsecurityagency)).
- Authors of educational resources, such as the "Dive into Deep Learning" book, manage and update their content using Git, demonstrating its role in the education sector ([GitHub Repository](https://github.com/d2l-ai/d2l-en)).

## Git Configuration

Before you start using Git, it is essential to configure your user name and email address. This configuration ensures that your commits are correctly attributed to you, maintaining a clear record of your contributions to any project. Follow these steps to set your user name and email address:

I. Open your terminal or command prompt.

II. Use the following commands to set your user name:

```bash
git config --global user.name "Your Name"
```

III. Set your email address with the command:

```bash
git config --global user.email "your@email.com"
```

These commands configure Git to use the specified user name and email address globally for all your repositories. If you need to set different credentials for a specific repository, omit the `--global` flag.

## Setting Up SSH

Setting up SSH for Git provides a secure and convenient way to authenticate with remote repositories, such as those hosted on GitHub. To set up SSH, you need to generate a public-private key pair. Here is a step-by-step guide to accomplish this:

I. First, check if an SSH key pair already exists on your system by looking for the files `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`. You can do this with the command:

```bash
ls -al ~/.ssh
```

II. If the files exist, you can skip the key generation step. If they do not exist, generate a new key pair using the command:

```bash
ssh-keygen -t rsa -C "your@email.com"
```

This command creates a new RSA key pair and stores it in the `~/.ssh` directory.

III. Follow the prompts to save the key in the default location (`~/.ssh/id_rsa`) and to enter a passphrase for added security.

IV. To view the contents of your public key, use the command:

```bash
cat ~/.ssh/id_rsa.pub
```

This command displays your public key, which you need to copy.

V. Finally, add your SSH key to your GitHub account:

- Log in to your GitHub account.
- Navigate to **Settings** > **SSH and GPG keys**.
- Click on **New SSH key** and paste the contents of your public key into the "Key" field.
- Give your key a descriptive title and save it.

## Fork a repo
A fork is a copy of a repository that you can modify without affecting the original repository. When you fork a repository, you can make changes and push them to your version of the repository on GitHub, but the changes will not be reflected in the original repository until you submit a pull request.

To fork a repository:

1. Go to the repository on GitHub.
2. Click the "Fork" button in the upper right corner of the page.

You now have your own copy of that repository on your github account.

### Cloning a Forked Repository

To download a forked repository to your local machine, use the `git clone` command and specify the URL of the repository from your GitHub account. This command creates a local copy of the repository on your machine:

```bash
git clone git@github.com:your_username/repo_link
```

### Adding the Upstream Repository

Keeping your fork synchronized with the upstream (original) repository is crucial, especially if you plan to submit pull requests. This process involves adding the upstream repository as a remote in your local repository. Here’s how to do it:

I. Navigate to your local repository directory:

```bash
cd repo_link
```

II. Add the upstream repository using the following command:

```bash
git remote add upstream git://github.com/author/repo_link
```

With the upstream repository added, you can now fetch and merge updates from the upstream repository into your local repository. Use these commands to keep your fork up-to-date:

I. Fetch updates from the upstream repository:

```bash
git fetch upstream
```

II. Merge the updates into your local branch:

```bash
git merge upstream/branch_name
```

Replace `branch_name` with the name of the branch you want to update (e.g., `main` or `master`).

### Creating a Pull Request

After making changes to a forked repository and pushing them to GitHub, you can submit a pull request to the upstream repository. This request allows the maintainer of the original repository to review your changes and potentially merge them. Follow these steps to create a pull request:

I. Ensure all your changes are committed and pushed to your forked repository on GitHub:

```bash
git add .
git commit -m "Describe your changes"
git push origin branch_name
```

Replace `branch_name` with the name of your feature branch.

II. Go to your fork of the repository on GitHub.

III. Click the "Pull Request" button at the top of the page.

IV. Click the green "Create Pull Request" button.

V. Give your pull request a descriptive title and add a brief description of the changes you made. This helps the maintainer understand the purpose and scope of your modifications.

After submitting, the maintainer of the upstream repository will review your pull request. They may ask for further modifications, accept, or reject the pull request based on the quality and relevance of your changes.

### Handling Pull Requests

If you are the maintainer of an upstream repository and someone else has submitted a pull request, you can review and merge the changes using the following steps:

I. Navigate to the directory containing your local copy of the repository:

- Open your terminal or command prompt.
- Use the `cd` command to change to the directory where your local repository is stored:

```bash
cd path/to/your/local/repo
```

II. If you haven't already, add the other user's repository as a remote. This step ensures you can fetch and merge their changes:

```bash
git remote add other_user git://github.com/other_user/repo_link
```

III. Fetch the changes from the other user's repository and merge them into your local branch. This process involves two commands:

First, fetch the changes from the other user's remote repository:

```bash
git fetch other_user
```

Then, merge the fetched changes into your local branch:

```bash
git merge other_user/branch_name
```

Replace `branch_name` with the name of the branch containing the changes from the pull request.

IV. Push the merged changes to your upstream repository on GitHub. This step updates the repository on GitHub with the changes you have just merged:

```bash
git push origin branch_name
```

Again, replace `branch_name` with the appropriate branch name.

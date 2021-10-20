<h1>Git configuration</h1>

Save your user name and email address for future usage with git:

```bash
git config --global user.name "example_name"
git config --global user.email "email@gmail.com"
```

<h1>Setup ssh</h1>

Check to see if the files ~/.ssh/id_rsa and ~/.ssh/id_rsa.pub exist on your system.

If this is the case, there is no need for you to take any action. Otherwise, you must generate a public-private key pair:

```bash
ssh-keygen -t rsa -C "email@gmail.com"
```

Copy and paste your public key into your github account settings. To display the file's contents, use:

```bash
cat ~/.ssh/id_rsa.pub
```

<h1>Fork a repo</h1>
A fork is a duplicate of a repository. When you fork a repository, your modifications will have no effect on the original project until you submit a pull request.

1. Go to the github repository.
2. In the upper right corner, click the "Fork" button.

You now have your own copy of that repository on your github account.

Use the following command to download the repository to your computer:

```bash
git clone git@github.com:your_username/repo_link
```

Add the link to the original repository:

```bash
git remote add author git://github.com/author/repo_link
```

You may now make modifications and push them to github. They will be directed to your version of the repository.

<h1>Creating a pull request</h1>

After making modifications in a forked repository and pushing them to github go to your version of the repository on github.

1. At the top, click the "Pull Request" button.

2. Click the green "Create pull request" button.

3. Give a simple and informative title, and in the remark section, provide a brief description of the modifications.

The original repository's creator can now choose whether or not to include your modifications in their version of the repository.

<h1>Handling a pull requests</h1>

To accept someone else's pull request to your repository, follow these steps:

1. Navigate to the directory containing your project.

2. If you haven't previously, connect to your the other user's version of the github repository:

```bash
git remote add other_user git://github.com/other_user/repo_link
```

3. Pull the changes:

```bash
git pull other_user branch_name
```

4. Push the changes back to your github repository:

```bash
git push origin branch_name
```

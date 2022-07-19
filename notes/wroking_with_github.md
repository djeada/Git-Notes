## Why would you want to learn Git?

* Practice now because you will (hopefully) need it at work someday.
* It might help in making backups of your projects (especially if you use external server like Github).
* It will archive your progress, from simple programs to large projects that span several years.
* It may be used similarly as a save system in old RPG games. If anything goes wrong and you don't know when or where the problem first appeared, you can check out the commit with the known working version.
* If you use multpile machines (or one machine and many virtual machines) you can synchronize work on single project between all of those devices.
* The same goes for work with other developers (your friends). 
* You may also use it to show people your code (also potential employers). This is especially true if you chose to use an internet-accessible Git server. 

## Git configuration

Save your user name and email address for future usage with git:

```bash
git config --global user.name "example_name"
git config --global user.email "email@gmail.com"
```

## Setup ssh

Check to see if the files ~/.ssh/id_rsa and ~/.ssh/id_rsa.pub exist on your system.

If this is the case, there is no need for you to take any action. Otherwise, you must generate a public-private key pair:

```bash
ssh-keygen -t rsa -C "email@gmail.com"
```

Copy and paste your public key into your github account settings. To display the file's contents, use:

```bash
cat ~/.ssh/id_rsa.pub
```

## Fork a repo
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

## Creating a pull request

After making modifications in a forked repository and pushing them to github go to your version of the repository on github.

1. At the top, click the "Pull Request" button.

2. Click the green "Create pull request" button.

3. Give a simple and informative title, and in the remark section, provide a brief description of the modifications.

The original repository's creator can now choose whether or not to include your modifications in their version of the repository.

## Handling a pull requests

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

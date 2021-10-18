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

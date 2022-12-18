## Creating tags

To create a new tag, use the git tag command followed by the name of the tag and the commit hash or reference. For example:

```bash
git tag v2.0 b4d373k8990g2b5de30a37bn843b2f51fks2b40
```

This will create a new tag called v2.0 that points to the commit with the hash b4d373k8990g2b5de30a37bn843b2f51fks2b40.

You can also specify the commit reference directly, such as the branch name or a commit message. For example:

```bash
git tag v2.0 master
git tag v2.0 "Fix bug in login form"
```

## Annotated tags

By default, tags in Git are "lightweight," meaning they are simply pointers to a specific commit and do not contain any additional metadata. However, you can also create "annotated" tags, which are tags that contain additional information such as a message and the tagger's name and email address.

To create an annotated tag, use the -a flag followed by the tag name and a message. For example:

```bash
git tag -a v2.0 -m "Release version 2.0"
```

## Viewing tags

To see a list of all the tags in your repository, you can use the git tag command without any arguments. For example:

```bash
$ git tag
v1.0
v1.1
v2.0
```

To see the details of a specific tag, you can use the git show command followed by the tag name. For example:

```bash
git show v2.0
```

This will show the commit the tag points to, as well as any additional metadata such as the tag message and tagger information.
Pushing tags to a remote repository

By default, tags are not pushed to a remote repository when you use the git push command. To push tags to a remote repository, you can use the git push command followed by the --tags flag. For example:

```bash
git push --tags
```

This will push all the tags in your local repository to the remote repository. You can also push a specific

## Tags vs baranches
Both branches and tags refer to a commit. Their behavior, on the other hand, is different. The tag is associated with a single commit, whereas the branch changes from one commit to the next as new code is added.

The most significant distinction is in how they are used. When working on a feature, you use branches to organize code over time. Tagging, in general, refers to metadata linked with a build or deployment that is automatically or manually updated.

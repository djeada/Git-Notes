## Checking Changes

One of the key features of Git is the ability to view and compare changes made to the codebase. This can be helpful for debugging, collaborating with others, and understanding the evolution of the project.

There are several commands you can use to view and compare changes in Git. These include:

### Viewing Changes

* `git status`: Lists new or modified files that have not yet been committed.
* `git diff`: Shows the changes to files that have not yet been staged.
* `git diff --cached`: Shows the changes to files that have been staged but not yet committed.
* `git diff HEAD`: Shows all staged and unstaged file changes.
* `git diff --word-diff`: Shows word-level changes in the diff output.
* `git blame filename`: Lists the change dates and authors for a file.

### Comparing Commits

* `git log`: Shows the repository's complete history of changes.
* `git diff commit_1 commit_2`: Shows the changes between two commit IDs.
* `git show commit_id:filename:` Shows the file changes for a commit ID and/or file.

## Tips and Best Practices

* Use clear and descriptive commit messages to help others understand the changes you've made.
* Use git diff to review your changes before staging and committing them. This can help catch mistakes and ensure that you're only committing relevant changes.
* When working with large or complex changes, consider using branches to isolate and manage different parts of the work. This can make it easier to review and merge changes later on.
* Understand the different stages of the Git workflow (e.g. staging, committing, pushing) and how they fit into your workflow. This can help you manage changes effectively and avoid mistakes.

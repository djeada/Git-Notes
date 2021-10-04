TODO: 
* add undo for add and commit 
* change commit message
* remove commit

Stages the file, ready for commit:

git add filename

Stage all changed files, ready for commit:

git add .

Commit all staged files to versioned history:

git commit -m “commit message”

Unstages file, keeping the file changes:

git reset filename

Revert everything to the last commit:

git reset --hard

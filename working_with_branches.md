List all local branches:

git branch

List all branches, local and remote:

git branch -av

Switch to a branch_name and update working directory:

git checkout branch_name

Create a new branch called branch_name

git checkout -b branch_name

Delete a branch called branch_name:

git branch -d branch_name

Merge branch_a into branch_b:

git checkout branch_b

git merge branch_a

go to a specific commit:

git checkout commit_number

Add a specific commit to your branch:

git cherry-pick commit_number

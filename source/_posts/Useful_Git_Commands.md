---
title: Useful Git Commands
date: 2019-08-25
categories:
- General
tags:
- blogs
---

`git log`: Output git commits history.
`git add <filename>`: Add files to the git tracking and committing system.
`git status`: Check Git Status
`.gitignore`: Input files that will be ignored by Git. **But if files are already inside the git tracking system, it won't delete it from git. We have to use `git rm filename` to remove it.**
`git diff`: check file changed that is not staged. 
`gif diff --staged`: check file changed that is staged.
`git add --all`: add all changes to the git tracking.
`git commit`: Submit changes to git.
`git commit -a`: Submit changes with all changes. Still need to call `git add all`
`git commit --amend`: Revert and re-commit the last git submission.

`git rm –cached < filename >`: Delete file from git submission. But does not delete local file.
`git reset – < file >`: change the current HEAD pointers to previous commits. Depends on --hard or --soft, will or will not change the local file.


`git checkout -- <file>`: Change to local file to specific commits. Does not move HEAD pointer in Git.

`git remote -v`: Output remote server and its URL.
`git remote add <short_name> <url>`:
`git remote rm <short_name>`:

`git fetch <remote-name>`:  Receive the remote server's update. Will put into <remote-name>/branch.
`git pull`: Automatically fetch and then merge into current.
`git push <remote-name> <branch-name>`: push to the remote server.
`git push <remote-name> --tags`: push to the remote server with tags information.

`git tag -d <tagname>`: Delete tags information

`git checkout master`:
`git merge <other_branch_name>`:
Git will decide which branch will become its ancestor.

If we have conflicts in merge.
`git mergetool`: GUI tool used to fix conflicts.
Then use `git merge` to merge.

`git branch`: Output all branches information.


It is OK to repeatedly merge other branches to one branch. For example, we can repeatedly merge *test_branch* to *master* branch. 

`git push origin --delete <branch_name>` Delete remote branch.

`git rebase -i HEAD~3` Change the last three commits, very useful tool to re-generate commits history.




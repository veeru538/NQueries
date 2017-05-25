Git cheat sheet is a git command list including common command when using command line.
# This article content some main point as create git, local changes, commit history, branches and tags, update and publish, merge and rebase and undo commit

    Create

### Clone an existing repository
$ git clone ssh://user@domain.com/repo.get

### Create a new local repository
$ git init

    Local Changes

### Changed files in your working directory
$ git status

### Changes to tracked files
$ git diff

### Add all current changes to the next commit
$ git add .

### Add some changes in <file> to the next commit
$ git add -p <file>

### Commit all local changes in tracked files
$ git commit -a

### Commit previously staged changed
$ git commit

### Change the last commit Don't ammend published commits!
$ git commit --amend

    Commit History

### Show all commits, starting with newest
$ git log

### Show changes over time for a specific file
$ git log -p <file>

### Who changed what and when in <file>
$ git blame <file>

    Branches and Tags

### List all existing branches
$ git branch

### Switch HEAD branch
$ git checkout <branch>

### Create a new branch based on a remote branch
$ git branch --track <new-branch> <remote-branch>

### Delete a local branch
$ git branch -d <branch>

### Mark the current commit with a tag
$ git tag <tag-name>

    Update and Publish

### List all currently configured remotes
$ git remote -v

### Show information about a remote
$ git remote show <remote>

### Add new remote repository, named <remote>
$ git remote add <remote> <url>

### Download all changes from <remote>, but don't integrate into HEAD
$ git fetch <remote>

### Download changes and directly merge/integrate into HEAD
$ git pull <remote> <branch>

### Publish local changes on remote
$ git push <remote> <branch>

### Delete a branch on the remote
$ git push <remote> :<branch>

### Publish your tags
$ git push --tags

    Merge and Rebase

### Merge <branch> into your current HEAD
$ git merge <branch>

### Rebase your current HEAD onto <brach>
Don't rebase published commits!

$ git rebase <branch>

### Abort a rebase
$ git rebase --abort

### Continue a rebase after resolving conflicts
$ git rebase --continue

### Use your configured merge tool to solve conflicts
$ git mergetool

### Use your editor to manually solve conflicts and (after resolving) mark file as resolved
$ git add <resolved-file>
$ git rm <resolved-file>

    Undo

### Discard all local changes in your working directory
$ git reset --hard HEAD

### Discard local changes in a specific file
$ git checkout HEAD <file>

### Revert a commit (by producing a new commit with contrary changes)
$ git revert <commit>

### Reset your HEAD pointer to a previous commit
...and discard all changes since then

$ git reset --hard <commit>

...and preserve all changes as unstaged changes

$ git reset <commit>

...and preserve uncommitted local changes

$ git reset --keep <commit>

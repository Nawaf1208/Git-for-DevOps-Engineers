# Git-for-DevOps-Engineers

## Git Basics

**_1.How do you know if a certain directory is a git repository?_**

- You can check if there is a ".git" directory.

**_2.Explain the following: `git directory`, `working directory` and `staging area`_**

- The `Git directory` is where Git stores the meta-data and object database for your project. This is the most important part of Git, and it is what is copied when you clone a repository from another computer.

- The `working directory` is a single checkout of one version of the project. These files are pulled out of the compressed database in the Git directory and placed on disk for you to use or modify.

- The `staging area` is a simple file, generally contained in your Git directory, that stores information about what will go into your next commit. It’s sometimes referred to as the index, but it’s becoming standard to refer to it as the staging area.

**_3.What is the difference between `git pull` and `git fetch`?_**

- Shortly, `git pull` = `git fetch` + `git merge`

- When you run `git pull`, it gets all the changes from the remote or central repository and attaches it to your corresponding branch in your local repository.

- `git fetch` gets all the changes from the remote repository, stores the changes in a separate branch in your local repository

**_4.How to check if a file is tracked and if not, then track it?_**

- There are different ways to check whether a file is tracked or not:
  - `git ls-files <file>` -> exit code of 0 means it's tracked
  - `git blame <file> ...`
 
**_5.Explain what the file gitignore is used for_**

- The purpose of gitignore files is to ensure that certain files not tracked by Git remain untracked. To stop tracking a file that is currently tracked, use `git rm --cached`.

**_6.How can you see which changes have done before committing them?_**

- `git diff`

**_7.What git status does?_**

- `git status` helps you to understand the tracking status of files in your repository. Focusing on working directory and staging area - you can learn which changes were made in the working directory, which changes are in the staging area and in general, whether files are being tracked or not.

**_8.You've created new files in your repository. How to make sure Git tracks them?_**

- `git add FILES`

## Scenarios

**_9.You have files in your repository you don't want Git to ever track them. What should you be doing to avoid ever tracking them?_**

- Add them to the file .gitignore. This will make sure these files are never added to staging area.

**_10.A development team in your organization is using a monorepo and it's became quite big, including hundred thousands of files. They say running many git operations is taking a lot of time to run (like git status for example). Why does that happen and what can you do in order to help them?_**

- Many Git operations are related to filesystem state. `git status` for example will run diffs to compare HEAD commit to index and another diff to compare index to working directory. As part of these diffs, it would need to run quite a lot of `lstat()` system calls. When running on hundred thousands of files, it can take seconds if not minutes.

- One thing to do about it, would be to use the built-in `fsmonitor` (filesystem monitor) of Git. With fsmonitor (which integrated with Watchman), Git spawn a daemon that will watch for any changes continuously in the working directory of your repository and will cache them . This way, when you run `git status` instead of scanning the working directory, you are using a cached state of your index.

- Next, you can try to enable `feature.manyFile` with `git config feature.manyFiles` true. This does two things:
  - 1.Sets `index.version = 4` which enables path-prefix compression in the index
  - 2.Sets `core.untrackedCache=true` which by default is set to `keep`. The untracked cache is quite important concept. What it does is to record the mtime of all the files and directories in the working directory. This way, when time comes to iterate over all the files and directories, it can skip those whom mtime wasn't updated.

- Before enabling it, you might want to run `git update-index --test-untracked-cache` to test it out and make sure mtime operational on your system.

- Git also has the built-in `git-maintainence` command which optimizes Git repository so it's faster to run commands like `git add` or `git fetch` and also, the git repository takes less disk space. It's recommended to run this command periodically (e.g. each day).

- In addition, track only what is used/modified by developers - some repositories may include generated files that are required for the project to run properly (or support certain accessibility options), but not actually being modified by any way by the developers. In that case, tracking them is futile. In order to avoid populating those file in the working directory, one can use the `sparse checkout` feature of Git.

- Finally, with certain build systems, you can know which files are being used/relevant exactly based on the component of the project that the developer is focusing on. This, together with the `sparse checkout` can lead to a situation where only a small subset of the files are being populated in the working directory. Making commands like `git add`, `git status`, etc. really quick

## Branches

**_11.What's is the branch strategy (flow) you know?_**

- Git flow
- GitHub flow
- Trunk based development
- GitLab flow

**_12.True or False? A branch is basically a simple pointer or reference to the head of certain line of work_**

- True

**_13.You have two branches - main and devel. How do you make sure devel is in sync with main?_**

- `git checkout main`
- `git pull`
- `git checkout devel`
- `git merge main`

**_14.Describe shortly what happens behind the scenes when you run git branch_**

- Git runs update-ref to add the SHA-1 of the last commit of the branch you're on into the new branch you would like to create

**_15.When you run `git branch` how does Git know the SHA-1 of the last commit?_**

- Using the HEAD file: `.git/HEAD`

**_16.What `unstaged` means in regards to Git?_**

- A file that is in the working directory but is not in the HEAD nor in the staging area is referred to as "unstaged".

**_17.True or False? when you `git checkout some_branch`, Git updates .git/HEAD to `/refs/heads/some_branch`_**

- True

## Merge

**_18.You have two branches - main and devel. How do you merge devel into main?_**

- `git checkout main`
- `git merge devel`
- `git push origin main`

**_19.How to resolve git merge conflicts?_**

- First, you open the files which are in conflict and identify what are the conflicts. Next, based on what is accepted in your company or team, you either discuss with your colleagues on the conflicts or resolve them by yourself After resolving the conflicts, you add the files with `git add ` Finally, you run `git rebase --continue`

**_20.What merge strategies are you familiar with?_**

- Fast-forward Merge: Used when the branch being merged is ahead of the target branch without any diverging commits. The target branch pointer is simply moved forward.

- Recursive Merge: The default strategy used when merging two diverging branches. It creates a new merge commit.

- Squash Merge: Combines all commits from the feature branch into a single new commit on the target branch. This cleans up history.

- Rebase: Not technically a merge strategy but an alternative workflow. It moves or combines a sequence of commits to a new base commit, creating a linear history.

- Ours/Theirs: Used to favor one side's changes completely during a merge. 'Ours' discards the incoming changes, while 'Theirs' discards the current branch's changes

**_21.Explain Git octopus merge_**

- Probably good to mention that it's:
  - It's good for cases of merging more than one branch (and also the default of such use cases)
  - It's primarily meant for bundling topic branches together
 
**_22.What is the difference between git reset and git revert?_**

- `git revert` creates a new commit which undoes the changes from last commit.

- `git reset` depends on the usage, can modify the index or change the commit which the branch head is currently pointing at.

## Rebase

**_23.You would like to move forth commit to the top. How would you achieve that?_**

- Using the `git rebase` command

**_24.In what situations are you using `git rebase`?_**

- Suppose a team is working on a `feature` branch that is coming from the `main` branch of the repo. At a point, where the feature development is done, and finally we wish to merge the feature branch into the main branch without keeping the history of the commits made in the feature branch, a `git rebase` will be helpful.

**_25.How do you revert a specific file to previous commit?_**

- `git checkout HEAD~1 -- /path/of/the/file`

**_26.How to squash last two commits?_**

- Interactive Rebase (Recommended)
  - 1.Start an interactive rebase for the last two commits: `git rebase -i HEAD~2`
  - 2.An editor will open with the commits listed (oldest first).
  - 3.Change the `pick` command for the second commit (the most recent one) to `squash` (or s):
    - `pick <commit-hash-1> Commit message 1`
    - `squash <commit-hash-2> Commit message 2`
  - 4.Save and close the editor.
  - 5.Another editor will open to let you write the new, combined commit message.
  - 6.Save and close this final editor.
 
**_27.What is the .git directory? What can you find there?_**

- The `.git` folder contains all the information that is necessary for your project in version control and all the information about commits, remote repository address, etc. All of them are present in this folder. It also contains a log that stores your commit history so that you can roll back to history.

**_28.What are some Git anti-patterns? Things that you shouldn't do_**

- Not waiting too long between commits
- Not removing the .git directory

**_29.How do you remove a remote branch?_**

- You delete a remote branch with this syntax:
  - `git push origin :[branch_name]`

**_30.Are you familiar with gitattributes? When would you use it?_**

- gitattributes allow you to define attributes per pathname or path pattern.

- You can use it for example to control endlines in files. In Windows and Unix based systems, you have different characters for new lines (\r\n and \n accordingly). So using gitattributes we can align it for both Windows and Unix with * text=auto in .gitattributes for anyone working with git. This is way, if you use the Git project in Windows you'll get \r\n and if you are using Unix or Linux, you'll get \n.

**_31.How do you discard local file changes? (before commit)_**

- `git checkout -- <file_name>`

**_32.How do you discard local commits?_**

- `git reset HEAD~1` for removing last commit If you would like to also discard the changes you `git reset --hard`

**_33.True or False? To remove a file from git but not from the filesystem, one should use git rm_**

- False. If you would like to keep a file on your filesystem, use `git reset   <file_name>`

## References

**_34.How to list the current git references in a given repository?_**

- `find .git/refs/`

## Git Diff

**_35.What git diff does?_**

- `git diff` can compare between two commits, two files, a tree and the staging area, etc.

**_36.Which one is faster? git diff-index HEAD or git diff HEAD_**

- `git diff-index` is faster but to be fair, it's because it does less.
- `git diff index` won't look at the content, only metadata like timestamps.

**_37.By which other Git commands does git diff used?_**

- The diff mechanism used by `git status` to perform a comparison and let the user know which files are being tracked

## Git Internal

**_38.Describe how `git status` works_**

- Shortly, it runs git diff twice:
  - 1.Compare between HEAD to staging area
  - 2.Compare staging area to working directory
 
**_39.If `git status` has to run diff on all the files in the HEAD commit to those in staging area/index and another one on staging area/index and working directory, how is it fairly fast?_**

- One reason is about the structure of the index, commits, etc.
  - Every file in a commit is stored in tree object
  - The index is then a flattened structure of tree objects
  - All files in the index have pre-computed hashes
  - The diff operation then, is comparing the hashes

- Another reason is caching
  - Index caches information on working directory
  - When Git has the information for certain file cached, there is no need to look at the working directory file

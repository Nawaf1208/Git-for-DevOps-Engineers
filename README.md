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

Finally, with certain build systems, you can know which files are being used/relevant exactly based on the component of the project that the developer is focusing on. This, together with the `sparse checkout` can lead to a situation where only a small subset of the files are being populated in the working directory. Making commands like `git add`, `git status`, etc. really quick

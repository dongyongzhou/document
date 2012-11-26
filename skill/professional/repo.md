---
layout: master
title: repo
---

# repo

## overview

### Git

Git is an **open-source version-control system** designed to handle **very large projects** that are distributed over **multiple repositories**. 

In the context of Android, we use Git for **local operations** such as **local branching, commits, diffs, and edits**. 

One of the challenges in setting up the Android project was figuring out how to best support the outside community--from the hobbiest community to large OEMs building mass-market consumer devices. We wanted components to be replaceable, and we wanted interesting components to be able to grow a life of their own outside of Android. We first chose a distributed revision control system, then further narrowed it down to Git.

### Repo

Repo is a **repository management tool** that we built on top of Git. 

**Repo unifies the many Git repositories** when necessary, 

	does the uploads to our revision control system
	automates parts of the Android development workflow. 

Repo is not meant to replace Git, only to make it easier to work with Git in the context of Android. The repo command is an **executable Python script** that you can put anywhere in your path. 

In working with the Android source files, you will use Repo for **across-network operations**. For example, with a single Repo command you can download files from multiple repositories into your local working directory.


### Gerrit

Gerrit is a **web-based code review system** for projects that use git. 

Gerrit encourages more centralized use of Git by allowing all authorized users to **submit changes**, which are automatically merged if they pass code review. 

In addition, Gerrit makes **reviewing easier** by displaying changes side by side in-browser and enabling inline comments.


##Basic Workflow

The basic pattern of interacting with the repositories is as follows:

![](http://source.android.com/images/submit-patches-0.png)

- Use repo start to start a new topic branch.
- Edit the files.
- Use git add to stage changes.
- Use git commit to commit changes.
- Use repo upload to upload changes to the review server.

## Installing Repo


Make sure you have a bin/ directory in your home directory, and that it is included in your path:

>	$ mkdir ~/bin
>	$ PATH=~/bin:$PATH

Download the Repo script and ensure it is executable:

>	$ curl https://dl-ssl.google.com/dl/googlesource/git-repo/repo > ~/bin/repo
>	$ chmod a+x ~/bin/repo


## Task reference

![](http://source.android.com/images/git-repo-1.png)

### help

>	repo help 

provides command-line help for repo sub-commands. For example to display help on the repo upload command

>	$ repo help upload


### git config

When the git commit author is not set and Gerrit rejects it

Fix your ID by doing the following:

>	$ git config --global user.email user@xxxx.com
>	$ git config --global user.name 'First Last <user@xxxx.com>'

Then the commit must be ammended:

>	$ git commit --amend --author 'First Last <user@xxxx.com>'

This will get the correct author on the commit.



### Synchronizing your client

To synchronize the files for all available projects:

>	$ repo sync

Syncing with multiple threads

Newer versions of tools/repo.git support multiple threads

We suggest no more than 4 as then you will notice slowness from your local machine disk I/O, and extra burden on the remote git server.

>	$ repo sync --jobs=4 or repo sync -j4


To synchronize the files for selected projects:

>	$ repo sync PROJECT0 PROJECT1 PROJECT2 ...


### Creating topic branches

Start a topic branch in your local work environment whenever you begin a change, for example when you begin work on a bug or new feature. 

A topic branch is not a copy of the original files; it is a **pointer to a particular commit**. This makes creating local branches and switching among them a light-weight operation. By using branches, you can isolate one aspect of your work from the others. 

To start a topic branch using Repo:

>	$ repo start BRANCH_NAME

To verify that your new branch was created:

>	$ repo status

### Using topic branches

To assign the branch to a particular project:

>	$ repo start BRANCH_NAME PROJECT

To switch to another branch that you have created in your local work environment:

>	$ git checkout BRANCH_NAME

To see a list of existing branches:

>	$ git branch

	or

>	$ repo branches

The name of the current branch will be preceded by an asterisk.

- Note: A bug might be causing repo sync to reset the local topic branch. If git branch shows * (no branch) after you run repo sync, then run git checkout again.


### Staging files

By default, Git notices but does not track the changes you make in a project. In order to tell git to preserve your changes, you must mark them for inclusion in a commit. This is also called "staging".

You can stage your changes by running

>	git add

which accepts as arguments any files or directories within the project directory. Despite the name, git add does not simply add files to the git repository; it can also be used to stage file modifications and deletions.


### Viewing client status

To list the state of your files:

>	$ repo status

To see uncommitted edits:

>	$ repo diff

The repo diff command shows** every local edit that you have made that would not go into the commit**, if you were to commit right now. 

To see **every edit that would go into the commit if you were to commit right** now, you need a Git command, **git diff**. Before running it, be sure you are in the project directory


>	$ cd ~/WORKING_DIRECTORY/PROJECT  
>	$ git diff --cached

### Committing changes

A commit is the basic unit of revision control in git, consisting of a snapshot of directory structure and file contents for the entire project. 

Creating a commit in git is as simple as typing

>	git commit

You will be prompted for a commit message in your favorite editor; please provide a helpful message for any changes you submit to the AOSP. If you do not add a log message, the commit will be aborted.

### Uploading changes to Gerrit

Before uploading, update to the latest revisions:

>	repo sync

Next run

>	repo upload

This will list the changes you have committed and prompt you to select which branches to upload to the review server. If there is only one branch, you will see a simple y/n prompt.

Adding a mandatory Reviewer

>	$ repo upload --reviewers=xxxx@xxxx.com


### Recovering sync conflicts

If a repo sync shows sync conflicts:

- View the files that are unmerged (status code = U).
- Edit the conflict regions as necessary.
- Change into the relevant project directory, run **git add** and **git commit** for the files in question, and then "rebase" the changes. For example:

>	$ git add .
>	$ git commit 
>	$ git rebase --continue


When the rebase is complete start the entire sync again:

>	$ repo sync PROJECT0 PROJECT1 ... PROJECTN

### Cleaning up your client files

To update your local working directory after changes are merged in Gerrit:

>	$ repo sync

To safely remove stale topic branches:

>	$ repo prune

### Deleting a client

Because all state information is stored in your client, you only need to delete the directory from your filesystem:

>	$ rm -rf WORKING_DIRECTORY

Deleting a client will permanently delete any changes you have not yet uploaded for review.

## example

### Android example

Initializing a Repo client

After installing Repo, set up your client to access the android source repository:

Create an empty directory to hold your working files. If you're using MacOS, this has to be on a case-sensitive filesystem. Give it any name you like:

>	$ mkdir WORKING_DIRECTORY
>	$ cd WORKING_DIRECTORY

Run repo init to bring down the latest version of Repo with all its most recent bug fixes. You must specify a URL for the manifest, which specifies where the various repositories included in the Android source will be placed within your working directory.

>	$ repo init -u https://android.googlesource.com/platform/manifest

To check out a branch other than "master", specify it with -b:

>	$ repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1

When prompted, please configure Repo with your real name and email address. 

A successful initialization will end with a message stating that Repo is initialized in your working directory. Your client directory should now contain a .repo directory where files such as the manifest will be kept.


To pull down files to your working directory from the repositories as specified in the default manifest, run

>	$ repo sync

The Android source files will be located in your working directory under their project names. 


## special cases




### Linux AU tags

The below procedure requires a completely new sync'd tree but is necessary in some circumstances. However, much quicker and easier is, especially if only going back a few days (wouldn't recommend it to go back too far):


>	$ repo forall -c 'git checkout au-01.08.01.xxx'


Each Android AU Release creates a **versioned manifest** and **tags** for all projects. 

- The versioned manifest (versioned.xml) lists the specific commit IDs for all projects at AU creation time. 
- The versioned manifest and tags point to **the same commit IDs**.
- The versioned manifest is itself tagged using the **AU name,** allowing Repo to pull the Android source at a specific AU tag.

### Sync an Android source tree at a specific AU using both the tag name and versioned manifest.


>	$ repo init -u git://git.quicinc.com/platform/manifest.git -b refs/tags/AU_LINUX_ANDROID_GINGERBREAD.02.03.01.00.037 -m versioned.xml

>	$ repo sync

### Syncing AUs
The AU tag is given to repo to pull a source branch at the release point

>  	  $ repo init -u git://git.quicinc.com/platform/manifest.git -b refs/tags/au-01.08.00.006
>	  $ repo sync



### Download a patch

#### repo downlod

The 'repo download' command downloads a change from the review system
and makes it available in your project's local working directory.


>	repo download kernel/msm 30366/1

#### checkout

Fetches named heads or tags from one or more other repositories, along with the objects necessary to complete them.


>	git fetch ssh://xxx.com:29418/kernel/xxx refs/changes/66/30366/1 && git checkout FETCH_HEAD

#### pull

> 	git pull ssh://xxx.com:29418/kernel/xxx refs/changes/66/30366/1

#### cherry-pick

Apply the changes introduced by some existing commits



>	git fetch ssh://xxx.com:29418/kernel/xxx refs/changes/66/30366/1 && git cherry-pick FETCH_HEAD

#### patch

>	git fetch ssh://xxx.com:29418/kernel/xxx refs/changes/66/30366/1 && git format-patch -1 --stdout FETCH_HEAD



## Reference

repo help [command]

usage: repo COMMAND [ARGS]

The most commonly used repo commands are:

    abandon      Permanently abandon a development branch
    branch       View current topic branches
    branches     View current topic branches
    checkout     Checkout a branch for development
    cherry-pick  Cherry-pick a change.
    diff         Show changes between commit and working tree
    download     Download and checkout a change
    grep         Print lines matching a pattern
    init         Initialize repo in the current directory
    list         List projects and their associated directories
    prune        Prune (delete) already merged topics
    rebase       Rebase local branches on upstream branch
    smartsync    Update working tree to the latest known good revision
    stage        Stage file(s) for commit
    start        Start a new branch for development
    status       Show the working tree status
    sync         Update working tree to the latest revision
    upload       Upload changes for code review

See 'repo help <command>' for more information on a specific command.

See 'repo help --all' for a complete list of recognized commands.


## repo operating 

## reference

* [Version Control with Repo and Git](http://source.android.com/source/version-control.html)





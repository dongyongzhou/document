---
layout: master
title: git
---

# git


## Reference

* [Pro Git](http://progit.org/book/)

When need help while using Git, there are three ways to get the manual page (manpage) help for any of the Git commands:
    $ git help <verb>
    $ git <verb> --help
    $ man git-<verb>

## Environment Setting

### Installing Git

#### Installing on Linux

    $ apt-get install git-core

>Copy

#### Installing on Windows

Download and install the latest version of [Git for Windows](http://code.google.com/p/msysgit/downloads/list)

    http://code.google.com/p/msysgit
>Copy

### Git Setup

Refer to [First-Time Git Setup](http://progit.org/book/ch1-5.html)

It is time to customize Git environment. You should have to do these things only once; they’ll stick around between upgrades. You can also change them at any time by running through the commands again. 

Git comes with a tool called git config that lets you get and set configuration variables that control all aspects of how Git looks and operates. These variables can be stored in three different places:

+ /etc/gitconfig file: Contains values for every user on the system and all their repositories. If you pass the option --system to git config, it reads and writes from this file specifically.
+ ~/.gitconfig file: Specific to your user. You can make Git read and write to this file specifically by passing the --global option.
+ config file in the git directory (that is, .git/config) of whatever repository you’re currently using: Specific to that single repository. Each level overrides values in the previous level, so values in .git/config trump those in /etc/gitconfig.

### Setup Identity

The first thing you should do when you install Git is to set your user name and e-mail address. 

    $ git config --global user.name "<username>"
    $ git config --global user.email username@example.com

Again, you need to do this only once if you pass the --global option, because then Git will always use that information for anything you do on that system. 
If you want to override this with a different name or e-mail address for specific projects, you can run the command without the --global option when you’re in that project.

Refer for more [First-Time Git Setup](http://progit.org/book/ch1-5.html)

## Basic operation

### Getting a Git Repository

#### Initializing a Repository in an Existing Directory

    $ git init
    $ git add .
    $ git commit -m 'initial project version'

#### Cloning an Existing Repository

    $ git clone [repository url] [<directory>]

Git has a number of different transfer protocols you can use. 

+ git:// protocol, 
+ http(s):// or user@server:/path.git, which uses the SSH transfer protocol.

### Checking the Status of the Files

    $ git status

#### Tracking New Files

    $ git add [new file]
    $ git status

#### Staging New Files

    $ git add [modified file]
    $ git status

If you modify a file after you run git add, you have to run git add again to stage the latest version of the file:

### Ignoring Files

create a file listing patterns to match them named .gitignore.

### Viewing Your Staged and Unstaged Changes

To see what you’ve changed but not yet staged, type git diff with no other arguments, That command compares what is in your working directory with what is in your staging area. The result tells you the changes you’ve made that you haven’t yet staged.

    $ git diff

If you want to see what you’ve staged that will go into your next commit, you can use git diff --cached. (In Git versions 1.6.1 and later, you can also use git diff --staged, which may be easier to remember.) This command compares your staged changes to your last commit: 

### Committing Your Changes

    $ git commit

Doing so launches your editor of choice. 
You can see that the default commit message contains the latest output of the git status command commented out and one empty line on top. You can remove these comments and type your commit message.

Alternatively, you can type your commit message inline with the commit command by specifying it after a -m flag, like this:

    $ git commit -m "comments"

Providing the -a option to the git commit command makes Git automatically stage every file that is already tracked before doing the commit, letting you skip the git add part:

### Removing Files

To remove a file from Git, you have to remove it from your tracked files (more accurately, remove it from your staging area) and then commit. The git rm command does that and also removes the file from your working directory so you don’t see it as an untracked file next time around.

If you modified the file and added it to the index already, you must force the removal with the -f option. 

Another useful thing you may want to do is to keep the file in your working tree but remove it from your staging area. 
To do this, use the --cached option:

    $ git rm --cached [file]

You can pass files, directories, and file-glob patterns to the git rm command. That means you can do things such as

    $ git rm log/\*.log

### Moving Files

    $ git mv file_from file_to


### Viewing the Commit History

    $ git log

### Undoing Things

#### Changing Your Last Commit

    $ git commit -m 'initial commit'
    $ git add forgotten_file
    $ git commit --amend

#### Unstaging a Staged File

    $ git reset HEAD <file>

#### Unmodifying a Modified File

    $ git checkout -- <file>

### Working with Remotes

Managing remote repositories includes knowing how to 
1. add remote repositories, 
2. remove remotes that are no longer valid, 
3. manage various remote branches and define them as being tracked or not

#### Showing Your Remotes

    $ git remote
origin — that is the default name Git gives to the server you cloned from:

You can also specify -v, which shows you the URL that Git has stored for the shortname to be expanded to:

    $ git remote -v

Notice that only the origin remote with an SSH URL(git@github.com:<who>/<repository>.git) is the one I can push to .

#### Adding Remote Repositories

    $ git remote add [shortname] [url]

#### Fetching and Pulling from Your Remotes

    $ git fetch [remote-name]

If you clone a repository, the command automatically adds that remote repository under the name origin. 
So, git fetch origin fetches any new work that has been pushed to that server since you cloned (or last fetched from) it. 
It’s important to note that the fetch command pulls the data to your local repository — it doesn’t automatically merge it with any of your work or modify what you’re currently working on. 
You have to merge it manually into your work when you’re ready.

git pull??
If you have a branch set up to track a remote branch (see the next section and Chapter 3 for more information), you can use the git pull command to automatically fetch and then merge a remote branch into your current branch. This may be an easier or more comfortable workflow for you.

Running git pull generally fetches data from the server you originally cloned from and automatically tries to merge it into the code you’re currently working on.

#### Pushing to Your Remotes

    $ git push [remote-name] [branch-name]

This command works only if you cloned from a server to which you have write access and if nobody has pushed in the meantime. 
If you and someone else clone at the same time and they push upstream and then you push upstream, your push will rightly be rejected. 
You’ll have to pull down their work first and incorporate it into yours before you’ll be allowed to push.

#### Inspecting a Remote

    $ git remote show [remote-name]

#### Removing and Renaming Remotes

    $ git remote rename old-name new-name
    $ git remote rm name


## Tagging

$ git tag

To be continued


## Tips and Tricks

Auto-Completion
Git Aliases

* [Tips and Tricks](http://progit.org/book/ch2-7.html)



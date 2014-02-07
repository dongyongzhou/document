---
layout: master
title: git
---

## Reference

* [Pro Git](http://progit.org/book/)
* [Git Community Book 中文版](http://gitbook.liuhui998.com/index.html)

When need help while using Git, there are three ways to get the manual page (manpage) help for any of the Git commands:
    $ git help <verb>
    $ git <verb> --help
    $ man git-<verb>

##1 Git Basics

###1.1 Git features

- Snapshots, Not Differences
- Nearly Every Operation Is Local
- Git Has Integrity
- Git Generally Only Adds Data
- The Three States:working directory/staging area/git directory

###1.2 Git Environment

####1.2.1 Configurations

- /etc/gitconfig file: Contains values for every user on the system and all their
 repositories. If you pass the option--system to git config, it reads and writes
 from this file specifically.
- ~/.gitconfig file: Specific to your user. You can make Git read and write to this
 file specifically by passing the --global option.
- config file in the git directory (that is, .git/config) of whatever repository you’re
 currently using: Specific to that single repository. Each level overrides values in
 the previous level, so values in .git/config trump those in /etc/gitconfig.

####1.2.2 Set up

- Identity

		$ git config --global user.name "John Doe"
		$ git config --global user.email johndoe@example.com

- Editor
		
		$ git config --global core.editor vi

- Diff Tool

use to resolve
merge conflicts
		
		$ git config --global merge.tool vimdiff

####1.2.3 Check Settings
check settings for all three config levels, and config for specified key value.
		
		$ git config --list
		$ git config user.name

###1.3 Git Help
			
			$ git help <verb>
			$ git <verb> --help
			$ man git-<verb>

##2 Environment Setting

###2.1 Installing Git

####2.1.1 Installing on Linux

    $ apt-get install git-core

####2.1.2 Installing on Windows

Download and install the latest version of [Git for Windows](http://code.google.com/p/msysgit/downloads/list)

    http://code.google.com/p/msysgit

###2.2 Git Setup

Refer to [First-Time Git Setup](http://progit.org/book/ch1-5.html)

It is time to customize Git environment. You should have to do these things only once; they’ll stick around between upgrades. You can also change them at any time by running through the commands again. 

Git comes with a tool called git config that lets you get and set configuration variables that control all aspects of how Git looks and operates. These variables can be stored in three different places:

+ /etc/gitconfig file: Contains values for every user on the system and all their repositories. If you pass the option --system to git config, it reads and writes from this file specifically.
+ ~/.gitconfig file: Specific to your user. You can make Git read and write to this file specifically by passing the --global option.
+ config file in the git directory (that is, .git/config) of whatever repository you’re currently using: Specific to that single repository. Each level overrides values in the previous level, so values in .git/config trump those in /etc/gitconfig.

###2.3 Setup Identity

The first thing you should do when you install Git is to set your user name and e-mail address. 

    $ git config --global user.name "<username>"
    $ git config --global user.email username@example.com

Again, you need to do this only once if you pass the --global option, because then Git will always use that information for anything you do on that system. 
If you want to override this with a different name or e-mail address for specific projects, you can run the command without the --global option when you’re in that project.

Refer for more [First-Time Git Setup](http://progit.org/book/ch1-5.html)

##3 Basic operation

###3.1 Getting a Git Repository

####3.1.1 Initializing a Repository in an Existing Directory

    $ git init
    $ git add .
    $ git commit -m 'initial project version'

####3.1.2 Cloning an Existing Repository

    $ git clone [repository url] [<directory>]

Git has a number of different transfer protocols you can use. 

+ git:// protocol, 
+ http(s):// or user@server:/path.git, which uses the SSH transfer protocol.

###3.2 Checking the Status of the Files

    $ git status

####3.2.1 Tracking New Files

    $ git add [new file]
    $ git status

####3.2.2 Staging New Files

    $ git add [modified file]
    $ git status

If you modify a file after you run git add, you have to run git add again to stage the latest version of the file:

###3.3 Ignoring Files

To ignore the file to be added or statused.

create a file listing patterns to match them named .gitignore.

- Globle Git: ~/.gitignore
- Local Git: .gitignore 
- Git exclude: .git/info/exclude


###3.4 Viewing Your Staged and Unstaged Changes

To see what you’ve changed but not yet staged, type git diff with no other arguments, That command compares what is in your working directory with what is in your staging area. The result tells you the changes you’ve made that you haven’t yet staged.

    $ git diff

If you want to see what you’ve staged that will go into your next commit, you can use git diff --cached. (In Git versions 1.6.1 and later, you can also use git diff --staged, which may be easier to remember.) This command compares your staged changes to your last commit: 

- git diff：是查看working tree与index file的差别的。
- git diff --cached：是查看index file与commit的差别的。
- git diff HEAD：是查看working tree和commit的差别的。（你一定没有忘记，HEAD代表的是最近的一次commit的信息）

###3.5 Committing Your Changes

    $ git commit

Doing so launches your editor of choice. 
You can see that the default commit message contains the latest output of the git status command commented out and one empty line on top. You can remove these comments and type your commit message.

Alternatively, you can type your commit message inline with the commit command by specifying it after a -m flag, like this:

    $ git commit -m "comments"

Providing the -a option to the git commit command makes Git automatically stage every file that is already tracked before doing the commit, letting you skip the git add part:

###3.6 Removing Files

Delete the file from working and staged space:

    $ git rm [file]

Delete the file being modified and staged from working working and staged space:

    $ git rm [file] -f

Delete the file only from staged space:

    $ git rm --cached [file]

You can pass files, directories, and file-glob patterns to the git rm command. That means you can do things such as

    $ git rm log/\*.log

###3.7 Moving Files

Rename a file

    $ git mv file_from file_to


###3.8 Viewing the Commit History

    $ git log


- -p, which shows the diff introduced in each commit.
- -N, which limits the output to only the last N entries:
- --stat,to see some abbreviated stats for each commit
- --pretty. changes the log output to formats other than the default. A few prebuilt options are available for you to use. The oneline option prints
each commit on a single line, which is useful if you’re looking at a lot of commits.
In addition, the short, full, and fuller options show the output in roughly the same
format but with less or more information, respectively:

###3.9 Undoing Things

####3.9.1 Changing Your Last Commit

    $ git commit -m 'initial commit'
    $ git add forgotten_file
    $ git commit --amend

####3.9.2 Unstaging a Staged File

    $ git reset HEAD <file>

####3.9.3 Unmodifying a Modified File

    $ git checkout -- <file>

###3.10 Working with Remotes

**Remote** repositories are versions of your project that are hosted
on the Internet or network somewhere. You can have several of them, each of which
generally is either read-only or read/write for you.


Managing remote repositories includes knowing how to 
1. add remote repositories, 
2. remove remotes that are no longer valid, 
3. manage various remote branches and define them as being tracked or not

####3.10.1 Showing Your Remotes

    $ git remote
origin — that is the default name Git gives to the server you cloned from:

You can also specify -v, which shows you the URL that Git has stored for the shortname to be expanded to:

    $ git remote -v

Notice that only the origin remote with an SSH URL(git@github.com:<who>/<repository>.git) is the one I can push to .

####3.10.2 Adding Remote Repositories

    $ git remote add [shortname] [url]

####3.10.3 Fetching and Pulling from Your Remotes

    $ git fetch [remote-name]

It fetches any new work that has been pushed to that server since you cloned (or last fetched from) it. 
It’s important to note that the fetch command pulls the data to your local repository — it doesn’t automatically merge it with any of your work or modify what you’re currently working on. 
You have to merge it manually into your work when you’re ready.

    $ git pull [remote-name] [branch-name]

It fetches data from the server you originally cloned from and automatically tries to merge it into the code you’re currently working on.

####3.10.4 Pushing to Your Remotes

    $ git push [remote-name] [branch-name]

This command works only if you cloned from a server to which you have write access and if nobody has pushed in the meantime. 
If you and someone else clone at the same time and they push upstream and then you push upstream, your push will rightly be rejected. 
You’ll have to pull down their work first and incorporate it into yours before you’ll be allowed to push.

####3.10.5 Inspecting a Remote

    $ git remote show [remote-name]

####3.10.6 Removing and Renaming Remotes

    $ git remote rename old-name new-name
    $ git remote rm name

###3.11 git format patch

Generate patch file for the commits.

Base usage:

    $ git format-patch -1 -o ./..

Generate a patch for the last commit.

###3.12 git am patch

merge patch file into the source code.

    $ cd source-code
    $ git-am xxx.patch

###3.13 Tagging

Git has the ability to tag specific points in history as being important.
Generally, people use this functionality to mark release points (v1.0, and so on).

####3.13.1 List tags

$ git tag

####3.13.2 Creating Tag

Git uses two main types of tags: lightweight and annotated. 

- A lightweight tag is very much like a branch that doesn’t change — it’s just a pointer to a specific commit.
- Annotated tags, however, are stored as full objects in the Git database. They’re checksummed;
contain the tagger name, e-mail, and date; have a tagging message; and can
 be signed and verified with GNU Privacy Guard (GPG). 

It’s generally recommended
that you create annotated tags so you can have all this information; but if you want a
temporary tag or for some reason don’t want to keep the other information, lightweight
tags are available too.

####3.13.3 Annotated Tags
		
		$ git tag -a tagname -m "tag message"
		$ git tag
		$ git show tagname
####3.13.4 Signed Tags
		
		$ git tag -s tagname -m "tag message"
		$ git tag
		$ git show tagname

####3.13.5 Lightweight Tags
		
		$ git tag v1.4-lw
		$ git tag

####3.13.6 Verifying Tags
To verify a signed tag, you use 

		$ git tag -v [tag-name]

####3.13.7 Tagging Later

Now, suppose you forgot to tag the project at tagname, You can add it after the fact. To tag that commit, you specify the
commit checksum (or part of it) at the end of the command:
		
		$ git tag -a tagname commit-checksum

####3.13.8 Sharing Tags

By default, the git push command doesn’t transfer tags to remote servers. You will
have to explicitly push tags to a shared server after you have created them. This process
is just like sharing remote branches you can run 
		
		$ git push origin [tagname]



If you have a lot of tags that you want to push up at once, you can also use the
--tags option to the git push command.
		
		$ git push origin --tags

###3.14 Tips and Tricks


####3.14.1 Auto-Completion

If you use the Bash shell, Git comes with a nice auto-completion script you can enable.
Download the Git source code, and look in the contrib/completion directory; there
should be a file called git-completion.bash. Copy this file to your home directory,
and add this to your .bashrc file:

source ˜/.git-completion.bash

If you want to set up Git to automatically have Bash shell completion for all users,
copy this script to the /opt/local/etc/bash completion.d directory on Mac systems
or to the /etc/bash completion.d/ directory on Linux systems. This is a directory of
scripts that Bash will automatically load to provide shell completions.
If you’re usingWindows with Git Bash, which is the default when installing Git on
Windows with msysGit, auto-completion should be preconfigured.

Press the Tab key when you’re writing a Git command, and it should return a set of
suggestions for you to pick from:

####3.14.2 Git Aliases

Git doesn’t infer your command if you type it in partially. If you don’t want to type
the entire text of each of the Git commands, you can easily set up an alias for each
command using git config. Here are a couple of examples you may want to set up:
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status


* [Tips and Tricks](http://progit.org/book/ch2-7.html)



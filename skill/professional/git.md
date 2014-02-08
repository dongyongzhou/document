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

###1.0 Git Objects

Object parts: type size content

type: blob tree commit tag

####1.0.1 Blob

content: store file data.

		$ git show 6ff87c4664

####1.0.2 tree
content: a bunch of pointers pointing to blob or tree 

每一个条目包括：mode、对象类型、SHA1值 和名字(这串条目是
按名字排序的)。它用来表示一个目录树的内容。

		$ git ls-tree fb3a8bdd0ce
####1.0.3 commit
pointing to a tree object

		$ git show -s --pretty=raw 2be7fcb476

- 一个tree　对象: tree对象的SHA1签名, 代表着目录在某一时间点的内容.
- 父对象(parent(s)): 提交(commit)的SHA1签名代表着当前提交前一步的项目历史. 上面的那个例子就只有一
个父对象; 合并的提交(merge commits)可能会有不只一个父对象. 如果一个提交没有父对象, 那么我们就叫
 它“根提交"(root commit), 它就代表着项目最初的一个版本(revision). 每个项目必须有至少有一个“根提
 交"(root commit). 一个项目可能有多个"根提交“，虽然这并不常见(这不是好的作法).
- 作者: 做了此次修改的人的名字,　还有修改日期.
- 提交者（committer): 实际创建提交(commit)的人的名字, 同时也带有提交日期. TA可能会和作者不是同一
个人; 例如作者写一个补丁(patch)并把它用邮件发给提交者, 由他来创建提交(commit).

####1.0.4 tag
一个标签对象包括一个对象名(译者注:就是SHA1签名), 对象类型, 标签名, 标签创建人的名字("tagger"), 还有一条可
能包含有签名(signature)的消息. 你可以用git cat-file 命令来查看这些信息

		$ git cat-file tag v1.5.0
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

##4 Branching

diverge from the main line of development and continue to do work without messing with that
main line.

Because a branch in Git is in actuality a simple file that contains the 40 character
SHA–1 checksum of the commit it points to, branches are cheap to create and destroy.
Creating a new branch is as quick and simple as writing 41 bytes to a file (40 characters
and a newline).

###4.0 git branch

The git branch command does more than just create and delete branches. If you
run it with no arguments, you get a simple listing of your current branches:

		$ git branch
		* master
		  test
		  test2

To see the last commit on each
branch, you can run git branch v:

		$ git branch -v
		* master 4bf2877 branch3 commit 2
		  test   de9f098 branch test
		  test2  67bb1ef branch test2


To see which branches are already merged into the branch you’re
on, you can run git branch merged:
		
		$ git branch --merged

To see all the branches that contain work you haven’t yet merged in, you can run
		
		git branch --no-merged

Because it contains work that isn’t merged in yet,
trying to delete it with git branch -d will fail

###4.1 git branch branchname

This creates a new pointer at the same commit you’re currently on. The git branch command
only created a new branch— it didn’t switch to that branch

.git/HEAD keeps a pointer to the local
branch you’re currently on.

###4.2 git checkout branchname

To switch to an existing branch, run the git checkout command.
This moves HEAD to point to the branchname

git checkout -b branchname == git branch branchname + git checkout branchname

**note** that if your working directory or staging area has
uncommitted changes that conflict with the branch you’re checking out, Git won’t let
you switch branches.

Git resets your working directory to look like the snapshot
of the commit that the branch you check out points to. It adds, removes, and modifies
files automatically to make sure your working copy is what the branch looked like on
your last commit to it.

###4.2 git merge branchname

Once you have complete the working and committing in branchname.

All you have to do is check out the
branch you wish to merge into and then run the git merge command:

If the commit on the
branch you’re on isn’t a direct ancestor of the branch you’re merging in,

Git does a simple three-way merge, using the two snapshots
pointed to by the branch tips and the common ancestor of the two.

Git automatically creates a new commit object that contains the merged
work.

####4.2.1 Merge conflicts

If you changed the same part of the
same file differently in the two branches you’re merging together, Git won’t be able to
merge them cleanly.

		$ git merge test2
		Auto-merging test.c
		CONFLICT (content): Merge conflict in test.c
		Automatic merge failed; fix conflicts and then commit the result.

Git hasn’t automatically created a new merge commit. It has paused the process
while you resolve the conflict. If you want to see which files are unmerged at any point
after a merge conflict, you can run git status:

		$ git status
		# On branch master
		# You have unmerged paths.
		#   (fix conflicts and run "git commit")
		#
		# Unmerged paths:
		#   (use "git add <file>..." to mark resolution)
		#
		#       both modified:      test.c
		#
		no changes added to commit (use "git add" and/or "git commit -a")

Anything that has merge conflicts and hasn’t been resolved is listed as unmerged.
Git adds standard conflict-resolution markers to the files that have conflicts, so you can
open them manually and resolve those conflicts.

		<<<<<<< HEAD
		        printf("End\n");
		=======
		>>>>>>> test2
				printf("TBD\n");


In order to resolve the conflict, you have to either choose
one side or the other or merge the contents yourself.

This resolution has a little of each section, and I’ve fully removed the <<<<<<<,
=======, and >>>>>>> lines. After you’ve resolved each of these sections in each conflicted
file, run** git add** on each file to mark it as resolved. Staging the file marks it as
resolved in Git. If you want to use a graphical tool to resolve these issues, you can run
git mergetool, which fires up an appropriate visual merge tool and walks you through
the conflicts:

verify that everything that had conflicts has been
staged, you can type **git commit **to finalize the merge commit. The commit message
by default looks something like this:


####4.3 git branch -d branchname

Once the branchname is not used anymore, Delete it.

####4.4 Remote branches

Remote branches are references to the state of branches on your remote repositories.

Remote branches act as bookmarks to remind you where the branches on your remote repositories were the last time you connected to them.


##### 4.4.1 Remote

master branch locally

##### 4.4.2 Local

master branch, locally

origin/master, remote branch status, saving the last remote master based on the last communication. maybe out of date.

#####4.4.3 git fetch remotename

synchronize work with remote
		
		git fetch remotename [branchname]

This command looks up which server remotename is, fetches any
data from it that you don’t yet have, and updates your local database, moving your
origin/master pointer to its new, more up-to-date position.

To merge it into the local master branch
		
		git checkout master
		git merge remotename/branchname

If have fetched a new branch, to make it local but not merge into current branch.
		
		$ git checkout -b serverfix origin/serverfix

#####4.4.4 git remote add


#####4.4.5 git push

have to explicitly push the branches you want to share.
		
		
		git push remotename branchname
		or git push [remotename] [localbranch]:[remotebranch]

or push a local branch into a remote branch that is named differently

		or git push origin branchname:branchname2


Git automatically expands the branchname
out to refs/heads/branchname:refs/heads/branchname,

		git push remotename refs/heads/branchname:refs/heads/branchname

NOTE: What about pushing the commits to review web site??


#####4.4.6 tracking branch

		git checkout -b [branch] [remotename]/[branch].

The first branch could be different from the last one.

Git version 1.6.2 or later, you can also use the --track shorthand:

		$ git checkout --track origin/serverfix

Checking out a local branch from a remote branch automatically creates what is called
a tracking branch. 

Tracking branches are local branches that have a direct relationship
to a remote branch. 

If you’re on a tracking branch and type git push, Git automatically
knows which server and branch to push to. Also, running git pull while on one of these branches fetches all the remote references and then automatically merges in the
corresponding remote branch.

#####4.4.7 Deleting Remote Branches


		git push [remotename] :[branch]


###4.5 Rebase

In Git, there are two main ways to integrate changes from one branch into another: the
merge and the rebase

The easiest way to integrate the branches is the merge
command. It performs a three-way merge between the two latest branch snapshots and the most recent common ancestor of the two, creating a new snapshot
(and commit). 

Rebase way is taking the patch of the change that was
introduced in other branch and reapply it on top of current commit of the current branch.

merge will keeps all the commits of the other branch.

With the
rebase command, you can take all the changes that were committed on one branch and
replay them on another one.
		
		$ git checkout experiment
		$ git rebase master: it will apply the commit as a patch to the master branch one by one.
		$ git checkout master: go back to the master branch and do a fast-forward merge
		$ git merge experiment

Difference between merge and rebase

it’s only the history that is different. Rebasing replays changes
from one line of work onto another in the order they were introduced, whereas merging
takes the endpoints and merges them together.



**common one:**

		git rebase [basebranch] [topicbranch] 

 which checks out the topic branch (in
this case, server) for you and replays it onto the base branch (master):


**Conplex one:**

		$ git checkout client
		$ git rebase --onto master server client
		$ git checkout master
		$ git merge client

This basically says, “Check out the client branch, figure out the patches from
the common ancestor of the client and server branches, and then replay them onto
master.”

**Note:**

if you only rebase commits that have never been available publicly, then
you’ll be fine. If you rebase commits that have already been pushed publicly, and
people may have based work on those commits, then you may be in for some frustrating
trouble.
69

## Reference

* [Tips and Tricks](http://progit.org/book/ch2-7.html)



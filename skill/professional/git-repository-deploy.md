---
layout: master
title: git
---

##1 Reference

* [Pro Git](http://progit.org/book/)
* [Git Community Book 中文版](http://gitbook.liuhui998.com/index.html)
* [搭建git服务器过程  ](http://xxw8393.blog.163.com/blog/static/3725683420105224424689/)

##2 工具安装

软件需求：git-core, gitosis, openssh-server, openssh-client

安装git和openssh：

	$ sudo apt-get install git-core
	$ sudo apt-get install openssh-server
	$ sudo apt-get install openssh-client


##3 环境设置

新加用户git，该用户将作为所有代码仓库和用户权限的管理者：

	$ sudo useradd -m git

为git设置密码：

	$ sudo passwd git

建立一个git仓库的存储点，我放在了/home/prj_git下，并且让除了git以外的用户对此目录无任何权限：

	$ sudo mkdir /home/prj_git
	$ sudo chown git:git /home/prj_git
	$ sudo chmod 700 /home/prj_git


初始化一下服务器的git用户，这一步其实是为了安装gitosis做准备，当然在任何一台机器上使用git，第一次必须要初始化一下，git向来不搞“知名不具”那一套：

	$ git config --global user.name "ly44770"
	$ git config --global user.email " ly44770@163.com "

安装一下python的setup tool， 这个也是为了gitosis做准备：

	$ sudo apt-get install python-setuptools

获得gitosis包：

	$ cd /tmp
	$ git clone git://eagain.net/gitosis.git
	失败时：git clone https://github.com/res0nat0r/gitosis.git
	$ cd gitosis
	$ sudo python setup.py install



切换到git用户下

---------------------------

	$ su git

默认状态下，gitosis会将git仓库放在git用户的home下，所以我们做一个链接到/home/prj_git

	$ ln -s /home/prj_git /home/git/repositories
再次返回到默认用户

	$ exit

##4 配置gitosis

假如你将作为git服务器的管理员，那么在你的电脑上(另一台pc）生成ssh公钥：

	$ ssh-keygen -t rsa

将公钥拷贝到服务器的/tmp下，并给其他人以读权限：

	$ scp .ssh/id_rsa.pub git@192.168.1.39:/tmp


	$ sudo chmod a+r /tmp/id_rsa.pub

让gitosis运行起来：

	/tmp/gitosis$ sudo -H -u git gitosis-init < /tmp/id_rsa.pub 

	Initialized empty Git repository in /home/prj_git/gitosis-admin.git/
	Reinitialized existing Git repository in /home/prj_git/gitosis-admin.git/

gitosis的有趣之处在于，它通过一个git仓库来管理配置文件，仓库就放在了/home/prj_git/gitosis-admin.git。我们需要为一个文件加上可执行权限：

	$ sudo passwd root
	$ su
	/home/git# cd repositories
	/home/git/repositories# cd gitosis-admin.git/
	/home/git/repositories/gitosis-admin.git# sudo chmod 755 /home/prj_git/gitosis-admin.git/hooks/post-update
	/home/git/repositories/gitosis-admin.git# exit



在你自己的电脑里，把gitosis-admin.git这个仓库clone下来，这样你就可以以管理员的身份修改配置了。

在你的电脑里：

	$ git clone git@192.168.1.39:gitosis-admin.git

	$ cd gitosis-admin/

为了测试添加一个新用户：

	~/work/gitosis-admin$ sudo useradd -m b
	~/work/gitosis-admin$ sudo passwd b

现在把你们team所有人的ssh公钥文件都拿来，按名字命名一下，比如b.pub, lz.pub等，统统拷贝到keydir下：

	~/work/gitosis-admin$ su root
	/home/a/work/gitosis-admin# cp /home/b/.ssh/id_rsa.pub ./keydir/b.pub
	~/work/gitosis-admin$ cp /tmp/lz.pub ./keydir/
	/home/a/work/gitosis-admin# exit

修改gitosis.conf文件，我的配置大致如下：

	[gitosis]
	[group gitosis-admin]
	writable = gitosis-admin
	members = a@ubuntu
	[group hello]
	writable = teamwork
	members = a@ubuntu b
	[group hello_ro]
	readonly = teamwork
	members = lz

这个配置文件表达了如下含义：gitosis-admin组成员有a，该组对gitosis-admin仓库有读写权限；

team组有a，b两个成员，该组对teamwork仓库有读写权限； 

team_ro组有lz一个成员，对teamwork仓库有只读权限。


当然目前这些配置文件的修改只是在你的本地，你必须推送到远程的gitserver上才能真正生效。

加入新文件、提交并push到git服务器：

	~/work/gitosis-admin$ git add .
	~/work/gitosis-admin$ git commit -am "add teamweok prj and users"
	~/work/gitosis-admin$ git push origin master




##4 新建项目仓库


在服务器上新建一个空的项目仓库，我建了一个叫“teamwork”的仓库。

切换到git用户：

	a@ubuntu:/home/git$ su - git
	$ cd /home/prj_git
	$ mkdir teamwork.git
	$ cd teamwork.git
	$ git init --bare
	$ exit



现在服务器就搭建完了，并且有一个空的项目teamwork在服务器上。接下来呢？当然是测试一下，空仓库是不能clone的，所以需要某一个有写权限的人初始化一个版本。就我来做吧，以下是在客户端完成。

	a@ubuntu:~/work$ mkdir teamwork-ori
	a@ubuntu:~/work$ cd teamwork-ori/
	a@ubuntu:~/work/teamwork-ori$ git init
	a@ubuntu:~/work/teamwork-ori$ echo "/*add something*/" > hello
	a@ubuntu:~/work/teamwork-ori$ git add .
	a@ubuntu:~/work/teamwork-ori$ git commit -am "initial version"
	a@ubuntu:~/work/teamwork-ori$ git remote add origin git@192.168.1.39:teamwork.git
	a@ubuntu:~/work/teamwork-ori$ git push origin master

到此为止teamwork已经有了一个版本了，team的其他成员只要先clone一下teamwork仓库，就可以任意玩了。

	a@ubuntu:~/work/teamwork-ori$ su b
	$ cd /home/b
	$ git clone git@192.168.1.39:teamwork.git
	$ cd teamwork
	$ vim hello
	$ git add .
	$ git commit -am "b add"
	$ git push origin master 
	$ exit

 
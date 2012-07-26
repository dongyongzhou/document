---
layout: master
title: ctags operation
---

##1 Overview

 tag记录了关于一个标识符在哪里被定义的信息，比如C或C++程序中的一个函数定义。这种tag聚集在一起被放入一个tags文 件。这个文件可以让Vim能够从任何位置起跳达到tag所指示的位置－标识符被定义的位置。

##2 Set up

###2.1 install ctags

    $ sudo apt-get install ctags


###2.2 生成tags文件

    $ ctags -R *

**为当前目录下的所有C程序文件生成对应的tags文件**：

    $ ctags *.c

**为同一个目录下的所有文件建立tags如下**

    $ ctags -R
    这个命令会给当前目录及子目录下的所有文件建立tags

###2.3 设置tags搜索路径

**设置一个源码tags，使用绝对路径**

    (ex command) : set tags=/dir/tags
    或者在~/.vimrc中添加设置

**要使用更多tags文件，可以通过改变'tags'选项的设置来引入更多的tags文件。**

    (ex command) :set tags=./tags, ./../tags, .tags
    或者在~/.vimrc中添加设置

**设置vim中tags搜索目录 **

    set tags=tags;当前目录。 
    set autochdir向上层搜索 

    比较适合有多个不同源码（例如linux2.4，linux2.6）需要同时看，互不干扰。 
    可以只设置一个绝对路径

##3 Usage

###3.1 SHELL命令行下：

    vim -t <name>

到源码目录下（如果设置绝对路径则在任何地方都起作用），vim -t <name> ，则会打开<name> 所在的文件，焦点落在<name> 所在行。

###3.2 vim文件上：

    (ex command) :ta <name>
    (ex command) :tag <name>

在Vim中会跳到一个函数的定义<name>处

:tags命令会列出现在你就已经到过哪些tag了

    (ex command):tags

有多个tags满足条件，用：tnext和：tprev在满足条件的定义中游荡。也可以用tselect列出所有 
满足条件的文件。 

在vim中，：tselect <name> 可以列出所有符合条件的文件，选择标号则打开不同文件。 

CTRL+] 命 令会取当前光标下的word作为tag的名字并直接跳转。这使得在大量C程序中进行探索更容易一些。假设你正看函数"write block"，发现它调用了一个叫"write line"的函数，这个函数是干什么的呢？你可以把光标置于"write_line"上，按下CTRL+] 即可。如果"write_line"函数又调用了 "write_ char".你当然又要知道这个函数又是什么功能。同时，置光标于"write_char"上按下CTRL+]。现在你位于函数"write_char" 的定义处。

CTRL+T 命 令会跳到你前一次的tag处。在上例中它会带你到调用了"write_char"的"write_line"函数的地方。CTRL+T可以带一个命令记 数, 以此作为往回跳的次数, 你已经向前跳过了，现在正在往回跳，我们再往前跳一次。
下面的命令可以直接跳转到当前tag序列的最后：

(ex command) :tag
也可以给它一个前辍, 让它向前跳指定的步长. 比如":3tag "。 CTRL+T也可以带一个前辍。这些命令可以让你向下深入一个函数调用树(使用CTRL+]), 也可以回溯跳转(使用CTRL+T). 还可以随时用":tags"看你当前的跳转历史记录。

":tag"命令会在当前窗口中载入包含了目标函数定义的文件。但假设你不仅要查看新的函数定义，还要同时保留当前的上下文呢？你可以 在":tag"后使 用一个分隔窗口命令":split"。Vim还有一个一举两得的命令：(ex command) :stag tagname
要分隔当前窗口并跳转到光标下的tag：

(normal mode command) CTRL+W+] 
如果同时还指定了一个命令记数, 它会被当作新开窗口的行高. 当一个函数被多次重载(或者几个类里都定义了一些同名的函 数)，":tag"命令会跳转到第一个符合条件的。如果当前文件中就有一个匹配的，那又会优先使用它。当然还得有办法跳转到其它符合条件的tag去：

(ex command) :tnext 
重复使用这个命令可以发现其余的同名tag。如果实在太多，还可以用下面的命令从中直接选取一个：

(ex command) :tselect tagname 
Vim会提供给你一个选择列表，例如：(Display)现在你只需键入相应的数字(位于第一栏的)。 其它栏中的信息是为了帮你作出决策的。在多个匹配的tag之间移动，可以使用下面这些命令：

    (ex command):tfirst              go to first match
    :[count]tprevious    go to [count] previous match
    :[count]tnext        go to [count] next match
    :tlast               go to last match

如果没有指定[count]，默认是1。

命令补齐真是避免键入一个长tag名的好办法。只要输入开头的几个字 符然后 按下制表符：

(ex command) :tag write_<Tab> 

Vim 会为你补全第一个符合的tag名。如果还不合你意，接着按制表符直到找到你要的。有时候你只记得一个tag名的片段，或者有几个tag开头相同。这里你可 以用一个模式匹配来告诉Vim你要找的tag。

假设你想跳转到一个包含"block"的tag。首先键入命令：(ex command) :tag /block。现在使用命令补齐：按<Tab>。Vim会找到所有包含"block"的tag并先提供给你第一个符合的。"/"告诉Vim下 面的名字不是一五一十的tag名，而是一个搜索模式。通常的搜索技巧都可以用在这里。比如你有一个tag以"write "开始：(ex command) :tselect / ^write_，"^"表示这个tag以"write_"开始。不然在半中间出现write的tag也会被搜索到。同样"$"可以用于告诉Vim要查找的 tag如何结束。

CTRL+] 可以直接跳转到以当前光标下的word为tag名的地方去，所以可以在一个tag列表中使 用它。下面是一个例子。首先建立一个标识符的列表(这需要一个好的ctags)：

    (shell command) ctags --c-types=f -f functions *.c 

现在直接启动Vim, 以一个垂直分隔窗口的编辑命令打开生成的文件：

    (shell command) vim:vsplit functions 

这个窗口中包含所有函数名的列表。可能会有很多内容，但是你可以暂时忽略它。用一个":setlocal ts=99"命令清理一下显示。在该窗口中，定义这样的一个映射：

    (ex command):nnoremap <buffer> <CR> 0ye<C-W>w:tag <C-R>"<CR>

现在把光标移到你想要查看其定义的函数名上，按下回车键，Vim就会在另一个窗口中打开相应的文件并定位到到该函数的定义上。

基本命令：

在vim中ctags的简单使用

- 1) 跳转到指定的函数进入vim后，用 “:tag func_name“ 跳到函数func_name处。使用tag
命令时，可以使用TAB键进行匹配查找，继续按TAB键向下切换。
某个函数有多个定义时

:tag 
跳到第一个定义处，优先跳转到当前文件

:tnext
跳到第一个

:tfirst
跳到前count个

:[count]tprevious
跳到后count个

:[count]tnext
跳到最后一个

:tlast
你也可以在所有tagname中选择：

:tselect tagname

如果想跳到包含block的标识符:“tag /block” 然后用TAB键来选择。这里'/'就是告诉vim 
'block'是一个语句块标签。

- 2)用“CTRL + ]“快捷键，跳转到光标所在函数标识符的定义处。
- 3)使用“CTRL + T”退回上层。如果想在以write_开头的标识符中选择一下， :tselect /^
write_ 这里，'^'表示开头，同理，'$'表示末尾。多个同名的标识符

## Reference


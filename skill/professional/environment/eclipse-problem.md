---
layout: master
title: Eclipse Problems
---

##问题及解决

###1 Workspace in use or cannot be created, choose a different one.

**Description**:

打开 Eclipse 提示：

Workspace in use or cannot be created, choose a different one.

**Solution**:

A:出现这种情况一般是workspace的配置文件中出现了.lock文件（workspace/.metadata/.lock），锁定了workspace。把.lock文件删除即可。

如果该文件不能删除，可能是因为javaw.exe进程未结束，结束该进程及eclipse.exe进程即可删除。

正常情况下，如果你打开了一个workspace,在想打开另一个workspace也会出现上面的提示。

B:eclipse\configuration\.settings目录下有个文件名为org.eclipse.ui.ide的PREFS 类型文件，用记事本打开，里面（最后一行）有个叫RECENT_WORKSPACES的键，这个键值保存着所有最近打开的目录信息，默认是用\n来换行的。把键值全删掉，保存，重新打开eclipse就行了。

	
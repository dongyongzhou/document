---
layout: master
title: Android app permission handling mechanism
---

# 1. Android 安全机制

- 基于Linux已有的权限分离的权限管理机制。

	通过为每一个 Application 分配不同的 uid 和 gid ， 从而使得不同的 Application 之间的私有数据和访问（ native 以及 java 层通过这种 sandbox 机制，都可以）达到隔离的目的 。

- Android permission 机制。

	它主要是用来对 Application 可以执行的某些具体操作进行权限细分和访问控制，同时提供了 per-URI permission 机制，用来提供对某些特定的数据块进行 ad-hoc 方式的访问。


# 2. 基于Linux已有的权限分离的权限管理机制

## 2.1 Android 的权限分离的基础是建立在 Linux 已有的 uid 、 gid 、 gids 基础上的

- UID 

	Android 在 安装一个应用程序，就会为 它 分配一个 uid （参考 PackageManagerService 中的 newUserLP 实现）。其中普通 A ndroid 应用程序的 uid 是从 10000 开始分配 （参见 Process.FIRST_APPLICATION_UID ）， 10000 以下是系统进程的 uid 。

- GID

	对 于普通应用程序来说， gid 等于 uid 。由于每个应用程序的 uid 和 gid 都不相同， 因此不管是 native 层还是 java 层都能够达到保护私有数据的作用 。

- GIDS

	gids 是由框架在 Application 安装过程中生成，与 Application 申请的具体权限相关。 如果 Application 申请的相应的 permission 被 granted ，而且中有对应的 gids ， 那么 这个 Application 的 gids 中将 包含这个 gids 。

##2.2 uid gid gids 的 详细 设置过程

	请参考 ActivityManagerService 中的 startProcessLocked 。

	在通过 zygote 来启动一个 process 时，直接将 uid 传给 给了 gid。

	再通过 zygote 来 fork 出新的进程（ zygote.java 中的 forkAndSpecialize），最终在 native 层（ dalvik_system_zygote.c ）中的 forkAndSpecializeCommon 中通过 linux 系统调用来进行 gid 和 uid 和 gids 的设置。


# 3. Android permission 机制

## 3.1 Permission

一个权限主要包含三个方面的信息：权限的名称；属于的权限组；保护级别。

- 权限的名称
	
	android.permission.SET_TIME
	
- 一个权限组是指把权限按照功能分成的不同的集合。

	每一个权限组包含若干具体权限，例如在 COST_MONEY 组中包含 android.permission.SEND_SMS ， android.permission.CALL_PHONE 等和费用相关的权限。

- 每个权限通过 protectionLevel 来标识保护级别： normal ， dangerous ， signature ， signatureorsystem 。

	不同的保护级别代表了程序要使用此权限时的认证方式。 

	normal 的权限只要申请了就可以使用； 
	
	dangerous 的权限在安装时需要用户确认才可以使用； 

	signature 和 signatureorsystem 的权限需要使用者的 app 和系统使用同一个数字证书。

## 3.2 permission 定义


Package 的权限信息主要 通过在 AndroidManifest.xml 中通过一些标签来指定。如 

	<permission> 标签， 
	<permission-group> 标签 
	<permission-tree> 等标签。

	/frameworks/base/core/res/AndroidManifest.xml
    <!-- Group of permissions that are related to system clock. -->
    <permission-group android:name="android.permission-group.SYSTEM_CLOCK"
        android:label="@string/permgrouplab_systemClock"
        android:icon="@drawable/perm_group_system_clock"
        android:description="@string/permgroupdesc_systemClock"
        android:priority="140" />

    <!-- Allows applications to set the system time.
    <p>Not for use by third-party applications. -->
    <permission android:name="android.permission.SET_TIME"
        android:protectionLevel="signature|system"
        android:label="@string/permlab_setTime"
        android:description="@string/permdesc_setTime" />

    <!-- Allows applications to set the system time zone -->
    <permission android:name="android.permission.SET_TIME_ZONE"
        android:permissionGroup="android.permission-group.SYSTEM_CLOCK"
        android:protectionLevel="normal"
        android:label="@string/permlab_setTimeZone"
        android:description="@string/permdesc_setTimeZone" />


如果 package 需要申请使用某个权限，那么需要使用 <use-permission> 标签来指定。

    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

##3.3 permission 的初始化

permission 的初始化，是指 permission 的向系统申请，系统进行检测并授权，并建立相应的数据结构。绝大多数的情况下 permission 都是从一个 package 中扫描所得，而这发生在 package **安装**和**升级**的时候。

一般有如下几种安装入口：

- packageInstaller ， package 被下载安装时会触发使用。 packageInstaller 会通过 AppSecurityPermissions 来检查 dangerous 的权限，并对用户给出提示。

- pm 命令 。

- adb install 。最终还是 调用 pm install 来安装 apk 包。

- 拷贝即安装。 PackageManagerService 中使用 AppDirObserver 对 /data/app/ 进行监视 ，如果有拷贝即触发安装。

这些安装方式 最终都会通过调用 PackageManagerService 中的函数来完成程序的安装。

##3.4 permission 创建

###3.4.1 从 AndroidManifest.xml 中提取 permission 信息

主要提取如下信息：

- shared uid 
	
	指定与其它 package 共享同一个 uid 。

- permission

	提取 permissions 标签指定属性。它使用 permissionInfo 来描述一个权限的基本信息。需要指定 protectedLevel 信息，并指定所属 group 信息。它将被添加到这个 package 的 permissions 这个 list 结构中。

- permission-tree

	提取 permissions-tree 标签属性。 permissions-tree 也通过 permissionInfo 来描述，并被添加到 package 的 permissions 这个 list 结构中。 permission-tree 只是一个名字空间，用来向其中动态添加一些所谓 Dynamic 的 permission ，这些 permission 可以动态修改。这些 permission 名称要以 permission-tree 的名称开头。它本身不是一种权限，没有 protectedLevel 和所属 group 。只是保存了所属的 packge 和权限名（带有 package 前缀的）。

- permission-group

	定义 permission 组信息，用 PermissionGroup 表示。本身不代表一个权限，会添加进入 package 的 permissionGroups 这个 list 中。

- uses-permission

	定义了 package 需要申请的权限名。将权限名添加到 package 的 requestedPermissions 这个 list 中。

- adopt-permissions

	将该标签指定的 name 存入 package 的 mAdoptPermissions 这个 list 中。 Name 指定了这个 package 需要从 name 指定的 package 进行权限领养。在 system package 进行升级时使用。


###3.4.2 获取 Package 中的证书，验证，并将签名信息保存在 Package 结构中


- 1. 如果该 package 来自 system img （系统 app ），

	那么只需要从该 Package 的 AndroidManifest.xml 中获取签名信息，而无需验证其完整性。但是如果这个 package 与其它 package 共享一个 uid ，那么这个共享 uid 对应的 sharedUser 中保存的签名与之不一致，那么签名验证失败。


- 2. 如果是普通的 package

	那么需要提取证书和签名信息，并对文件的完成性进行验证。


第三步。如果是普通的 package ，那么清除 package 的 mAdoptPermissions 字段信息（系统 package 升级才使用）。

第四步。如果在 AndroidManifest.xml 中指定了 shared user ，那么先查看全局 list 中（ mSharedUsers ）是否该 uid 对应的 SharedUserSetting 数据结构，若没有则新分配一个 uid ，创建 SharedUserSetting 并保存到全局全局 list （ mSharedUsers ）中。

mUserIds 保存了系统中已经分配的 uid 对应的 SharedUserSetting 结构。每次分配时总是从第一个开始轮询，找到第一个空闲的位置 i ，然后加上 FIRST_APPLICATION_UID 即可。

第五步。创建 PackageSettings 数据结构。并将 PackageSettings 与 SharedUserSetting 进行绑定。其中 PackageSettings 保存了 SharedUserSetting 结构；而 SharedUserSetting 中会使用 PackageSettings 中的签名信息填充自己内部的签名信息，并将 PackageSettings 添加到一个队列中，表示 PackageSettings 为其中的共享者之一。

在创建时，首先会以 packageName 去全局数据结构 mPackages 中查询是否已经有对应的 PackageSettings 数据结构存在。如果已经存在 PackageSettings 数据结构（比如这个 package 已经被uninstall ，但是还没有删除数据，此时 package 结构已经被释放）。那么比较该 package 中的签名信息（从 AndroidManifest 中扫描得到）与 PackageSettings 中的签名信息是否匹配。如果不匹配但是为 system package ，那么信任此 package ，并将 package 中的签名信息更新到已有的 PackageSettings 中去，同时如果这个 package 与其它 package 共享了 uid ，而且 shared uid 中保存的签名信息与当前 package 不符，那么签名也验证失败。

第六步。如果 mAdoptPermissions 字段不为空，那么处理 permission 的领养（从指定的 package 对应的 PackageSettings 中，将权限的拥有者修改为当前 package ，一般在 system app 升级的时候才发生，在此之前需要验证当被领养的 package 已经被卸载，即检查 package 数据结构是否存在）。

第七步。添加自定义权限。将 package 中定义的 permissionGroup 添加到全局的列表 mPermissionGroups 中去；将 package 中定义的 permissions 添加到全局的列表中去（如果是 permission-tree 类型，那么添加到 mSettings.mPermissionTrees ，如果是一般的 permission 添加到 mSettings.mPermissions 中）。

第八步。清除不一致的 permission 信息。

1. 清除不一致的 permission-tree 信息。如果该 permission-tree 的 packageSettings 字段为空，说明还未对该 package 进行过解析（若代码执行到此处时 packageSettings 肯定已经被创建过），将其remove 掉。如果 packageSettings 不为空，但是对应的 package 数据结构为空（说明该 package 已经被卸载，但数据还有保留），或者 package 数据结构中根本不含有这个 permission-tree ，那么将这个 permission-tree 清除。

2. 清除不一致的 permission 信息。如果 packageSettings 或者 package 结构为空（未解析该 package 或者被卸载，但数据有保留），或者 package 中根本没有定义该 permission ，那么将该 permission清除。

第九步。对每一个 package 进行轮询，并进行 permission 授权。

1. 对申请的权限进行检查，并更新 grantedPermissions 列表

2. 如果其没有设置 shared user id ，那么将其 gids 初始化为 mGlobalGids ，它从 permission.xml 中读取。

3. 遍历所有申请的权限，进行如下检查

1 ）如果是该权限是 normal 或者 dangerous 的。通过检查。

2 ）如果权限需要签名验证。如果签名验证通过。还需要进行如下检查

* 如果程序升级，而且是 system package 。那么是否授予该权限要看原来的 package 是否被授予了该权限。如果被授予了，那么通过检查，否则不通过。

* 如果是新安装的。那么检查通过。

4. 如果 3 中检查通过，那么将这个 permission 添加到 package 的 grantedPermissions 列表中，表示这个 permission 申请成功（ granted ）。申请成功的同时会将这个申请到的 permission 的 gids 添加到这个 package 的 gids 中去。

5. 将 permissionsFixed 字段标准为 ture ，表示这个 packge 的 permission 进行过修正。后续将禁止对非 system 的 app 的权限进行再次修正。

# APP side

# framework side

## defination for permissions ##

    /frameworks/base/api/current.txt
    115    field public static final java.lang.String SET_TIME = "android.permission.SET_TIME";
    116    field public static final java.lang.String SET_TIME_ZONE = "android.permission.SET_TIME_ZONE";

## processing in framework


    /frameworks/base/services/java/com/android/server/AlarmManagerService.java
    public void setTime(long millis) {
        mContext.enforceCallingOrSelfPermission(
                "android.permission.SET_TIME",
                "setTime");

        SystemClock.setCurrentTimeMillis(millis);
    }

    public void setTimeZone(String tz) {
        mContext.enforceCallingOrSelfPermission(
                "android.permission.SET_TIME_ZONE",
                "setTimeZone");

        long oldId = Binder.clearCallingIdentity();


### enforceCallingOrSelfPermission

	/frameworks/base/core/java/android/content/
	H A D	ContextWrapper.java	549 public void enforceCallingOrSelfPermission( method in class:ContextWrapper 
	551 mBase.enforceCallingOrSelfPermission(permission, message);

## Steps

* Install Repo

    $ mkdir ~/bin
    $ export PATH=~/bin:$PATH
    $ curl http://android.git.kernel.org/repo > ~/bin/repo
    $ chmod a+x ~/bin/repo

For more info on repo, see: 

[version-control](http://source.android.com/source/version-control.html)

* The next step is to set up a local working directory (on a case-sensitive file-system)

    $ mkdir android-src
    $ cd android-src

* Next, we repo init to update repo itself as well as specify where we are going to download the sources from
(including the particular version/branch)

    $ repo init -u git://android.git.kernel.org/platform/manifest.git -b gingerbread

* The final step is to pull the actual files from the repository (2.6GB download)

    $ repo sync

## Source Code List



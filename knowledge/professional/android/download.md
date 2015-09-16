---
layout: master
title: Download android Source code
---


# Download android Source code

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

https://android.googlesource.com/



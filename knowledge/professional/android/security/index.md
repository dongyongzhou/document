---
layout: master
title: Android Security
---

## Introduction 

### Platform Security Architecture

Android seeks to be the most secure and usable operating system for mobile platforms by re-purposing traditional operating system security controls to:

- Protect user data
- Protect system resources (including the network)
- Provide application isolation


To achieve these objectives, Android provides these key security features:

- Robust security at the OS level through the Linux kernel
- Mandatory application sandbox for all applications
- Secure interprocess communication
- Application signing
- Application-defined and user-granted permissions




### [Android Security Evolution](android-security-evolution.html)


## Topics

### [Kernel Security](kernel-security.html)
### [App Security](app-security.html)
### [Android keymaster](android-keymaster.html)
### [Android Encryption](android-encryption.html)
### [Android verified boot](android-verified-boot.html)
https://source.android.com/devices/tech/security/verifiedboot/verified-boot.html
### [Android Security MISC](android-security.html)
### [Android Rooting principle(to be continued...)](android-rooting.html)


## Refernece & Resource

https://static.googleusercontent.com/media/source.android.com/en//devices/tech/security/reports/Google_Android_Security_2014_Report_Final.pdf

There are two great books.
 
- The first is Nikolay Elenkov's "Android Security Internals". He also writes a blog, called Android Explorations, which describes (in several posts) the keystore in depth. 

	[http://nelenkov.blogspot.com/](http://nelenkov.blogspot.com/)

- The second is "Android Internals" by Jonathan Levin, which discusses the keystore daemon. 

	[http://newandroidbook.com/index.php](http://newandroidbook.com/index.php)
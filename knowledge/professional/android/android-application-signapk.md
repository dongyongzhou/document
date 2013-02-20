---
layout: master
title: android-application-signapk
---

##1 OverView

Descriptions for how to sign an apk

##2 HOWTO


###2.1 tools

####2.1.1 signapk.jar

	out\host\linux-x86\framework\signapk.jar

####2.1.2 platform.x509.pem&platform.pk8

	build\target\product\security

###2.2 signapk


	java -jar signapk.jar platform.x509.pem platform.pk8 update.apk update_signed.apk


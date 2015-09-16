---
layout: master
title: Android Keymaster
---

## Overview

- Keystore: A keystore is a signed collection of public keys.
- Keymaster: is a newly introduced key management Hardware Abstraction Layer (HAL)
 component.

## Types

Types of keymaster HAL

### Software-based Keymaster
Uses the OpenSSL software implementation. Jelly Bean comes
with a default softkeymaster module that does all key operations in software only.

### Hardware-based keymaster
Uses TZ application APIs (keymaster application). Hardware
keymaster support essentially ensures that the key stored is not accessible in HLOS.
Regardless of key type (RSA/EC), the keyblob generated is encrypted by a key accessible by
TZ software only and stored in the File System (FS) on the HLOS end.



## Technology

## Reference

* [jelly-bean-hardware-backed-credential](http://nelenkov.blogspot.com/2012/07/jelly-bean-hardware-backed-credential.html)
* 
* 



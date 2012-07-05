---
layout: master
title: security
---

## Overview

Secure Sockets Layer (SSL) and Transport Layer Security (TLS) are cryptographic protocols that provide security for data as it moves across networks. 

The use of SSL/TLS is most often recognized for its role in securing web traffic, transforming normal HTTP into the secure HTTPS. 

Due to limitations and security concerns, **SSL is no longer considered a sufficient method for ensuring secure communications**. 

**Simply enabling TLS isn’t enough to protect your communications**, there are **a set of configuration options that should also be set properly in order to assure the confidentiality of network connections**. These settings and issues are described in the sections below.


### Cipher Suite

A cipher suite is a set of cryptographic algorithms defined for use with TLS.

TLS ciphersuite consists of the following three components:

- Public key handshake algorithm
- Symmetric encryption algorithms
- Hash function to be used in HMAC message authentication

## Deployment 

### Apache TLS Configuration

The following settings should be used to set the appropriate security settings in apache:

**SSLRandomSeed startup file:/dev/urandom 1024**

**SSLRandomSeed connect file:/dev/urandom 1024**

Use of /dev/urandom for connect is also desirable, but the built-in mechanism is also ok. There are some denial of service attacks against random number generation that blocks when there is not enough entropy on a system. To avoid this, a nonblocking source of random numbers such as /dev/urandom or built-in should be used on connect.

**SSLHonorCipherOrder on**

When choosing a cipher during a TLSv1 handshake, normally the client's preference is used. Our list of Ciphers has been carefully ranked in order, so the server should pick what we feel is the strongest cipher that the client and server have in common.

**SSLProtocol TLSv1**

**SSLCipherSuite "ECDHE-ECDSA-AES256-SHA,ECDHE-RSA-AES256-SHA,DHE-DSS-AES256-SHA,DHE-RSA-AES256-SHA,AES256-SHA,ECDHE-ECDSA-AES128-SHA,ECDHE-RSA-AES128-SHA,DHE-DSS-AES128-SHA,DHE-RSA-AES128-SHA,AES128-SHA,ECDHE-ECDSA-DES-CBC3-SHA,ECDHE-RSA-DES-CBC3-SHA,EDH-DSS-DES-CBC3-SHA,EDH-RSA-DES-CBC3-SHA,DES-CBC3-SHA"**


## Reference

 [NIST 800-57 (5.6.1 Comparable Algorithm Strengths)](http://csrc.nist.gov/publications/nistpubs/800-57/sp800-57-Part1-revised2_Mar08-2007.pdf)


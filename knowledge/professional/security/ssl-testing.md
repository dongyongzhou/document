---
layout: master
title: security ssl testing
---

## Overview

SSL/TLS wrapped services typically include port 443, which is the standard https port

however, this may change because 

- a) https services may be configured to run on non-standard ports
- b) there may be additional SSL/TLS wrapped services related to the web application.
 
In general, a service discovery is required to identify such ports. 


nmap scanner, via the “–sV” scan option, is able to identify SSL services. 

Vulnerability Scanners, in addition to performing service discovery, may include checks against weak ciphers (for example, the Nessus scanner has the capability of checking SSL services on arbitrary ports, and will report weak ciphers)

## Probe open ports to determine service/version info

$ nmap -F -sV xx.xxx.xxx.xx

    Starting Nmap 5.21 ( http://nmap.org ) at 2012-07-02 10:54 CST
    Nmap scan report for xx.com (xx.xxx.xxx.xx)
    Host is up (0.0011s latency).
    Not shown: 88 closed ports
    PORT     STATE SERVICE     VERSION
    21/tcp   open  ftp         vsftpd 2.3.2
    22/tcp   open  ssh         OpenSSH 5.8p1 Debian 7ubuntu1 (protocol 2.0)
    80/tcp   open  http        Apache httpd 2.2.17 ((Ubuntu))
    110/tcp  open  pop3        Dovecot pop3d
    111/tcp  open  rpcbind     2 (rpc #100000)
    139/tcp  open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
    143/tcp  open  imap        Dovecot imapd
    443/tcp  open  ssl/http    Apache httpd 2.2.17 ((Ubuntu))
    445/tcp  open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
    993/tcp  open  ssl/imap    Dovecot imapd
    995/tcp  open  ssl/pop3    Dovecot pop3d
    2049/tcp open  nfs         2-4 (rpc #100003)
    Service Info: OSs: Unix, Linux

    Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 18.98 seconds

## Identifying weak ciphers with Nessus. 

needs download and testing.

## Manually audit weak SSL cipher levels with OpenSSL

$ openssl s_client -no_tls1 -no_ssl3 -connect www.xxx.com:443


## Testing supported protocols and ciphers using sslscan. 

SSLScan queries SSL services, such as HTTPS and SMTP that supports STARTTLS, in order to determine the ciphers that are supported. SSLScan is designed to be easy, lean and fast. The output includes prefered ciphers of the SSL service, the certificate and is in Text and XML formats.

[sslscan](http://www.michaelboman.org/books/sslscan)

    $ ./sslscan --no-failed xxx.com


### Errors:

    Prefered Server Cipher(s):
    ERROR: Could not create CTX object.

Solution:

This is probably because, afaik, recent Ubuntu versions ship openssl without SSLv2 support. By default, sslscan will try SSLv2, SSLv3, and TLSv1. However, sslscan isn't expecting SSLv2 to be not available at all.

To use sslscan on recent Ubuntu versions, use the --ssl3 or --tls1 options to sslscan.

Note that you should be very careful about testing remote servers for SSLv2 support when using an Ubuntu client, including with "openssl s_client -ssl2 -connect somehost:443". Using an Ubuntu client, you may incorrectly determine that the remote server doesn't support SSLv2 when in fact it really does, it's just your client that doesn't support SSLv2.


## Testing common SSL flaws with ssl_tests 

[ssl_tests](http://www.pentesterscripting.com/discovery/ssl_tests)

**SSL Tests - v2, weak ciphers, MD5, Renegotiation**

bash script that uses sslscan and openssl to check for various flaws - ssl version 2, weak ciphers, md5withRSAEncryption,SSLv3 Force Ciphering Bug/Renegotiation. 


    $ ./tlssled.sh xx.xxx.xxx.xx 443


## use nmap to determine what ciphers a server is currently offering. 

[Nmap download](http://nmap.org/download.html)

### Installation:

    $ bzip2 -cd nmap-6.01.tar.bz2 | tar xvf -
    $ cd nmap-6.01
    $ ./configure
    $ make
    $ su root
    $ make install

### testing

$ nmap --script ssl-enum-ciphers --script-args sslenum=weak -p 443 <host>
$ nmap --script ssl-enum-ciphers -p 443 <host>






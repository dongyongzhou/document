---
layout: master
title: Base Security Configuration for server deployment
---

## Overview



## To disable the Server header in Apache

### phenomenant

    Date: Tue, 05 Jun 2012 21:38:28 GMT
    Server: Apache/2.2.17 (Ubuntu)
    X-Powered-By: Phusion Passenger (mod_rails/mod_rack) 3.0.11 

### configuration

To disable the Server header in Apache 2.x you first need to make sure you have configured mod_headers to load in httpd.conf:

    # customization of HTTP response headers
    LoadModule headers_module /usr/lib/apache/modules/mod_headers.so

At some point after this line, you can remove all Server header information with the following line:

    Header unset Server

但是不管用，Header unset对Server无用。用ServerTokens命令：

    ServerTokens Prod

    现象:Server: Apache 

For completeness, issue the ServerSignature option:

    ServerSignature Off


## Disable X-Powered-By

### phenomenant

    X-Powered-By: Phusion Passenger (mod_rails/mod_rack) 3.0.11


### configuration

    Header always unset "X-Powered-By"



## X-Frame-Options

X-Frame-Options instructs the browser to disallow framing of a domain or limit framing to only sites of the same domain. The value of this header can be DENY to deny any framing or SAMEORIGIN to allow sites of the same domain to frame content.

### setting

    Header set X-Frame-Options DENY
    ....

    <LocationMatch "....">
      Header unset X-Frame-Options
    </LocationMatch>


    Header set X-Frame-Options SAMEORIGIN


## Remove Old, Backup and Unreferenced Files

## avoid weak Ciphers

    /etc/apache2/sites-available/xxxx

    Old: SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL

    New:  SSLCipherSuite "ECDHE-ECDSA-AES256-SHA,ECDHE-RSA-AES256-SHA,DHE-DSS-AES256-SHA,DHE-RSA-AES256-SHA,AES256-SHA,ECDHE-ECDSA-AES128-SHA,ECDHE-RSA-AES128-SHA,DHE-DSS-AES128-SHA,DHE-RSA-AES128-SHA,AES128-SHA,ECDHE-ECDSA-DES-CBC3-SHA,ECDHE-RSA-DES-CBC3-SHA,EDH-DSS-DES-CBC3-SHA,EDH-RSA-DES-CBC3-SHA,DES-CBC3-SHA"


## Reference

* [OWASP Testing Guide v3 Table of Contents](https://www.owasp.org/index.php/OWASP_Testing_Guide_v3_Table_of_Contents)


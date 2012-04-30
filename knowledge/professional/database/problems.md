---
layout: master
title: Server
---

## PERL certificate verify failed

### Description

LWP::Protocol::https::Socket: SSL connect attempt failed with unknown errorerror:14090086:SSL routines:SSL3_GET_SERVER_CERTIFICATE:certificate verify failed at /usr/share/perl5/LWP/Protocol/http.pm <https::Socket: SSL connect attempt failed with unknown errorerror:14090086:SSL routines:SSL3_GET_SERVER_CERTIFICATE:certificate verify failed at /usr/share/perl5/LWP/Protocol/http.pm>  line 51.

### Solution one

Turn off the verification

You’re seeing this because the most recent versions of LWP::UserAgent require a signed certificate. 

The easiest way to tell the library to ignore this check is to set a command line environment variable from bash:

> export PERL_LWP_SSL_VERIFY_HOSTNAME=0

### Solution two

Reference: 

[LWP::UserAgent - Web user agent class](http://cpan.uwinnipeg.ca/htdocs/libwww-perl/LWP/UserAgent.html#code_verify_hostname_code_gt_bool)
The same with the first one.

When TRUE LWP will for secure protocol schemes ensure it connects to servers that have a valid certificate matching the expected hostname. If FALSE no checks are made and you can't be sure that you communicate with the expected peer. The no checks behaviour was the default for libwww-perl-5.837 and earlier releases.

    my $ua = LWP::UserAgent->new(
      ssl_opts => {
        verify_hostname => 0,
     #    SSL_ca_path => 'xxx',
     #    SSL_ca_path => 'xxx',
     },
   );


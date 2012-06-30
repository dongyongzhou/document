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


## Apache2: you don't have permission to access / on this server

### Description

browser: 
    
    you don't have permission to access / on this server

/var/log/apache2/...

    [Fri Jun 29 19:57:24 2012] [error] [client <ip>] client denied by server configuration: /var/www/xxx/

### Solution

**change from** 

    <VirtualHost *:80>
      ServerName xxx
      DocumentRoot /var/www/xxx/
      <Directory /var/www/xxx/>
        AllowOverride all
        Options -MultiViews
      </Directory>

**change to**

    <VirtualHost *:80>
      ServerName xxx
      DocumentRoot /var/www/xxx/
      <Directory /var/www/xxx/>
        Order allow,deny
        Allow from all
      </Directory>


## Apache2: The requested URL / was not found on this server.
 

### Description

browser: 
    
    The requested URL / was not found on this server.

/var/log/apache2/...

    [Fri Jun 29 20:07:53 2012] [error] [client <ip>] Attempt to serve directory: /var/www/xxxx/

### solution:

for ruby project.it is wrong for us to define both DocumentRoot and Directory as a link in /var/www/xxxx from the project`s public directory. 

define it as /home/dongyong/xxxx/public

or put the xxxx to /var/www location.
 
    <VirtualHost *:443>
      ServerName qrd-dm.qualcomm.com
      DocumentRoot /home/dongyong/xxxx/public
      <Directory /home/dongyong/xxxx/public>

 

## [Fri Jun 29 23:08:03 2012] [error] *** Passenger could not be initialized because of this error: The option PassengerDefaultUser is set to 'nobody', but its primary group doesn't exist. In other words, your system's user account database is broken. Please fix it.

### Description

/var/log/apache2/...

    [Fri Jun 29 23:08:03 2012] [error] *** Passenger could not be initialized because of this error: The option PassengerDefaultUser is set to 'nobody', but its primary group doesn't exist. In other words, your system's user account database is broken. Please fix it.

### Solution

    LoadModule passenger_module /usr/local/rvm/gems/ruby-1.8.7-p352/gems/passenger-3.0.13/ext/apache2/mod_passenger.so
    PassengerRoot /usr/local/rvm/gems/ruby-1.8.7-p352/gems/passenger-3.0.13
    PassengerRuby /usr/local/rvm/wrappers/ruby-1.8.7-p352/ruby
    +PassengerDefaultUser www-data
    PassengerAnalyticsLogGroup www-data


##[ pid=5567 thr=139829314213696 file=ext/common/LoggingAgent/Main.cpp:287 time=2012-06-29 23:47:19.465 ]: *** ERROR: The configuration option 'PassengerAnalyticsLogGroup' (Apache) or 'passenger_analytics_log_group' (Nginx) wasn't set, so PassengerLoggingAgent tried to use the default group for user 'nobody' - which is GID #60001 - as the group for the analytics log dir, but this GID doesn't exist. You can solve this problem by explicitly setting PassengerAnalyticsLogGroup (Apache) or passenger_analytics_log_group (Nginx) to a group that does exist. In any case, it looks like your system's user database is broken; Phusion Passenger can work fine even with this broken user database, but you should still fix it.

### Description

/var/log/apache2/...

[ pid=5567 thr=139829314213696 file=ext/common/LoggingAgent/Main.cpp:287 time=2012-06-29 23:47:19.465 ]: *** ERROR: The configuration option 'PassengerAnalyticsLogGroup' (Apache) or 'passenger_analytics_log_group' (Nginx) wasn't set, so PassengerLoggingAgent tried to use the default group for user 'nobody' - which is GID #60001 - as the group for the analytics log dir, but this GID doesn't exist. You can solve this problem by explicitly setting PassengerAnalyticsLogGroup (Apache) or passenger_analytics_log_group (Nginx) to a group that does exist. In any case, it looks like your system's user database is broken; Phusion Passenger can work fine even with this broken user database, but you should still fix it.

### Solution


    LoadModule passenger_module /usr/local/rvm/gems/ruby-1.8.7-p352/gems/passenger-3.0.13/ext/apache2/mod_passenger.so
    PassengerRoot /usr/local/rvm/gems/ruby-1.8.7-p352/gems/passenger-3.0.13
    PassengerRuby /usr/local/rvm/wrappers/ruby-1.8.7-p352/ruby
    PassengerDefaultUser www-data
    +PassengerAnalyticsLogGroup www-data
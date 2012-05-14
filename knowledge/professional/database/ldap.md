---
layout: master
title: LDAP
---

##1 Overview

**LDAP** stands for **Lightweight Directory Access Protocol**. It is a means of reading and writing data to a Directory. LDAP version 3 (LDAP v.3) is the commonly supported current version.

An LDAP v.3 Directory consists of a backend database that contains **directory objects and indexes**. These objects are organized into a **hierarchical structure** which makes *searching of the directory extremely fast*. This structure makes LDAP an excellent choice to support applications that primarily execute read transactions against a database.

The Hierarchical Structure of the database is also effective for establishing easy to manage and understand Access Control rules that **control authorization for management of objects**.

##2 LDAP Understanding

To be continued.

##3 Authenticating against LDAP

**User (Person) Objects**

- base: "ou=people,o=xxx"
- user name attribute: "uid"
- user objectclass: "inetorgperson"

**Group Objects**

- base: "ou=groups,o=xxx"
- user name attribute: "cn"
- user objectclass: "groupOfNames"

**Testing Authentication on Linux**

Install ldap utils

    $ sudo apt-get install ldap-utils

Testing command

    $ ldapsearch -h <ldaphost> -p <ldapport> -x -W -D "uid=username,ou=people,o=xxx" -b "ou=people,o=xxx" "uid=username"

       -D binddn
              Use the Distinguished Name binddn to bind to the LDAP directory.  For SASL binds, the server is expected to ignore this value.

       -W     Prompt for simple authentication.  This is used instead of specifying the password on the command line.

If authenticate succeeds, the command will output a copy of your LDAP record, otherwise it will throw an error.

**Reading Users information**

    $ ldapsearch -h <ldaphost> -p <ldapport> -x -b "ou=people,o=xxx" "uid=username"

    -b searchbase
              Use searchbase as the starting point for the search instead of the default.

**Reading Groups information**

    $ ldapsearch -h <ldaphost> -p <ldapport> -x -b "ou=<xgroups>,ou=groups,o=xxx" "cn=groupname"


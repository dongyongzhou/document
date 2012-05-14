---
layout: master
title: LDAP-ror
---

##1 Overview

Solutions: net-ldap  

Net::LDAP for Ruby (also called net-ldap) implements client access for the Lightweight Directory Access Protocol (LDAP), an IETF standard protocol for accessing distributed directory services. Net::LDAP is written completely in Ruby with no external dependencies. It supports most LDAP client features and a subset of server features as well.

Net::LDAP has been tested against modern popular LDAP servers including OpenLDAP and Active Directory. The current release is mostly compliant with earlier versions of the IETF LDAP RFCs (2251–2256, 2829–2830, 3377, and 3771). Our roadmap for Net::LDAP 1.0 is to gain full client compliance with the most recent LDAP RFCs (4510–4519, plus portions of 4520–4532).

LDAP library for Ruby with no extensions; a full-featured LDAP client is currently available. Development is now on GitHub at http://github.com/ruby-ldap/ruby-net-ldap/

##2 Installation

Version: net-ldap-0.2.2

    gem install net-ldap

Simply require either ‘net-ldap’ or ‘net/ldap’.

##3 Programming

###3.1 Authenticating against LDAP

    require 'rubygems'
    require 'net/ldap'
    
    ldap = Net::LDAP.new
    ldap.host = your_server_ip_address
    ldap.port = 389
    ldap.auth "joe_user", "opensesame"
    if ldap.bind
      # authentication succeeded
    else
      # authentication failed
    end

###3.1 Search against an LDAP directory

    require 'rubygems'
    require 'net/ldap'
    
    ldap = Net::LDAP.new :host => server_ip_address,
         :port => 389,
         :auth => {
               :method => :simple,
               :username => "cn=manager, dc=example, dc=com",
               :password => "opensesame"
         }
    
    filter = Net::LDAP::Filter.eq("cn", "George*")
    treebase = "dc=example, dc=com"
    
    ldap.search(:base => treebase, :filter => filter) do |entry|
      puts "DN: #{entry.dn}"
      entry.each do |attribute, values|
        puts "   #{attribute}:"
        values.each do |value|
          puts "      --->#{value}"
        end
      end
    end

OR

Check if one is in a specified group

    attrs = ["member","dn"]
    
    ldap.search(:base => treebase, :filter => "cn=#{group}", :attributes => attrs,
            :return_result => false) do |entry|
            puts "DN: #{entry.dn}"
            entry.each do |attr, values|
                    puts ".......#{attr}:"
                    values.each do |value|
                            logger.info "          #{value}"

                            if value.match(user_name)
                                    puts "#{user_name} is in the group #{group}"
                                    return true
                            end

                    end
            end

    end

##4 Reference

* [net-ldap-0.2.2 Documentation](http://net-ldap.rubyforge.org/)
* [Net::LDAP Quick-start for the Impatient](http://net-ldap.rubyforge.org/Net/LDAP.html#method-c-new)

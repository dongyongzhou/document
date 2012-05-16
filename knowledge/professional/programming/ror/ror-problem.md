---
layout: master
title: Problems of Ruby on Rails
---

## Problems of Ruby on Rails

###1 rubygems.rb:335:in `bin_path': can't find executable rails for rails-3.2.3  

**Description**:

    $ rails server
    /usr/local/rvm/rubies/ruby-1.8.7-p352/lib/ruby/site_ruby/1.8/rubygems.rb:335:in `bin_path': can't find executable rails for rails-3.2.3 (Gem::Exception)
    from /usr/local/rvm/gems/ruby-1.8.7-p352/bin/rails:19

**Solution**

 install the railties gem.

    $ gem install railties
    Successfully installed railties-3.2.3
    1 gem installed
    Installing ri documentation for railties-3.2.3...
    Installing RDoc documentation for railties-3.2.3...

###2 execjs/runtimes.rb:50:in `autodetect': Could not find a JavaScript runtime 

**Description**:

usr/local/rvm/gems/ruby-1.8.7-p352/gems/execjs-1.3.2/lib/execjs/runtimes.rb:50:in `autodetect': Could not find a JavaScript runtime. See https://github.com/sstephenson/execjs for a list of available runtimes. (ExecJS::RuntimeUnavailable)

**Solution**

Add to Gemfile

    gem 'execjs'
    gem 'therubyracer'

bundle install    

---
layout: master
title: Rails
---

## 参考资料

* [Ruby On Rails官方网站](http://rubyonrails.org/)
* [RailsCast](http://railscasts.com/)
* [The Ruby Toolbox](https://www.ruby-toolbox.com/)
* [The RubyonRails API](http://api.rubyonrails.org/)

## Rails 运行模式

### development

每次修改后，下一个请求以后都会自动生效，因为reload了code

config/environments/development.rb

    # In the development environment your application's code is reloaded on
    # every request.  This slows down response time but is perfect for development
    # since you don't have to restart the webserver when you make code changes.
    config.cache_classes = false

### Production 

每次修改后，不会自动生效

config/environments/production.rb

    # The production environment is meant for finished, "live" apps.
    # Code is not reloaded between requests
    config.cache_classes = true


## before_filter

    before_filter :require_admin,     :only => [:destroy]
    before_filter :authorize,         :except => [:index, :new]

## script/plugin

There is no longer have a script/plugin directory Rails 3. 

Rails 3 now runs commands through the "rails" script.

    rails plugin install http://url/of/your/package

## cookie-session

Rails2之前是把session放在服务器端

Rails2之后默认采用了在browser的cookie中保存session数据的方式，因此保存在session中的数据不能超过4K，否则会出现CookieOverflow的例外

在session中一般保存有user_id和flash的内容，在使用了protect_from_forgery后，还会保存有_csrf_token的字段。此外，可能还会有session_id的字段，但是由于session内容是保存在单独的cookie中，而不是数据库中，所以在使用cookie-session的时候，这个session_id并没有实际的意义。

session中的数据在保存在cookie中时是先marshal后，然后利用密码来加密的。假设session的内容是data，那么实际在cookie中的内容就会是如下形式

    marshal(data)---digest_with_secret_key(marshal(data))

这里在加密时所用的secret key就是在config/initializer/session_store.rb中所设置的serect字段；而在该文件中设置的key字段，就是保存在browser中的cookie的名称。

## protect_from_forgery

rails2.0以后的版本都会默认开启该选项(在application_controller.rb中)，它会利用保存在cookie中的_csrf_token字段来生成自动添加在form中的隐藏字段_authenticity_token，然后利用_authenticity_token来实现CSRF的功能。

## CSRF

**Cross Site Reference Forgery** works by including malicious code or a link in a page that accesses a web application that the user is believed to have authenticated. If the session for that web application has not timed out, an attacker may execute unauthorized commands.

## helper_method

helper_method : 可以指定控制器中的某些方法为helper方法，这些方法可以直接在视图中使用。

### example

**application_controller.rb**

    class ApplicationController < ActionController::Base
      protect_from_forgery
      helper_method :current_user
    
      private
    
      def current_user
        @current_user ||= User.find(session[:user_id]) if session[:user_id]
      end
    end
    
**views/layouts/application.html.erb**
    
    <html>
    <head>
      <title>Auth</title>
      <%= stylesheet_link_tag :all %>
      <%= javascript_include_tag :defaults %>
      <%= csrf_meta_tag %>
    </head>
    <body>
    
    <div id="user_nav">
      *<% if current_user %>*
        *Logged in as <%= current_user.email %>.*
        <%= link_to "Log out", log_out_path %>
      <% else %>
        <%= link_to "Sign up", sign_up_path %> or
        <%= link_to "log in", log_in_path %>
      <% end %>
    </div>
    
    <% flash.each do |name, msg| %>
      <%= content_tag :div, msg, :id => "flash_#{name}" %>
    <% end %>
    
    <%= yield %>
    
    </body>
    </html>


---
layout: master
title: user authentication
---

##1 Solution: devise

* [Devise](https://github.com/plataformatec/devise)

###1.1 Installation

    $ gem install devise

Successfully installed devise-2.0.4
1 gem installed
Installing ri documentation for devise-2.0.4...
Installing RDoc documentation for devise-2.0.4...


you need to run the generator:

    $ rails generate devise:install

The generator will install an initializer which describes ALL Devise's configuration options and you MUST take a look at it. When you are done, you are ready to add Devise to any of your models using the generator:

$ rails generate devise:install
      create  config/initializers/devise.rb
      create  config/locales/devise.en.yml
===============================================================================

Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { :host => 'localhost:3000' }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root :to => "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. If you are deploying Rails 3.1 on Heroku, you may want to set:

       config.assets.initialize_on_precompile = false

     On config/application.rb forcing your application to not access the DB
     or load models when precompiling your assets.


and delete the .git folder inside the directory.

###1.2 Configuration

**1.2.1 create a model**

    rails generate devise MODEL

Replace MODEL by the class name used for the applications users, it's frequently 'User' but could also be 'Admin'. This will create a model (if one does not exist) and configure it with default Devise modules. 
Next, you'll usually run "**rake db:migrate**" as the generator will have created a migration file (if your ORM supports them). 

This generator also configures your **config/routes.rb** file to point to Devise controller.

Note that you should re-start your app here if you've already started it. Otherwise you'll run into strange errors like users being unable to login and the route helpers being undefined.

**1.2.2 Controller filters and helpers**

Devise will create some helpers to use inside your controllers and views. **To set up a controller with user authentication**, just add this before_filter:

    before_filter :authenticate_user!

**To verify if a user is signed in**, use the following helper:

    user_signed_in?

For the **current signed-in user**, this helper is available:

    current_user

You can **access the session** for this scope:

    user_session

**1.2.3 Configuring Models**


**1.2.4 Configuring multiple models**


**1.2.5 Configuring views**


## Solution: restful-authentication

Not so available for rails3

## Reference

* [Rails Authentication](https://www.ruby-toolbox.com/categories/rails_authentication.html)

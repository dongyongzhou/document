---
layout: master
title: Ruby on Rails
---

# Ruby on Rails


## Email Services

### Reference

[Action Mailer Basics](http://guides.rubyonrails.org/action_mailer_basics.html) .

Action Mailer allows you to send emails from your application using a mailer model and views. So, in Rails, emails are used by creating mailers that inherit from ActionMailer::Base and live in app/mailers. Those mailers have associated views that appear alongside controller views in app/views.

### Sending Emails

#### Create the Mailer

    $ rails generate mailer UserMailer

#### Edit the Mailer

Let’s add a method called welcome_email, that will send an email to the user’s registered email address:
   
    def welcome_email(user)
      @user = user
      @url  = "http://example.com/login"
      mail(:to => user.email, :subject => "Welcome to My Awesome Site")
    end 

#### Create a Mailer View

Refer to: http://guides.rubyonrails.org/action_mailer_basics.html

Create a file called welcome_email.html.erb in app/views/user_mailer/. 

This will be the template used for the email, formatted in HTML.

It is also a good idea to make a text part for this email, to do this, create a file called welcome_email.text.erb
When you call the mail method now, Action Mailer will detect the two templates (text and HTML) and automatically generate a multipart/alternative email.

#### Wire It Up So That the System Sends the Email When in need

There are several ways to do this

1. some people create Rails Observers to fire off emails.

2. others do it inside of the User Model. 

3. in Rails 3, mailers are really just another way to render a view. Instead of rendering a view and sending out the HTTP protocol, they are just sending it out through the Email protocols instead. Due to this, it makes sense to just have your controller tell the mailer to send an email when in need.

Sending Email in controllers  

By inserting a call to UserMailer.welcome_email in the specify method.
    UserMailer.welcome_email(@user).deliver

#### Configuration

Action Mailer Configuration for GMail.
Adding the following to appropriate config/environments/$RAILS_ENV.rb file:

    config.action_mailer.delivery_method = :smtp
    config.action_mailer.smtp_settings = {
      :address              => "smtp.gmail.com",
      :port                 => 587,
      :domain               => 'baci.lindsaar.net',
      :user_name            => '<username>',
      :password             => '<password>',
      :authentication       => 'plain',
      :enable_starttls_auto => true  }


#### End 

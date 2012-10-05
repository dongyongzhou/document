---
layout: master
title: Email Services
---

## Email Services

### 1. Reference

[Action Mailer Basics](http://guides.rubyonrails.org/action_mailer_basics.html) .

Action Mailer allows you to send emails from your application using a mailer model and views. So, in Rails, emails are used by creating mailers that inherit from ActionMailer::Base and live in app/mailers. Those mailers have associated views that appear alongside controller views in app/views.

### 2. Sending Emails

#### 2.1 Create the Mailer

    $ rails generate mailer UserMailer

create    app/mailers/user_mailer.rb
create    app/views/user_mailer
create    test/functional/user_mailer_test.rb

#### 2.2 Edit the Mailer

##### Complete List of Action Mailer Methods

+ headers – Specifies any header on the email you want, you can pass a hash of header field names and value pairs, or you can call headers[:field_name] = ‘value’

+ attachments – Allows you to add attachments to your email, for example attachments[‘file-name.jpg’] = File.read(‘file-name.jpg’)

Pass the file name and content and Action Mailer and the Mail gem will automatically guess the mime_type, set the encoding and create the attachment.
    attachments['filename.jpg'] = File.read('/path/to/filename.jpg')

Mail to turn an attachment into an inline attachment, you just call #inline on the attachments method within your Mailer:
    attachments.inline['image.jpg'] = File.read('/path/to/image.jpg')


content_type:

     ".asf"     ContentType = "video/x-ms-asf" 
     ".avi"     ContentType = "video/avi" 
     ".doc"    ContentType = "application/msword" 
     ".zip"     ContentType = "application/zip" 
     ".xls"     ContentType = "application/vnd.ms-excel" 
     ".gif"     ContentType = "image/gif" 
     ".jpg", "jpeg"       ContentType = "image/jpeg" 
     ".wav"   ContentType = "audio/wav" 
     ".mp3" ContentType = "audio/mpeg3" 
     ".mpg", "mpeg"    ContentType = "video/mpeg" 
     ".rtf"     ContentType = "application/rtf" 
     ".htm", "html"      ContentType = "text/html" 
     ".txt"     ContentType = "text/plain"
     ".pdf"    ContentType = "application/pdf"
     ".swf"    ContentType = "application/x-shockwave-flash"
    其他      ContentType = "application/octet-stream"

+ mail – Sends the actual email itself. You can pass in headers as a hash to the mail method as a parameter, mail will then create an email, either plain text, or multipart, depending on what email templates you have defined.

##### Let’s add a method called welcome_email, that will send an email to the user’s registered email address:
app/mailers/user_mailer.rb
    def welcome_email(user)
      @user = user
      @url  = "http://example.com/login"
      mail(:to => user.email, :subject => "Welcome to My Awesome Site")
    end 

##### Add a method called send_email, with Attachments, to Multiple Recipients.

* Attachments:
    attachments['image.jpg'] = File.read('/path/to/image.jpg')

* Multiple Recipients:

It is possible to send email to one or more recipients in one email (for e.g. informing all admins of a new signup) by setting the list of emails to the :to key. The list of emails can be an array of email addresses or a single string with the addresses separated by commas.
    class AdminMailer < ActionMailer::Base
      default :to => Admin.all.map(&:email),
              :from => "notification@example.com"

or

      default :from => "notification@example.com",
          :to => ["1x@xxx.com","2x@xxx.com","3x@xxx.com"],
          :cc => [...]

The same format can be used to set carbon copy (Cc:) and blind carbon copy (Bcc:) recipients, by using the :cc and :bcc keys respectively.

*  interpolate file contents

Controller

   @data = ""
    File.open("file", "r") { |fp|
            @data<<fp.read()
    }


    ``r'' Read-only, starts at beginning of file (default mode).
    只读，清除原有内容（默认方式）
    ``r+'' Read-write, starts at beginning of file.
    读写，清除原有内容
    ``w'' Write-only, truncates existing file to zero length or creates a new file for writing.
    只写，创建一个新的文件覆盖旧的
    ``w+'' Read-write, truncates existing file to zero length or creates a new file for reading and writing.
    读写，创建一个新的文件覆盖旧的
    ``a'' Write-only, starts at end of file if file exists, otherwise creates a new file for writing.
    只写，追加
    ``a+'' Read-write, starts at end of file if file exists, otherwise creates a new file for reading and writing.
    读写，追加
    ``b'' (DOS/Windows only) Binary file mode (may appear with any of the key letters listed above).
    二进制模式


view:

    <%= @data %>

#### 2.3 Create a Mailer View

Refer to: http://guides.rubyonrails.org/action_mailer_basics.html

For welcome_email:
Create a file called welcome_email.html.erb or welcome_email.text.erb in app/views/user_mailer/. 
Reference for templates [Example](http://guides.rubyonrails.org/action_mailer_basics.html#complete-list-of-action-mailer-methods)

When you call the mail method now, Action Mailer will detect the two templates (text and HTML) and automatically generate a multipart/alternative email.
This will be the template used for the email, formatted in HTML.

For send_email:
Create a file called send_email.html.erb or send_email.text.erb in app/views/user_mailer/. 


#### 2.4 Wire It Up So That the System Sends the Email When in need

There are several ways to do this

1. some people create Rails Observers to fire off emails.

2. others do it inside of the User Model. 

3. in Rails 3, mailers are really just another way to render a view. Instead of rendering a view and sending out the HTTP protocol, they are just sending it out through the Email protocols instead. Due to this, it makes sense to just have your controller tell the mailer to send an email when in need.

Sending Email in controllers  

By inserting a call to UserMailer.welcome_email in the specify method.
    UserMailer.welcome_email(@user).deliver

### 2. Receive email

* [Receiving Incoming Email in Rails 3 – choosing the right approach](http://steve.dynedge.co.uk/2010/09/07/incoming-email-in-rails-3-choosing-the-right-approach/)



### 4. Configuration

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


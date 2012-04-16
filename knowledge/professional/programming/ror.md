---
layout: master
title: Ruby on Rails
---

# Ruby on Rails

## Ruby 

### 参考资料

[Ruby的官方网站](http://www.ruby-lang.org/en/)

[Programming Ruby](http://www.rubycentral.com/pickaxe/)

### Ruby 变量

Ruby属于动态类型。

在大多数语言里,变量都必须指定其类型,可更改性(是不是个常数)和范围;,在Ruby里变量不需要声明.

>
一般小写字母、底线开头：变量（Variable）。

$开头：全局变量（Global variable）。

@开头：实例变量（Instance variable）。

@@开头：类变量（Class variable）类别变量被共享在整个继承链中

大写字母开头：常数（Constant）。

- 局部变量
  
  小写字母或'_'起始, 局部变量不像全局和实变量一样在初始化前含nil值

  作用域：定义它的方法或循环内部


- 实例变量
 
  @作字首, 像全局变量一样,实变量在初始前的值是nil

  作用域：限制在 self 对象内
  
- 类变量

  @@作字首

- 全局变量

  $作字首  在初始化前,全局变量有一个特殊的值 nil.

  作用域：程序的任何位置
  
- 常量

  大写字母开头  它应最多被赋值一次.

  作用域：常量可以定义在类里,但不像实变量,它们可以在类的外部访问.
  
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

### 3. Configuration

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


### 4. End 

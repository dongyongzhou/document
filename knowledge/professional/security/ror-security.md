---
layout: master
title: security of Ruby on rails
---

## Overview

描述的web应用里普遍的安全问题，同时给出了在Rails里如何避免这些问题。

- 在Rails里session的概念， 该放什么在session里，以及一些流行的攻击方法
- 只是浏览一个站点，怎么就有安全问题呢？(with CSRF) 
- 当你使用files或提供一个管理界面的时候需要注意些什么
- The Rails-specific mass assignment problem 
- 如何管理用户：登陆，注销以及对所有层面的攻击方法
- 最流行的注入攻击方法

##2 Knowledge

安全依赖于正在使用的框架，有时候也依赖于开发方式。它取决于web应用的所有环境：后端存储，网络服务器和网络应用程序本身（以及可能的其他层或应用程序） 。

web application框架帮助开发者建立种种web应用，某些框架在安全方面也帮你省心不少。事实上，一个框架并不比另一个安全。对于大多数的框架来说，如果你正确使用它，可以建立安全的应用。Ruby on Rails有许多聪明的helper方法，例如防止注入攻击的方法，这让sql注入变成了困难的事情。

在受到威胁的web应用中，包括**用户帐户劫持**，**绕开访问控制**，**阅读或修改敏感数据**，或**出示虚假的内容**。或攻击者可以**安装一个特洛伊木马程序或不请自来的电子邮件发送软件**，目的是在金融活动或造成损害品牌名称，修改公司的资源。为了防止攻击，最大限度地减少其影响和消除攻击点，首先，你必须充分了解攻击方法， 以便找到正确的对策。这正是该指南的目的。

###2.1 Sessions

sessions最易受到特别攻击

#### 2.1.1   什么是sessions

- HTTP是一个无状态协议，Sessions使它有状态。

大多数的应用需要对一个特别用户的某些状态进行跟踪。比如，一个购物车的内容，一的当前登陆用户的id，如果没有sessions这个好主意，用户必须在每一个请求都得去标识验证身份。如果一个新用户访问这个应用，Rails会自动创建一个新的session。如果用户之前使用过这个应用，它会自动加载一个存在的session。

一个session通常是一个hash和一个session id（通常是一个32个字符的字符串）来标识这个hash。

Rails里你可以用如下方式保存和使用session：

    session[:user_id] = @current_user.id
    User.find(session[:user_id])

#### 2.1.2  session id
- The session id is a 32 byte long MD5 hash value.
session id就是个一32位的md5 hash值。

一个session id由一个随机字符串的hash值组成。这个随机字符串是当前的时间，一个0和1之间的随机数字，一个ruby解释器进程id数字和一个常量字符串组成。

#### 2.1.3  session劫持

- 窃取用户session id的攻击者可以在一个web应用里使用受害者的名字。

**许多web应用都有一个验证系统**：一个用户提供一个用户名和密码，web应用检查并且存储相应的user id到session的这个hash里。从现在开始，session是有效的。在每个http请求里， 应用会加载这些在session里用user id来标识的用户，并不需要新的验证。这个session id是放在cookie里用来标识这个session的。

因此cookie作为web应用的临时验证。任何人得到一个别人的cookie，他就可以伪装为这个人使用对应的web应用－可能会有严重的后果。这里有一些session劫持的方法，以及对策：

1.在不安全的网络进行嗅探cookie。无线局域网就是这样一个例子。在这样的一个网络里，特别容易去监听所有链接的客户端， 这就是不去咖啡店工作的原因（某些sb就喜欢去星巴克打开本本装比）。对于web应用开发者，这就意味着去提供一个安全的ssl连接。

2.大多数的人在公共场所工作之后不清除cookie，所以如果是最后一个没有在web应用注销的用户，你有可能伪装成这个用户。在web应用里提供给用户一个注销按钮，并且放在醒目的位置。

3.许多跨站点脚本（ XSS ）攻击的目的是获得用户的cookie 。你之后会看到关于xss的更多内容。

4.Session定制，之后会读到. 

大多数攻击者的目的是为了钱，失窃的银行登陆帐号的价格范围从$10 - $1000不等（取决于可用的资金数额），信用卡号码是$0.40-$20，在线拍卖网站的帐号为$1-$8，email密码为$4-$30 ，参考赛门铁克的网络安全威胁报告。

#### 2.1.4 session 准则

- 这里有一些session的一般准则

1.不要在session里存储大的对象。相反，你应该把它们存储在数据库里，把它们的id保存在session里。这将消除同步的麻烦，而且也不会占用你session的存储空间（取决于你选择的session的存储）

2.关键的数据不应该被存储在session里。如果用户清除了cookie或者关闭了浏览器，它们都将丢失，用一个客户端session存储，用户可以读取数据。

#### 2.1.5 session存储

- Rails为session提供了很多存储机制，最重要的是ActiveRecordStore和CookieStore。

大多数实际生活的应用选择ActiveRecordStore （或其衍生物）的文件存储由于性能和维修的原因。 ActiveRecordStore保持session ID和散列在一个数据库表，在每次请求里都保存和检索这个hash。

Rails2 介绍了一种新的session存储机制， CookieStore. 它直接在客户端的cookie里保存这个session hash。服务端从cookie里检索这个session hash，无须session id。这将大大增加速度的应用，但它是一个有争议的存储选项，你必须思考安全的影响：

1.cookie意味着严格的大小限制，4k。 这点是好的，本身就不应该存储大的数据在session里，前面也说过。存储一个当前用户的数据库id一般是没有问题的。

2.客户端能看到你存储在session里的一切。因为是明文的（实际是base64编码的），所以，你别想在这里存储任何秘密。为了防止session hash被篡改，会从服务端加一个secret到cookie的末尾。 

这就意味着，这个安全存储取决于这个scret（digest算法，默认是sha512，至今没有被破解），所以不要用一个简单的scret，例如字典里的一个单词，或者比30个字符短的任意字符串。把这个scret放在environment.rb里：

    config.action_controller.session = {
     :session_key => ‘_app_session’ 
     :secret      => ‘0x0dkfj3927dkc7djdh36rkckdfzsg...’
    }

但是也有经过加密session hash的CookieStore派生存储机制，为了客户端不能看到它。

#### 2.1.6   针对于CookieStore sessions的重放攻击

- 另一种必须要知道的攻击是针对于cookiestore攻击是重放攻击。

它的运作方式如下：
 1.用户收到贷款，金额是储存在一个session里，（这是坏主意，但是我们做这些只是示范）
 2.这个用户买了一些东西
 3.他的new，lower credit将要被存储在session里。
 4.邪恶的用户得到了他的cookie并且覆盖了当前的cookie。
 5.这个用户有了他的credit， 就可以进行重放攻击了。

在session里加一个（nonce）暂时标志（随机值）可以解决重放攻击这个问题。一个nonce仅有效一次，并且服务端保持对所有有效nonce的跟
踪，如果你有多个应用服务器，它可能甚至会更复杂，避免把nonce存储在数据库里，这样会破坏整个CookieStore。

最好的解决方法是不要存放这种数据在session里，而是在数据库里。这个例子，存储credit在数据库里，logged in user id 放在session里。

#### 2.1.7 session 定制攻击

- 除了窃取用户的session id，攻击者还可能去定制一个session id来伪装，这就是所谓的session定制。

![](http://guides.rubyonrails.org/images/session_fixation.png)
这个攻击的重点是定制一个用户的可以识别他的session id，并且强制用户的浏览器用这个id。因此后来攻击者也不需要去窃取一个session id了。来看看攻击是如何展开的：

1.攻击者创建一个有效的session id： 他登陆想定制session的那个web应用的登陆页面，这时就通过response在cookie里有一个session id了（图中的1，2）

2.他可能保持这次会话（session）。session是会过期的，比如每隔20分钟，这会大大的减少了攻击者的时间。因此，他会不断的访问这个web应用以便于保持session可用。

3.现在，攻击者会强迫用户来使用这个session id（看图里的3），因为你不能改变另一个域的cookie，攻击者不得不在目标web应用里运行一个javascript脚本。通过往目标web应用注入javascript代码来完成这次攻击。下面是一个例子：
    <script> document.cookie="_session_id=16d5b78abb28e3d6206b60f22a03c8d9"; </script>

4. 攻击者引诱受害人向被感染的网页上的JavaScript代码。通过查看网页时，受害者的浏览器将把session id改变成攻击者定制好的session id。
5.因为这个陷进session id没有被用，web应用会需要用户去验证。
6.从现在开始，受害者和攻击者都会共有一个相同的session在这个web应用里。此时受害者并没有注意到这次攻击。

2.8  session 定制 －－－ 对策
－ 一行代码就保护你不被session定制侵害。
最有效的对策就是在成功登陆以后，创建一个新的session id，并且宣告旧的session id失效。那样，攻击者就不能用固定的session id了。这是一个对方session劫持的好的对策。Rails里我们可以这样来产生一个新的session：
reset_session
如果你用流行的rails插件，RestfulAuthentication，请在SessionsController#create action里加 reset_session.  注意，这个行为消除了原session里存储的所有值，你必须把这些值得转移到新的session里。

另
一个有效的对策是，在session里保存用户的特定属性，在每次请求进来的时候验证他们，如果信息不匹配，则拒绝访问。可以用远程ip地址，或者是访问
浏览器名称，尽管后者是不常用的。当保持ip的时候要注意，应该保持用户真实ip 而不是互联网服务供应商或大型组织的ip，如果这些变化以后，某些用户
可能无法使用这个web应用，或者会有限制。

2.9 session 过期
－ 永不过期的session会更加方便这些session攻击。

session过期时间在服务端设置更加安全。下面是一个例子， 如何在数据库里设置session的过期时间。如果超过20分钟就调用Session.swap(20m)：
class Session < ActiveRecord::Base
 def self.sweep(time_ago = nil)
 time = case time_ago
 when /^(d+)m$/ then Time.now - $1.to_i.minute
 when /^(d+)h$/ then Time.now - $1.to_i.hour
 when /^(d+)d$/ then Time.now - $1.to_i.day
 else Time.now - 1.hour
 end
 self.delete_all "updated_at < '#{time.to_s(:db)}'"
 end
 end

这节讲述了session定制需要保持连接。虽然你有过期时间，但攻击者也不傻，
他会保持每隔五分钟使session可用，不过期。一个简单的解决方法是，增加一个created_at到你的session表里。现在你可以删除被创建
了很久的那些session，用下面的这行替换上面swap方法的：

self.delete_all "updated_at < '#{time.to_s(:db)}' OR created_at < '#{2.days.ago.to_s(:db)}'"

## reference

http://article.yeeyan.org/bilingual/17603


[Rails每周一题(七)：Security Guide（中）](http://rails-weekly.group.iteye.com/group/wiki/1815-rails-questions-weekly-7-security-guide-two)

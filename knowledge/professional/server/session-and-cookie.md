---
layout: master
title: session and cookie
---

##1 session

session就是一系列某用户和服务器间的通讯。服务器有能力分辨出不同的用户。一个session的建立是从一个用户向服务器发第一个请求开始，而以用户显式结束或session超时为结束。 

session被用于表示一个持续的连接状态，在网站访问中一般指代客户端浏览器的进程从开启到结束的过程。session其实就是网站分析的访问（visits）度量，表示一个访问的过程。


###1.1 session的常见实现形式

session的常见实现形式是会话cookie（session cookie），即未设置过期时间的cookie，这个cookie的默认生命周期为浏览器会话期间，只要关闭浏览器窗口，cookie就消失了。

###1.2 实现机制

session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。

- 当用户发起一个请求的时候，服务器会检查该请求中是否包含sessionid，如果未包含，则系统会创造一个名为JSESSIONID的输出 cookie返回给浏览器(只放入内存，并不存在硬盘中)，并将其以HashTable的形式写到服务器的内存里面；session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个 session id将被在本次响应中返回给客户端保存。
- 当已经包含sessionid时，服务端会检查找到与该session相匹配的信息，如果存在则直接使用该sessionid，若不存在则重新生成新的 session。这里需要注意的是session始终是由服务端创建的，并非浏览器自己生成的。
 
这种机制不使用IP作为标识，是因为很多机器是通过代理服务器方式上网，没法区分每一台机器。 

对于**session标识号**（sessionID）保存，客户端有两种**方式实现**：

- cookies：保存这个session id的方式可以采用cookie，这样在交互过程中浏览器可以自动的按照规则把这个标识发挥给服务器。一般这个cookie的名字都是类似于SEEESIONID
- URL重写。 由于cookie可以被人为的禁止，必须有其他机制以便在cookie被禁止时仍然能够把session id传递回服务器。经常被使用的一种技术叫做URL重写，就是把session id直接附加在URL路径的后面，附加方式也有两种，**一种是作为URL路径的附加信息**，表现形式为http://...../xxx;jsessionid= ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764
**另一种是作为查询字符串附加在URL后面**，表现形式为http://...../xxx?jsessionid=ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764
这两种方式对于用户来说是没有区别的，只是服务器在解析的时候处理的方式不同，采用第一种方式也有利于把session id的信息和正常程序参数区分开来。
为了在整个交互过程中始终保持状态，就必须在每个客户端可能请求的路径后面都包含这个session id。

但是浏览器的cookie被禁止后session就需要用get方法的URL重写的机制或使用POST方法提交隐藏表单的形式来实现。

###1.3 session失效时间的设置

这里要分两方面来看：浏览器端和服务端。

- 对于浏览器端而言，session与访问进程直接相关，当浏览器被关闭时，session也随之消失；
- 而服务器端的session失效时间一般是人为设置的，目的是能定期地释放内存空间，减小服务器压力，一般的设置为当会话处于非活动状态达20或30分钟时清除该 session
- 所以浏览器端和服务端的session并非同时消失的，session的中断也并不一定意味着用户一定离开了该网站。目前Google Analytics和Omniture都定义当间隔30分钟没有动作时，算作一次访问结束，所以session的最后一步不只是离开，也有可能是静止、休眠或者发呆的状态。

**需要注意**

现在的浏览器好像趋向于多进程的session共享，即通过多个标签或页面打开多个进程访问同一网站时共享一个 session cookie，只有当浏览器被关闭时才会被清除，也就是你有可能在标签中关闭了该网站，但只要浏览器未被关闭并且在服务器端的session未失效前重新开启该网站，那么就还是使用原session进行浏览；

- 而某些浏览器在打开多页面时也可能建立独立的session，IE8、Chrome默认都是共享 session的
- 在IE8中可以通过菜单栏中的文件->新建会话来建立独立session的浏览页面。

##2 cookie 

cookie 是一**小段文本信息**，伴随着用户请求和页面在Web服务器和浏览器之间传递。用户每次访问站点时，Web应用程序都可以读取cookie包含的信息。


###2.1 持久cookie（persistent cookies）

session的实现机制里面已经介绍了常见的方法是使用会话cookie（session cookie）的方式

平常所说的cookie主要指的是另一类cookie——**持久cookie（persistent cookies）**。

持久cookie是指存放于客户端硬盘中的 cookie信息（设置了一定的有效期限）

###2.2 实现机制

- 当用户访问某网站时，浏览器就会在本地硬盘上查找与该网站相关联的cookie。如果该cookie 存在，浏览器就将它与页面请求一起通过HTTP报头信息发送到您的站点，
- 然后在系统会比对cookie中各属性和值是否与存放在服务器端的信息一致，并根据比对结果确定用户为“初访者”或者“老客户”。

持久cookie一般会保存用户的用户ID，该信息在用户注册或第一次登录的时候由服务器生成包含域名及相关信息的cookie发送并存放到客户端的硬盘文件上，并设置cookie的过期时间，以便于实现用户的自动登录和网站内容自定义。

Apache自带的mod_usertrack模块可以在用户首次来到当前网站的时候给用户种下一个唯一的cookie（较长时间过期），这个 cookie是用户首次来当前网站的IP地址加上一个随机字符串组成的。同时在自定义WEB日志中在最后增加%{cookie}n字段可以实现 cookie在apache日志中的输出，用于数据统计与用户跟踪。

##3 cookie和session区别 

具体来说cookie机制采用的是在**客户端保持状态的方案**。它是在**用户端的会话状态的存贮机制**，他需要用户打开客户端的cookie支持。cookie的作用就是为了解决HTTP协议无状态的缺陷所作的努力.

而session机制采用的是一种在**客户端与服务器之间保持状态的解决方案**。同时我们也看到，由于采用服务器端保持状态的方案在客户端也需要保存一个标识，所以session机制可能需要借助于cookie机制来达到保存标识的目的。而session提供了方便管理全局变量的方式


##4 Others 

1. HTTP协议本身是“连接-请求-应答-关闭连接”模式的，是一种无状态协议（HTTP只是一个传输协议）； 
2. Cookie规范是为了给HTTP增加状态跟踪用的，但不是唯一的手段； 
3. 所谓Session，指的是客户端和服务端之间的一段交互过程的状态信息（数据）；这个状态如何界定，生命期有多长，这是应用本身的事情； 
4. 由于B/S计算模型中计算是在服务器端完成的，客户端只有简单的显示逻辑，所以，Session数据对客户端应该是透明的不可理解的并且应该受控于服务端；Session数据要么保存到服务端（HttpSession），要么在客户端和服务端之间传递（Cookie或url rewritting或Hidden input）； 
5. 由于HTTP本身的无状态性，服务端无法知道客户端相继发来的请求是来自一个客户的，所以，当**使用服务端HttpSession存储会话数据**的时候客户端的每个请求都应该包含一个**session的标识**(sid, jsessionid 等等)来告诉服务端； 
6. 会话数据保存在服务端（如HttpSession）的好处是**减少了HTTP请求的长度**，提高了网络传输效率；客户端session信息存储则相反； 
7. 客户端Session存储只有一个办法：cookie(url rewritting和hidden input因为无法做到持久化，不算，只能作为交换session id的方式，即a method of session tracking)，
而服务端做法大致也是一个道理：容器有个session管理器（如tomcat的org.apache.catalina.session包里面的类），提供session的生命周期和持久化管理并提供访问session数据的api； 
8. 使用服务端还是客户端session存储要看应用的实际情况的。一般来说**不要求用户注册登录的公共服务系统**（如google）采用cookie做客户端session存储（如google的用户偏好设置），而有**用户管理的系统**则使用服务端存储。原因很显然：无需用户登录的系统唯一能够标识用户的就是用户的电脑，换一台机器就不知道谁是谁了，服务端session存储根本不管用；而有用户管理的系统则可以通过用户id来管理用户个人数据，从而提供任意复杂的个性化服务； 
9. 客户端和服务端的session存储在性能、安全性、跨站能力、编程方便性等方面都有一定的区别，而且优劣并非绝对（譬如TheServerSide号称不使用HttpSession，所以性能好，这很显然：一个具有上亿的访问用户的系统，要在服务端数据库中检索出用户的偏好信息显然是低效的，Session管理器不管用什么数据结构和算法都要耗费大量内存和CPU时间；而用cookie，则根本不用检索和维护session数据，服务器可以做成无状态的，当然高效）； 
10. 所谓的“会话cookie”简单的说就是没有明确指明有效期的cookie，仅在浏览器当前进程生命期内有效，可以被后继的Set-Cookie操作清除掉



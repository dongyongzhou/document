---
layout: master
title: rails-url-info
---

##1 参考资料

[rails-in-some-of-the-ways-to-obtain-information-url](http://zool.me/rails/rails-in-some-of-the-ways-to-obtain-information-url/)

##2 Details 


打开的uri为 /post/Hello-World

fullurl为 http://test.blog.zool.it:3000/post/Hello-world

则rails的路由生成一下几个方法

###2.1 取得域名

domain(tld_length = 1)

###2.2 取得子域名
request.domain #=>  zool.it request.domain(2) #=> blog.zool.it 

subdomain(tld_length = 1)

subdomains(tld_length = 1)


request.subdomain #=>  "test.blog" request.subdomain(2) #=> "test" request.subdomain #=>  ["test", "blog"] request.subdomain(2) #=> ["test"] 

###2.3 取得主机名

host()


request.host #=> "test.blog.zool.it" 

###2.4 取得带端口的主机名

host_with_port()

request.host_with_port #=> "test.blog.zool.it:3000" 

###2.5 代理服务器的主机名和端口

raw_host_with_port()

request.raw_host_with_port #=> "test.blog.zool.it:3000" 

###2.6 取得由raw_host_with_port()获得的端口数值

port()

request.port #=> 3000 

###2.7 取得raw_host_with_port()获得的端口文本字符串

port_string()

request.port_string #=> ":3000" 

###2.8 取得当前使用网络协议

protocol()

request.protocol #=> "http://" 

###2.9 取得网络协议

scheme()

request.scheme #=> "http" 

###2.10 request请求的uri地址

request_uri()

request.request_uri #=> "/posts/Hello-World" 
server_port

###2.11 取得由env['SERVER_PORT']返回的端口值

server_port()

request.server_port #=> "3000" 

###2.12 当前是否在是用https加密协议

ssl?()

request.ssl?() #=> false 

###2.13 返回网络协议标准端口(http为80, https为443)

standard_port()

request.standard_port #=> 80 

###2.14 判断当前协议是否是标准端口

standard_port?()

request.standard_port? #=> false 

###2.15 取得当前requset完整url

url()

request.url #=> "http://test.blog.zool.it:3000/posts/Hello-World"



---
layout: master
title: Rails download
---

## Overview

download

Questions

If you store some files (say some static PDFs) in your public directory, is there a way that someone who isn't authorized to view those files, can view them by typing in a url like example.com/public/static_document.pdf? If so, can you disable this in Rails?


If I upload files to my server and thus have clients/customers download these files. How may I restrict the access of the file?

Such as, if I upload a file to www.domain.com/files/download.zip

And if the user has correct permissions he can download the file, but what if the user knows the direct link to the file itself?

Cause I can imagine how to not show the link to the user on the site if they don't have permission to see the link, but how do I prevent someone from just typing in the direct URL of the location of the file to download the file?



## Basics

不要把上传文件置于Rails的public目录下 -- 如果这个目录是Apache的home目录。

Apache Web服务器有个特点，就是它会执行DocumentRoot目录下的有特定扩展名的文件，比如PHP和CGI文件。

假设攻击者上传了一个file.cgi文件，那么当他下载这个文件时，这个文件就会被执行。


如果Rails的public目录是Apache的home目录，就不要在此目录下存储上传文件。至少把文件存于下一级目录。


The **public** folder contains the static files and compiled assets for the client to read. The folder by default is accessible to anyone visiting your site. 



## work

— 不要让用户下载任意文件。
 
Ruby代码  

	send_file('/var/www/uploads/' + params[:filename])  
 
上面这段代码可以让用户下载任意文件。假如用户输入的文件名是 “../../../etc/passwd”，那么passwd文件就会被下载。
 
### 应对策略：
 

验证被下载文件在合法的目录下
 
Ruby代码  

	basename = File.expand_path(File.join(File.dirname(__FILE__), '../../files'))   
	filename = File.expand_path(File.join(basename, @file.public_filename))   
	raise if basename =! File.expand_path(File.join(File.dirname(filename), '../../../'))   
	send_file filename, :disposition => 'inline'   





### Problem


	AbstractController::DoubleRenderError (Render and/or redirect were called multiple times in this action. Please note that you may only call render OR redirect, and at most once per action. Also note that neither redirect nor render terminate execution of the action, so if you want to exit an action after redirecting, you need to do something like "redirect_to(...) and return".):

  app/controllers/download_controller.rb:9:in `download'


send_file' that is a render in it's self. So displaying a 'success' page and sending a file is two separate renders.

I would make the success page and file download two separate controller actions, and display a link to the file download on the success page.


## Reference



[Rails Routing from the Outside In](http://guides.rubyonrails.org/routing.html)
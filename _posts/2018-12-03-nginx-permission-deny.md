---
layout: post
title: nginx运行出现failed (13:Permission denied) while reading upstream
categories: nginx
description: 个人对js中原型及原型链的理解
keywords: nginx 服务器 填坑达人
---

## 问题 
在用nginx反向道理到文件的时候，一直出现403的问题，看了一下配置文件nginx.conf下的root路径，发现路径是正确的，但还是一直显示403，后来看了一下:
```
/usr/local/var/log/nginx/error.log
``` 
的日志，发现一直显示以下错误：  
![errlog](http://binzhome.com/assets/images/others/log.jpg)

最终定位原因应该由于当前nginx用户权限不足导致的。 

## 解决方案  

由于本人目前是本地开发，所以下面的流程都是mac本地环境下的配置，正式服务器的话也可以按照这个思路来。  
首先确定当前用户的群组，在shell中输入：  
```
groups
``` 
我目前用的群组是admin

进入nginx的配置文件:  
```
vim /usr/local/etc/nginx/nginx.conf
``` 

打开文件后看到第一行：
```
# user nobody;
```
将该行注释去掉，修改为你当前对应的群组与用户：
```
user yourusername yourusergroup;
```
最后保存后重启nginx服务器：
```
sudo nginx -s reload
```
问题解决。

## 思考 
以下是nginx配置文档中原文：
```
Syntax:	user user [group];
Default: user nobody nobody;

Defines user and group credentials used by worker processes. If group is omitted, a group whose name equals that of user is used.
```
可以看出，这个是定义了当前进程的所属用户，当我们为普通用户的时候，由于权限问题导致无法访问nginx设置的root文件，并且如果省略group时，group将默认为user相同的配置。
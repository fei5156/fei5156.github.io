---
layout: post
title: openresty lua的使用（一）
categories: Linux
description: linux，openresty，lua
keywords: linux，openresty，lua

---

#openresty lua的使用（一）环境搭建

##说在前面的话
第一次写博客也是第一次使用markdown来编辑，总之，都是一个学习的过程，因为工作中使用了lua这种脚本语言，所以就分享记录一下自己的熟悉过程，工作中使用为主，很少接触到linux中的安装配置的过程，一般只是在windows环境中使用,废话不多说了,直接进入正题.

##一、准备
- Linux环境，我这里是在虚拟机安装的CentOS 7.
- Openresty lua 安装包

##二、安装

安装步骤总结:
1.下载安装包,[下载地址](http://openresty.org/cn/installation.html)
2.上传至服务器,不用纠结目录
3.安装前服务器需先安装环境 
```
yum install readline-devel pcre-devel openssl-devel gcc curl perl
```
4.解压安装包
5.打开进入解压后的目录
6.配置configure,可以直接复制如下设置:

```
./configure \
             --with-cc-opt="-I/usr/local/include" \
             --with-ld-opt="-L/usr/local/lib" \
             --prefix=/usr/local/openresty
``` 


如设置--prefix 指定安装目录
7.上一步没error的话按照提示输入 
````
gmake 
gmake install
````

8.安装完成即可进入安装目录下查看

9.配置配置文件,可以先简单设置 如打印hello world 或者使用默认配置启动,在安装目录/usr/local/openresty/nginx/conf/nginx.conf

10.启动 openresty: 
```
/usr/local/openresty/nginx/sbin/nginx -c /usr/local/openresty/nginx/conf/nginx.conf -p /usr/local/openresty/nginx/
```
-c 指定配置文件 -p 指定文件目录 如lua文件存放目录

11.如果觉得每次前面太长可以设置环境变量 vi  /etc/profile

PATH=/usr/local/openresty/nginx/sbin:$PATH
export PATH
或者
export PATH=/usr/local/openresty/nginx/sbin:$PATH

``source /etc/profile`` 重新加载设置

12
测试  curl 127.0.0.1

13.如果修改了文件 可以使用重新加载文件
```
/usr/local/openresty/nginx/sbin/nginx -c /usr/local/openresty/nginx/conf/nginx.conf -p /usr/local/openresty/nginx/ -s reload
```

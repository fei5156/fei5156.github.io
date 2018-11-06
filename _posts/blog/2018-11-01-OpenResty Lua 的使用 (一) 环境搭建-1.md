---
layout: post
title: OpenResty Lua 的使用 (一) 环境搭建
categories: Nginx
description: Linux，OpenResty，Lua，Nginx
keywords: Linux，OpenResty，Lua，Nginx
---

OpenResty Lua 的使用 (一) 环境搭建

## 前言

## 一、准备

* Linux环境，我这里是在虚拟机安装的CentOS 7。
* Openresty lua 安装包。

## 二、安装 

1. 如果没有 `wget` 命令的话可以执行如下操作进行安装。
 `yum -y install wget`

2. 安装执行环境。
  `yum install readline-devel pcre-devel openssl-devel gcc curl perl`

3. 为了方便我在opt下新建文件夹用以放下载的文件和备份文件，/opt/download,/opt/backup，在主目录新建/data/,并新建/data/nginx,/data/nginxlog,/data/software。

4. 下载OpenResty Lua的安装包，可以在Win系统下载后上传，也可以直接在服务器下载。[[中文下载地址](http://openresty.org/cn/download.html)],[[英文地址](https://openresty.org/en/download.html)]，选择对应版本进行下载。
```
可以直接执行在服务器下载：wget https://openresty.org/download/openresty-${VERSION}.tar.gz 替换成你要下载的版本即可。
cd /opt/download 
wget https://openresty.org/download/openresty-1.13.6.2.tar.gz  
```
5. 进入cd /data/software，下载一些一会需要的模块和依赖的软件。
  + 下载ngx_cache_purge模块，该模块用于清理nginx缓存
    - `wget https://github.com/FRiCKLE/ngx_cache_purge/archive/2.3.tar.gz`
  + 下载nginx_upstream_check_module模块，该模块用于ustream健康检查
    - `wget https://github.com/yaoweibin/nginx_upstream_check_module/archive/v0.3.0.tar.gz`
    - 也可以到GitHub主页自己打包下载zip，然后上传解压，https://github.com/yaoweibin/nginx_upstream_check_module ，
  + 下载openssl-1.0.2l 模块,可以到 https://www.openssl.org/source/ 下载最新版，https://ftp.openssl.org/source/old/1.0.2/ 下载旧版
    - `wget https://www.openssl.org/source/openssl-1.1.1.tar.gz`
    - `wget https://ftp.openssl.org/source/old/1.0.2/openssl-1.0.2l.tar.gz`
  + 下载pcre，https://ftp.pcre.org/pub/pcre/
    - `wget https://ftp.pcre.org/pub/pcre/pcre-8.41.tar.gz`
  + 安装drizzle模块(访问mysql数据库模块，非必需，建议安装)
    - `wget http://agentzh.org/misc/nginx/drizzle7-2011.07.21.tar.gz`
  + 下载nginx-http-concat(合并静态文件请求模块，非必需，建议安装)
    - `wget https://github.com/alibaba/nginx-http-concat/archive/master.zip`
  + 安装Zlib
    - `wget https://zlib.net/fossils/zlib-1.2.8.tar.gz`
  + 下载luajit，使用OpenResty可以不安装
    - `wget http://luajit.org/download/LuaJIT-2.0.5.tar.gz`

6.  解压上面下载的几个文件。
   `tar -zxvf 2.3.tar.gz`
   `tar -zxvf openssl-1.0.2l.tar.gz`
   `tar -zxvf v0.3.0.tar.gz`
   `tar -zxvf pcre-8.41.tar.gz`
   `tar -zxvf drizzle7-2011.07.21.tar.gz`
   `unzip master` 
    如果没有unzip命令的话执行：`yum install -y unzip zip` 安装
    `tar -zxvf zlib-1.2.8.tar.gz`
    `tar -zxvf LuaJIT-2.0.5.tar.gz`
    ...
    ```
    安装PCRE
        yum install -y gcc gcc-c++
        cd pcre-8.41
        ./configure
        make
        make install
    安装openSSL
        cd openssl-1.0.2l 
        设定Openssl 安装，( --prefix )参数为欲安装之目录，也就是安装后的档案会出现在该目录下：
        ./config --prefix=/usr/local/openssl
        ./config -t
        make
        make install 
    安装drizzle模块   
        cd drizzle7-2011.07.21  
        ./configure --without-server  
        make libdrizzle-1.0  
        make install-libdrizzle-1.0 
    安装Zlib
        cd zlib-1.2.8
        ./configure    --prefix=/data/progam/zlib
        make
        make install
        再进行配置一下系统的文件，加载刚才编译安装的zlib生成的库文件
        vi /etc/ld.so.conf.d/zlib.conf
        加入如下内容后保存退出: /data/program/zlib/lib
        也就是添加安装目录的文件路径，库文件。ldconfig  运行之后就会加载安装的库文件了。
    注意使用openresty不用安装luajit，忽略此步
        cd LuaJIT-2.0.3 
        make && make install PREFIX=/usr/local/luajit
        export LUAJIT_LIB=/usr/local/luajit/lib 
        export LUAJIT_INC=/usr/local/luajit/include/luajit-2.0
    ```

7. 进入cd /opt/download 解压文件。
  `tar -zxvf openresty-1.13.6.2.tar.gz`

8. 进入解压后的目录。
  `cd openresty-1.13.6.2`

9. 可以看到存在一个名为configure的可执行脚本程序。它是用于检查系统是否有编译时所需的库，以及库的版本是否满足编译的需要等安装所需要的系统信息。为随后的编译工作做准备。在当前文件夹下执行如下设置，注意不能有换行,注意版本，注意刚刚下载的几个模块路径需要修改：
```
两种写法： 都可以，第二种是每行后面一个/ 代表不换行
./configure --prefix=/usr/local/openresty --with-debug --add-module=/data/software/ngx_cache_purge-2.3 --add-module=/data/software/nginx_upstream_check_module-0.3.0 --with-http_sub_module --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module --with-pcre-jit --with-http_v2_module --with-openssl=/data/software/openssl-1.0.2l --with-pcre=/data/software/pcre-8.41 --with-http_gzip_static_module --with-http_flv_module --with-stream --with-stream_ssl_module

 ./configure --prefix=/usr/local/openresty \
  --with-debug \
  --add-module=/data/software/ngx_cache_purge-2.3 --add-module=/data/software/nginx_upstream_check_module-0.3.0 --with-http_sub_module --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module --with-pcre-jit --with-http_v2_module --with-openssl=/data/software/openssl-1.0.2l --with-pcre=/data/software/pcre-8.41 --with-http_gzip_static_module --with-http_flv_module --with-stream --with-stream_ssl_module --with-stream --with-stream_ssl_module
```
```
如下这些参数会默认加载 不需要另外操作，如果加上的话重复加载模块，容易出错，同时也会出现找不到config文件的请客，我也没找到怎么处理，所及就没加。
  --with-cc-opt='-DNGX_LUA_USE_ASSERT -DNGX_LUA_ABORT_AT_PANIC -O2' \
  --add-module=../ngx_devel_kit-0.3.0 \
  --add-module=../echo-nginx-module-0.61 \
  --add-module=../xss-nginx-module-0.06 \
  --add-module=../ngx_coolkit-0.2rc3 \
  --add-module=../set-misc-nginx-module-0.32 \
  --add-module=../form-input-nginx-module-0.12 \
  --add-module=../encrypted-session-nginx-module-0.08 \
  --add-module=../srcache-nginx-module-0.31 \
  --add-module=../ngx_lua-0.10.13 \
  --add-module=../ngx_lua_upstream-0.07 \
  --add-module=../headers-more-nginx-module-0.33 \
  --add-module=../array-var-nginx-module-0.05 \
  --add-module=../memc-nginx-module-0.19 \
  --add-module=../redis2-nginx-module-0.15 \
  --add-module=../redis-nginx-module-0.3.7 \
  --add-module=../rds-json-nginx-module-0.15 \
  --add-module=../rds-csv-nginx-module-0.09 \
  --add-module=../ngx_stream_lua-0.0.5 \
  --with-ld-opt='-Wl,-rpath,/usr/local/openresty/nginx/luajit/lib' \
```
 可以在测试环境打开debug，使用`--with-debug` ，nginx默认是info以上的。
有的地方看到-j2参数，应该是说明电脑支持多核工作，假设是双核，可以使用-j2。
其他具体参数介绍可以看之前的一篇文章[[^Nginx的使用之编译配置参数]]

10. 如果执行出错说明配置的参数有问题，一般是可能版本与安装的不对应， 具体原因可以看源码包目录下的 build/nginx-VERSION/objs/autoconf.err文件查看。上一步没出错误的话，根据提示执行即可。

11. 执行
   `gmake`  
   `gmake install`

## 三、启动


## 结束



[^Nginx的使用之编译配置参数]:http://cangin.top/2018/11/01/Nginx%E5%8F%82%E6%95%B0%E7%9A%84%E8%AF%B4%E6%98%8E-1/
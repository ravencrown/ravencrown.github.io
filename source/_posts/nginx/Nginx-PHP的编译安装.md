---
layout: post
title: Nginx+PHP的编译安装
date: 2016-12-18 22:45:12
tags: Nginx
categories: Nginx
---

## 安装依赖包

```shell
yum install gd gd-devel
yum install ttf
yum install freetype
yum install mysql mysql-server
yum install libvpx libjpeg libpng zlib libXpm libXpm-devel FreeType t1lib  libt1-devel -y
```
  
nginx+php的配置比较简单,核心就一句话----
把请求的信息转发给9000端口的PHP进程,
让PHP进程处理 指定目录下的PHP文件.




## PHP的编译

apache一般是把php当做自己的一个模块来启动的.
而nginx则是把http请求变量(如get,user_agent等)转发给 php进程,即php独立进程,与nginx进行通信. 称为 fastcgi运行方式.
因此,为apache所编译的php,是不能用于nginx的.  
  
注意: 我们编译的PHP 要有如下功能:
连接mysql, gd, ttf, 以fpm(fascgi)方式运行  

```shell
./configure  --prefix=/usr/local/fastphp \
--with-mysql=mysqlnd \
--enable-mysqlnd \
--with-gd \
--enable-gd-native-ttf \
--enable-gd-jis-conv
--enable-fpm

cd /usr/local/fastphp
cp /usr/local/src/php-5.5.16/php.ini-development ./lib/php.ini #制作php.ini文件
cp etc/php-fpm.conf.default etc/php-fpm.conf                   #制作fpm文件
./sbin/php-fpm
```

## PHP命令

```shell
#php-fpm 启动：
/usr/local/php/sbin/php-fpm
#php-fpm 关闭：
kill -INT `cat /var/run/php-fpm/php-fpm.pid`
#php-fpm 重启：
kill -USR2 `cat /var/run/php-fpm/php-fpm.pid`
#批量删除进程
kill -9  ps aux | grep php-fpm | awk '{print $2}'
```

如下例子

```shell
#1:碰到php文件,
#2: 把根目录定位到 html,
#3: 把请求上下文转交给9000端口PHP进程,
#4: 并告诉PHP进程,当前的脚本是$document_root$fastcgi_scriptname
#(注:PHP会去找这个脚本并处理,所以脚本的位置要指对)
location ~ \.php$ {
    root html;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include    fastcgi_params;
}
```


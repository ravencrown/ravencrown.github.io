---
layout: post
title: Nginx日志管理
date: 2016-11-27 17:45:03
tags: Nginx
categories: Nginx
---

## Nginx 允许针对不同的server做不同的log
 
我们观察nginx的server段,可以看到如下类似信息

```shell
access_log  logs/host.access.log  main;
```

这说明 该server, 它的访问日志的文件是  logs/host.access.log ,
使用的格式”main”格式.
除了main格式,你可以自定义其他格式.
 
main格式是什么?

```shell
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';
```


main格式是我们定义好一种日志的格式,并起个名字,便于引用.
以上面的例子, main类型的日志,记录的 remote_addr.... http_x_forwarded_for等选项.
 
 
### 日志格式 是指记录哪些选项
默认的日志格式: main

```shell
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" '
	'"$http_user_agent" "$http_x_forwarded_for"';
```

如默认的main日志格式,记录这么几项
远程IP- 远程用户/用户时间 请求方法(如GET/POST) 请求体body长度 referer来源信息
http-user-agent用户代理/蜘蛛 ,被转发的请求的原始IP
 
http_x_forwarded_for:在经过代理时,代理把你的本来IP加在此头信息中,传输你的原始IP

例如：

```shell
58.132.169.60 - - [07/Oct/2015:11:40:19 +0800] "GET / HTTP/1.1" 200 30 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.10; rv:41.0) Gecko/20100101 Firefox/41.0" "-"
```

### 声明一个独特的log_format并命名

```shell
log_format  mylog '$remote_addr- "$request" '
 	'$status $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';
```

在下面的server/location,我们就可以引用 mylog
 
在server段中,这样来声明
Nginx允许针对不同的server做不同的Log ,(有的web服务器不支持,如lighttp)

```shell
access_log logs/access_8080.log mylog;  
#声明log   log位置          log格式;
```

## 参考链接

- Nginx官网, [Module ngx_http_log_module](http://nginx.org/en/docs/http/ngx_http_log_module.html)

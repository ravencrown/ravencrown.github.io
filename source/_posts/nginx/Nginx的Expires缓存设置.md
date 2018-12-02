---
layout: post
title: Expires缓存设置
date: 2016-12-26 20:43:23
tags: Nginx
categories: Nginx
---

## Expires 设置

对于网站的图片,尤其是新闻站, 图片一旦发布, 改动的可能是非常小的.我们希望 能否在用户访问一次后, 图片缓存在用户的浏览器端,且时间比较长的缓存.
可以, 用到 nginx的expires设置 .
nginx中设置过期时间,非常简单,
在location或if段里,来写.

格式

```shell
expire -1;
expire 0;
expire 1h;
expire max;
expire off;
```

这个指令表示在HTTP响应头中是否增加或修改Expires 和 Cache-Control ，仅当响应状态是200, 201, 204, 206, 301, 302, 303, 304, 或者 307的时候有效。  
  
当expire 为负时，会在响应头增加Cache-Control: no-cache；  
  
当为正或者0时，就表示Cache-Control: max-age=指定的时间(秒)；  
  
当为max时，会把Expires设置为 “Thu, 31 Dec 2037 23:55:55 GMT”， Cache-Control 设置到 10 年；  
  
当为off时，表示不增加或修改Expires 和 Cache-Control 。  
  
例如

```shell
$ curl -I http://su.bdimg.com/static/superplus/css/super_min_2b5190eb.css
HTTP/1.1 200 OK
Server: JSP3/2.0.4
Date: Fri, 31 Oct 2014 07:28:20 GMT
Content-Type: text/css
Content-Length: 25076
Connection: keep-alive
ETag: “1121644245”
Last-Modified: Wed, 24 Sep 2014 15:50:22 GMT
Expires: Mon, 23 Mar 2015 16:29:25 GMT
Age: 3164335
Cache-Control: max-age=15552000
Accept-Ranges: bytes
Vary: Accept-Encoding
```

Date 的意思是服务器发送消息的时间； Age 的意思有点复杂，它的存在暗示你访问的服务器不是源服务器，而是一台缓存服务器，Age 的大小表示这个资源已经”存活了”多长时间，所以这个值不会大于 源服务器设置的最大缓存时间。  
  
这里Expires 表示过期时间，Cache-Control 表示最大的存活时间，在服务器端的Nginx 我们可以用 expires 指令来定义这两项。  
  
另: 304 也是一种很好的缓存手段  
原理是: 服务器响应文件内容是,同时响应etag标签(内容的签名,内容一变,他也变), 和 last_modified_since 2个标签值  
浏览器下次去请求时,头信息发送这两个标签, 服务器检测文件有没有发生变化,如无,直接头信息返回 etag,last_modified_since  
浏览器知道内容无改变,于是直接调用本地缓存.  
这个过程,也请求了服务器,但是传着的内容极少.  
对于变化周期较短的,如静态html,js,css,比较适于用这个方式  




## 参考链接

- Nginx官网，[Location Expires](http://nginx.org/en/docs/http/ngx_http_headers_module.html#expires)

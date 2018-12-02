---
layout: post
title: Nginx的Gzip设置
date: 2016-12-29 23:02:24
tags: Nginx
categories: Nginx
---

## Gzip配置参数

```shell
gzip配置的常用参数
gzip on|off;  #是否开启gzip
gzip_buffers 32 4K| 16 8K #缓冲(压缩在内存中缓冲几块? 每块多大?)
gzip_comp_level [1-9] #推荐6 压缩级别(级别越高,压的越小,越浪费CPU计算资源)
gzip_disable #正则匹配UA 什么样的Uri不进行gzip
gzip_min_length 200 # 开始压缩的最小长度(再小就不要压缩了,意义不在)
gzip_http_version 1.0|1.1 # 开始压缩的http协议版本(可以不设置,目前几乎全是1.1协议)
gzip_proxied          # 设置请求者代理服务器,该如何缓存内容
gzip_types text/plain  application/xml # 对哪些类型的文件用压缩 如txt,xml,html ,css
gzip_vary on|off  # 是否传输gzip压缩标志 Vary是用来标志缓存的依据.

#注意:
#图片/mp3这样的二进制文件,不必压缩
#因为压缩率比较小, 比如100->80字节,而且压缩也是耗费CPU资源的.
#比较小的文件不必压缩,
```

**思考:**

- 如果2个人,一个浏览器支持gzip,一个浏览器不支持gzip2个同时请求同个页面, chinaCache缓存压缩后,还是未压缩的?
- 如果1人,再次请求页面,chinaCache返回压缩后的缓存内容,还是压缩前的缓存内容?
   
这个时候 Vary的作用体现出来.即------缓存的内容受 Accept-Encoding头信息的影响.  
  
所以如果  
请求时,不支持gzip, 缓存服务器将会生成一份未gzip的内容.  
请求时,支持gzip, 缓存服务器将会生成一份gzip的内容.  
  
下次再请求时, 缓存服务器会考虑客户端的Accept-Encoding因素,并合理的返回信息  
  
Nginx对于图片,js等静态文件的缓存设置  
注:这个缓存是指针对浏览器所做的缓存,不是指服务器端的数据缓存.  
   
主要知识点: location expires指令  

```shell
location ~ \.(jpg|jpeg|png|gif)$ {
    expires 1d;
}

location ~ \.js$ {
    expires 1h;
}
```

设置并载入新配置文件,用firebug观察,  
会发现 图片内容,没有再次产生新的请求,原因--利用了本地缓存的效果.  

**注: 在大型的新闻站,或文章站中,图片变动的可能性很小,建议做1周左右的缓存Js,css等小时级的缓存.**

如果信息流动比较快,也可以不用expires指令,  
用last_modified, etag功能(主流的web服务器都支持这2个头信息)  

原理是:  
响应: 计算响应内容的签名, etag 和 上次修改时间  
请求: 发送 etatg, If-Modified-Since 头信息.  
服务器收到后,判断etag是否一致, 最后修改时间是否大于if-Modifiled-Since  
如果监测到服务器的内容有变化,则返回304,  
浏览器就知道,内容没变,直接用缓存.  
  

**304 比起上面的expires 指令多了1次请求,但是比200状态,少了传输内容.**

## 参考链接

- Nginx官网, [Nginx Gzip Module](http://nginx.org/en/docs/http/ngx_http_gzip_module.html)
- Nginx官网, [Nginx Gzip Static Module](http://nginx.org/en/docs/http/ngx_http_gzip_static_module.html)




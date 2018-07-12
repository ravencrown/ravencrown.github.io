---
layout: post
title: Nginx的Location语法
date: 2016-12-07 20:15:32
tags: Nginx
categories: Nginx
---

location 有”定位”的意思, 根据URI来进行不同的定位.
在虚拟主机的配置中,是必不可少的,location可以把网站的不同部分,定位到不同的处理方式上.
比如, 碰到.php, 如何调用PHP解释器?  --这时就需要location

## Location 的语法

location 有”定位”的意思, 根据Uri来进行不同的定位. 在虚拟主机的配置中,是必不可少的,location可以把网站的不同部分,定位到不同的处理方式上. 比如, 碰到.php, 如何调用PHP解释器? --这时就需要location location 的语法


```
Syntax:	location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
Default:	—
Context:	server, location

中括号可以不写任何参数,此时称为一般匹配,也可以写参数
因此,大类型可以分为3种:
location = patt {} [精准匹配]
location patt{}  [一般匹配]
location ~ patt{} [正则匹配]
```


location示例

```shell
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```

### 如何发挥作用?

首先看有没有精准匹配,如果有,则停止匹配过程.

``` shell
#如果 $uri == patt,匹配成功，使用configA
location = / {
    root   /var/www/html/;
    index  index.htm index.html;
}
    
  location / {
     root   /usr/local/nginx/html;
    index  index.html index.htm;
}
```

如果访问http://xxx.com/
定位流程是　

```
1: 精准匹配中　”/”   ,得到index页为　　index.htm
2: 再次访问 /index.htm , 此次内部转跳uri已经是”/index.htm” ,根目录为/usr/local/nginx/html
3: 最终结果,访问了/usr/local/nginx/html/index.htm
```

## 参考链接

- Nginx官网，[Location Syntax](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)




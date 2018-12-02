---
layout: post
title: Nginx的Location正则
date: 2016-12-12 10:15:34
tags: Nginx
categories: Nginx
---

## Location正则

```shell
location / {
    root   /usr/local/nginx/html;
    index  index.html index.htm;
}
 
location ~ image {
    root /var/www/image;
    index index.html;
}
```

如果我们访问  http://xx.com/image/logo.png  
此时, “/” 与”/image/logo.png” 匹配  
同时,”image”正则 与”image/logo.png”也能匹配,谁发挥作用?  
正则表达式的成果将会使用.  
 
图片真正会访问 /var/www/image/logo.png  


```shell
location / {
    root	/usr/local/nginx/html;
    index	index.html index.htm;
}
 
location /foo {
    root	/var/www/html;
    index	index.html;
}
```

我们访问http://xxx.com/foo  
 对于uri “/foo”,   两个location的patt,都能匹配他们  
即 ‘/’能从左前缀匹配 ‘/foo’, ‘/foo’也能左前缀匹配’/foo’,  
此时, 真正访问 /var/www/html/index.html  
原因:’/foo’匹配的更长,因此使用之.;  

**Location的正则写法**

```shell
location  = / {
    # 精确匹配 / ，主机名后面不能带任何字符串
    [ configuration A ] 
}

location  / {
    # 因为所有的地址都以 / 开头，所以这条规则将匹配到所有请求
    # 但是正则和最长字符串会优先匹配
    [ configuration B ] 
}

location /documents/ {
    # 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索
    # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
    [ configuration C ] 
}

location ~ /documents/Abc {
    # 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索
    # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
    [ configuration CC ] 
}

location ^~ /images/ {
    # 匹配任何以 /images/ 开头的地址，匹配符合以后，停止往下搜索正则，采用这一条。
    [ configuration D ] 
}

location ~* \.(gif|jpg|jpeg)$ {
    # 匹配所有以 gif,jpg或jpeg 结尾的请求
    # 然而，所有请求 /images/ 下的图片会被 config D 处理，因为 ^~ 到达不了这一条正则
    [ configuration E ] 
}

location /images/ {
    # 字符匹配到 /images/，继续往下，会发现 ^~ 存在
    [ configuration F ] 
}

location /images/abc {
    # 最长字符匹配到 /images/abc，继续往下，会发现 ^~ 存在
    # F与G的放置顺序是没有关系的
    [ configuration G ] 
}

location ~ /images/abc/ {
    # 只有去掉 config D 才有效：先最长匹配 config G 开头的地址，继续往下搜索，匹配到这一条正则，采用
    [ configuration H ] 
}

location ~* /js/.*/\.js
```

- 以=开头表示精确匹配如 A 中只匹配根目录结尾的请求，后面不能带任何字符串。
- ^~ 开头表示uri以某个常规字符串开头，不是正则匹配
- ~ 开头表示区分大小写的正则匹配;
- ~* 开头表示不区分大小写的正则匹配
- / 通用匹配, 如果没有其它匹配,任何请求都会匹配到

**顺序or优先级**

```shell
(location =) > (location 完整路径) > (location ^~ 路径) > (location ~,~* 正则顺序) > (location 部分起始路径) > (/)
```


**location解析过程如下图**  

![location解析过程](https://camo.githubusercontent.com/7df924e86acaa7b2d44f709bc86aec92d64444c0/687474703a2f2f692e696d6775722e636f6d2f655336424476622e706e67)

## 参考链接

- Nginx官网，[Location Syntax](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)



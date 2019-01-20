---
layout: post
title: CSS 根据设备像素比引用不同比例的图片
date: 2019-01-03 22:19:26
tags: CSS
categories: CSS
---

有时候会遇到需要根据设备像素比引用不同比例的图片（两倍或者三倍），代码如下
主要采用 `-webkit-min-device-pixel-ratio: 3` 这个属性

```css
bg-image ($url) {
    background-image: url('./img/' + $url + "@2x.png");
    @media (-webkit-min-device-pixel-ratio: 3), (min-device-pixel-ratio: 3) {
        background-image: url('./img/' + $url + "@3x.png");
    }
}
    
```



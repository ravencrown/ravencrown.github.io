---
layout: post
title: CSS 实现1px border
date: 2019-01-03 21:39:26
tags: CSS
categories: CSS
---

经常会用到 CSS 1px border 功能，代码如下

```css
border-1px ($color) {
    position: relative;
    &:after {
        display: block;
        position: absolute;
        left: 0;
        bottom: 0;
        width: 100%;
        border-top: 1px solid $color;
        content: ' ';
    }
}
```

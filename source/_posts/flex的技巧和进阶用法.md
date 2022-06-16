---
title: flex的技巧和进阶用法
author: DeepHug
index_img: /img/deep-hug-blog-wallpaper.JPG
category: 技术
tags:
  - Fluid
excerpt: 简介。。。
date: 2022-05-06 18:31:54
---

### 子元素继承父元素的高度

#### 课程表需求，手写表格的时候，需要让同一层的td具有相同的高度。就像下面的图片一样，如果我们直接使用 table 标签去做则不会有这个问题，如果用ul li这样去写就会一个无法设置子元素同一高度的问题。

<div>
    <img src="img2.png" />
</div>


> 只要父元素设置了 **display: flex** 同时 **ailgn-items** 属性值默认，那么子元素就会继承父元素的高度。
```js
    <div class="box ">
        <div class="d1">
            我是d1我没有设置高度
        </div>
        <div class="d2">
            我的d2我的高度有200px我的d2我的高度有200px我的d2我的高度有200px
        </div>
    </div>

    <style>
        .box {
            display: flex;
        }
        .d2 {
            width: 50px;
            background: peru;
        }
        .d1 {
            width:200px;
            background: pink;
        }
    </style>
```
<div>
    <img src="img1.png" />
</div>

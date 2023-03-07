---
title: ios输入框必须重压或长按才能唤起软键盘
author: DeepHug
index_img: /img/ios输入框必须重压或长按才能唤起软键盘.png
category: 技术
tags:
  - IOS 键盘 软键盘 输入框
excerpt: 最近做搜索框时发现，ios点击输入框之后，点击软键盘上的 完成 时发现，轻击input就无法唤起软键盘，无法对输入框聚焦，必须长按或重压才行，这边经过测试，发现应该是fastclick.js 引起的冲突，ios11 后修复了移动点击300ms延迟
date: 2021-12-28 15:45:01
---

### 为什么移动端点击事件要加300ms延迟呢？

早在 2007 年初，苹果公司在发布首款 iPhone 前夕，遇到一个问题：当时的网站都是为大屏幕设备所设计的。于是苹果的工程师们做了一些约定，应对 iPhone 这种小屏幕浏览桌面端站点的问题。

<div>
    <img src="ios.png" style="display: block;margin: 0 auto;" />
</div>

这当中最出名的，当属双击缩放(double tap to zoom)，这也是会有上述 300 毫秒延迟的主要原因。

双击缩放，顾名思义，即用手指在屏幕上快速点击两次，iOS 自带的 Safari 浏览器会将网页缩放至原始比例。 那么这和 300 毫秒延迟有什么联系呢？

<div>
    <img src="1.png" style="display: block;margin: 0 auto;" />
</div>

假定这么一个场景： 用户在 iOS Safari 里边点击了一个链接。由于用户可以进行双击缩放或者双击滚动的操作，当用户一次点击屏幕之后，浏览器并不能立刻判断用户是确实要打开这个链接，还是想要进行双击操作。因此，iOS Safari 就等待 300 毫秒，以判断用户是否再次点击了屏幕。 鉴于iPhone的成功，其他移动浏览器都复制了 iPhone Safari 浏览器的多数约定，包括双击缩放，几乎现在所有的移动端浏览器都有这个功能。那时人们刚刚接触移动端的页面，不会在意这个300ms的延时问题，可是如今移动端如雨后春笋，用户对体验的要求也更高，这300ms带来的卡顿慢慢变得让人难以接受。

<div>
    <img src="event.png" style="display: block;margin: 0 auto;" />
</div>

### 那么如何解决300ms延迟问题呢？

我们就推荐一种最有效、最方便的解决方案，大家应该都用过这个方法，那就是FastClick.js。

<div>
    <img src="FastClick.png" style="display: block;margin: 0 auto;" />
</div>

FastClick 是 FT Labs 专门为解决移动端浏览器 300 毫秒点击延迟问题所开发的一个轻量级的库。FastClick的实现原理是在检测到touchend事件的时候，会通过DOM自定义事件立即出发模拟一个click事件，并把浏览器在300ms之后的click事件阻止掉。

#### 如何使用FastClick

```js
    npm install fastclick -S
```

如何你是**vue**项目可以在main.js里面直接引入，当然这样是全局的，如果你需要某个页面用到，那就单个页面引入。

```js
    //引入
    import fastClick from 'fastclick'
    //初始化FastClick实例。在页面的DOM文档加载完成后
    fastClick.attach(document.body)
```

如果你用过FastClick在移动端，就会发现有一个体验很不好的问题，某些ios上，点击输入框想唤启软键盘，触点不是很灵敏，必须重压或长按才能成功唤启，快速点击是不会唤启软键盘的。

<div>
    <img src="3.png" style="display: block;margin: 0 auto;" />
</div>

### 如何解决ios input框唤启软键盘不灵敏问题？

```js
    /**
     * @param {EventTarget|Element} targetElement
     */
    FastClick.prototype.focus = function(targetElement) {
    var length;
    // Issue #160: on iOS 7, some input elements (e.g. date datetime month) throw a vague TypeError on setSelectionRange. These elements don't have an integer value for the selectionStart and selectionEnd properties, but unfortunately that can't be used for detection because accessing the properties also throws a TypeError. Just check the type instead. Filed as Apple bug #15122724.
        if (deviceIsIOS && targetElement.setSelectionRange && targetElement.type.indexOf('date') !== 0 && targetElement.type !== 'time' && targetElement.type !== 'month' && targetElement.type !== 'email') {
            length = targetElement.value.length;
            targetElement.focus();// 加入这一句话
            targetElement.setSelectionRange(length, length);
        } else {
            targetElement.focus();
        }
    };
```

你可以直接去node_module里找到fastClick文件修改focus方法，但是不建议这样做。如果以后npm install，就会被覆盖。

<div>
    <img src="4.png" style="display: block;margin: 0 auto;" />
</div>

建议你在引用fastclick的地方，重写focus方法。如vue项目，你可以在main.js文件里面，引入fastclick模块后，重写focus方法。
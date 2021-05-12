---
title: new Date()的使用和注意
author: DeepHug
index_img: /img/new-date-img.png
category: 技术
tags:
  - H5
excerpt: 移动端使用new Date()的使用和注意
date: 2021-05-12 16:26:46
---


# new Date()的使用和注意

## 写在前面

> 在实际开发中，我们可能经常会对日期进行操作，这个时候我们就用到 `new Date()`。由于ios系统对 `new Date('2020-01-01')` 这种写法不支持，会导致直接报错，所以尽量使用 `new Date('2020/01/01')` 或者 `new Date(2020,01,01)` 这两种写法。

**为了避免书写重复的代码和避免出错，项目中在日期对象的原型上面添加了`format()`,以后可以使用`format()`来满足对日期对象的操作。**

## 日期对象format的使用

> 常见使用 ``` new Date().format('yyyy-MM-dd hh:mm:ss') ```

| 格式 | 含义 | 备注 | 举例 |
| -- | -- | -- | -- |
| yyyy | 年 |  | 2021 |
| M | 月 | 不补0 | 1 |
| MM | 月 | 补0 | 01 |
| d | 日 | 不补0 | 1 |
| dd | 日 | 补0 | 01 |
| h | 小时 | 24小时制；不补0 | 1 |
| hh | 小时 | 24小时制；补0 | 01 |
| m | 分钟 | 不补0 | 1 |
| mm | 分钟 | 补0 | 01 |
| s | 秒 | 不补0 | 1 |
| ss | 秒 | 补0 | 01 |

```js
    // 具体使用如下：
    let date = new Date('2021,1,2 03:04:05');
    date.format('yyyy')  // 2021
    date.format('M')  // 1
    date.format('MM')  // 01
    date.format('d')  // 2
    date.format('dd')  // 02
    date.format('h')  // 3
    date.format('hh')  // 03
    date.format('m')  // 4
    date.format('mm')  // 04
    date.format('s')  // 5
    date.format('ss')  // 05

    date.format('yyyy/MM/dd hh:mm:ss')  // 2021/01/02 03:04:05
```
    

## 使用场景

### 两个时间的对比

**有的时候会遇到两个时间去做比对，对比两个时间的先后，此时我们应该去比较两个时间的时间戳大小**

```js
    // 获取两个时间的时间戳，然后再进行比较
    let start = new Date(2020,01,01).getTime();
    let end  = new Date(2020,01,02).getTime();
    if (end > start) {
        console.log('end的时间在start的后面')
    } else {
        console.log('start的时间在end的后面')
    }
```

### 获取前几天或者未来几天的日期（多用于日历）

**如果要获取前几天或者后几天的日期，只需要在`new Date(year, month, day)`的时候，对`day`进行加减即可。**

```js
    // 知道某一天的日期，获取前一天的日期,
    let toDay = new Date(2020,1,1).format('yyyy-MM-dd');  // "2020-02-01"
    let yesterday = new Date(2020,1,1-1).format('yyyy-MM-dd');  // "2020-01-31"
    // 获取前两天的日期
    new Date(2020,1,1-2).format('yyyy-MM-dd');  // "2020-01-30"
    // 获取前7天的日期
    new Date(2020,1,1-7).format('yyyy-MM-dd');  // "2020-01-25"
    // 获取后一天的日期
    new Date(2020,1,1+1).format('yyyy-MM-dd');  // "2020-02-02"
    // 获取后两天的日期
    new Date(2020,1,1+2).format('yyyy-MM-dd');  // "2020-02-03"
    // 获取后7天的日期
    new Date(2020,1,1+7).format('yyyy-MM-dd');  // "2020-02-08"
```

**如果要获取前几个月或者后几个月的日期，只需要在`new Date(year, month, day)`的时候，对`month`进行加减即可。**

```js
    // 知道某一天的日期，获取前一个月的日期
    let toDay = new Date(2020,1,1).format('yyyy-MM-dd');  // "2020-02-01"
    new Date(2020,1-1,1).format('yyyy-MM-dd');  // "2020-01-01"
    // 获取后一个月的日期
    new Date(2020,1+1,1).format('yyyy-MM-dd');  // "2020-03-01"
```

**如果要获取前几年或者后几年的日期，只需要在`new Date(year, month, day)`的时候，对`year`进行加减即可。**

```js
    // 知道某一天的日期，获取前一年的日期
    let toDay = new Date(2020,1,1).format('yyyy-MM-dd');  // "2020-02-01"
    new Date(2020-1,1,1).format('yyyy-MM-dd');  // "2019-02-01"
    // 获取后一年的日期
    new Date(2020+1,1,1).format('yyyy-MM-dd');  // "2021-02-01"
```

### 获取某个月有多少天

```js
    // 有的时候需要我们去计算某一个月的天数，比如我们要2月份总共有多少天， 而2月份是一个特殊的月份，因为分平年和闰年，所以计算起来比较复杂和麻烦，现在我们可以通过`new Date().getDate()`就可以轻松的获取某个月具体有多少天
    new Date(2020,2,0).getDate();  // 29
```

### 获取某天是星期几

```js
    let arrs = ['日', '一', '二', '三', '四', '五', '六'];
    let toDay = new Date(2021,3,7).getDay();
    console.log(arrs[toDay]);  // 三
```

## 写在时间对象原型(prototype)上的format方法

```js
    /**
     * 日期格式化拓展函数  new Date().format("yyyy-MM-dd") 返回2019-04-03
     * new Date().format("yyyy-MM-dd hh:mm:ss") 返回2019-04-03 09:20:10
     */
    Date.prototype.format = function (formatStr) {
        var o = {
            'M+': this.getMonth() + 1, // 月
            'd+': this.getDate(), // 日
            'h+': this.getHours(), // 时
            'm+': this.getMinutes(), // 分
            's+': this.getSeconds(), // 秒
            'q+': Math.floor((this.getMonth() + 3) / 3), // 季节
            'S': this.getMilliseconds() //毫秒数
        };
        if (/(y+)/.test(formatStr)) {
            formatStr = formatStr.replace(RegExp.$1, (this.getFullYear() + '').substr(4 - RegExp.$1.length));
        }
        for (var k in o) {
            if (new RegExp('(' + k + ')').test(formatStr)) {
                formatStr = formatStr.replace(RegExp.$1, RegExp.$1.length == 1 ? o[k] : ('00' + o[k]).substr(('' + o[k]).length));
            }
        }
        return formatStr;
    };
```
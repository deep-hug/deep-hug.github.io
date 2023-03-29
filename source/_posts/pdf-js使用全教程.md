---
title: pdf.js使用全教程
author: DeepHug
index_img: /img/wallhaven-e7zogk.jpg
category: 技术
tags:
  - Fluid
excerpt: 简介。。。
date: 2023-03-28 16:59:35
---

项目有需求要预览pdf，一开始是使用了iframe标签进行预览，后面发现效果并不理想，加载速度慢，自定义按钮难，最重要的是IE兼容太差!

PDFjs似乎是一个完美的解决方案，号称兼容各种浏览器，快速且高效，界面按钮可以配置，而且也比原生iframe框架好看一些，但是一轮的使用下来，确确实实躺了不少的坑，特此记录一下全过程，希望以后使用的时候注意一点，

开头先罗列一下一轮的使用下来遇到的问题:

1，跨域报错，

2，不支持预览base64格式pdf文件流

3，源码中使用了let const 导致老项目不兼容

4，兼容base64后，谷歌正常预览pdf文件，ie报错，

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

先去 PDF.js[官方文档](https://mozilla.github.io/pdf.js/)

<div>
    <img src="1.png" />
</div>

<div>
    <img src="2.png" />
</div>


我们只需要构建后的这些文件(其他的直接删除)，

**踩坑1**:这里就有一些坑了，引入项目使用报错，为了找这个问题，我还花了半天的时间...发现pdf.js里面用的声明变量的是es6 let const 由于负责的项目是比较老的项目了 不支持es6语法，于是**全局修改了所有的let const为var**，新项目的话，这一步可以忽略

<div>
    <img src="3.png" />
</div>

把这个文件夹直接当成一个组件 放到自己的项目中，嫌麻烦直接，网盘下载构建完成的dpf.js

链接：https://pan.baidu.com/s/13i-CQxBrONytaZJaQk6v3w 
提取码：o35m 
使用方法很简单，PDF.js接收一个file参数 在iframe标签 src属性 /viewer.html?file="url"进行使用     url可以是**pdf文件**，可以是**pdf的文件地址(流)**，

文件地址，直接使用没什么好说的，直接上代码，这里也有一个需要注意的点，就是url如果包含中文，需要转码，我们尽量进行转码，pdf会自动解码，这样靠谱一点

```js

var downloadUrl = baseUrl + 'xxx/接口地址?token=' + this.token + '&filePath='+paymentFiles[i].FILE_PATH+'&fileName='+paymentFiles[i].FILE_NAME;
//由于地址需要传文件名字和路径 文件名字包含中文，所以需要转码
var url = encodeURIComponent(downloadUrl); //获取pdf预览地址
//把文件流地址天津到file参数中
url = "/libs/DPFjs/web/viewer.html?file=" + url;
//把dom添加到页面中，打开页面即可显示
accountingArchivesQuery_ViewHTML+= '<iframe id="imgStyle_pdf" style="width:100%;height:100%;min-height:490px" src="'+url+'" scrolling="no" frameborder="0"></iframe>';

```

把dom添加到页面中，打开页面就可以看到预览了

<div>
    <img src="4.png" />
</div>

如果不进行转码就会出现以下错误

<div>
    <img src="5.png" />
</div>

原因是中文会导致url解析时乱码

<div>
    <img src="9.png" />
</div>

<div>
    <img src="10.png" />
</div>


**踩坑2**:把文件地址赋值给file，打开浏览器跨域报错.这个的原因是因为viewer.js中有跨域判断的代码注释掉就好了

<div>
    <img src="6.png" />
</div>

**但是有一些场景后台给我返回的数据是base64格式，如果是base64数据的pdf又如何处理呢?直接使用是不ok的，pdf不支持base64，继续折腾**

**踩坑3**PDF.js默认不支持pdf base64格式数据的预览.解决方法修改源码 让pdf.js支持base64位数据，但是这里有一个大坑(**IE不兼容...**)，不知道各位使用之后有没有同样问题，

[RFC2045]中有规定：Base64一行不能超过76字符，超过则添加回车换行符。因此需要把base64字段中的换行符，回车符给去掉。且需要转换成pdf.js能直接解析的Uint8Array类型

在viewer.html中引入viewer.js之前添加以下代码

```js

<script type="text/javascript">
  var DEFAULT_URL = "";
  var pdfUrl = document.location.search.substring(1);
  if(null == pdfUrl || "" == pdfUrl){
      var BASE64_MARKER = ';base64，';//声明文件流编码格式
      var preFileId = "";
      var pdfAsDataUri = sessionStorage.getItem("_imgUrl");//这里就是pdf文件的base64码，我是通过session传递base64的
      var pdfAsArray = convertDataURIToBinary(pdfAsDataUri);
      DEFAULT_URL = pdfAsArray;
      //编码转换
      function convertDataURIToBinary(dataURI) {
          //[RFC2045]中有规定：Base64一行不能超过76字符，超过则添加回车换行符。因此需要把base64字段中的换行符，回车符给去掉。
          var base64Index = dataURI.indexOf(BASE64_MARKER) + BASE64_MARKER.length;
          var newUrl = dataURI.substring(base64Index).replace(/[\n\r]/g，'');
          var raw = window.atob(newUrl);//这个方法在ie内核下无法正常解析。
          var rawLength = raw.length;
          //转换成pdf.js能直接解析的Uint8Array类型
          var array = new Uint8Array(new ArrayBuffer(rawLength));
          for (i = 0; i < rawLength; i++) {
              array[i] = raw.charCodeAt(i) & 0xff;
          }
          return array;
      }
  }
</script>

```

修改viewer.js中 defaultUrl 的 value 为 DEFAULT_URL

<div>
    <img src="7.png" />
</div>

在页面中使用 把请求回来的base64数据存储到本地 sessionStorage 打开/viewer.html 自动获取本地数据进行预览

```js

if(fileList[i].source == 'pdf'){
  //使用pdf.js 预览base64格式         
  sessionStorage.setItem('_imgUrl'，"data:application/pdf;base64，"+fileList[i].fileBase64);
  imageHTML = '<iframe src="/libs/DPFjs/web/viewer.html" class="imgStyle_pdf" 
  frameborder="0" scrolling="no" name="发票图片" ></iframe>';
  preview_div.append(imageHTML)
}

```

谷歌正常预览，ie报错!


<div>
    <img src="4.png" />
</div>

<div>
    <img src="5.png" />
</div>

打印看一下，分析原因可能是atob(newUrl);//这个方法在ie内核下可能无法正常解析。

<div>
    <img src="8.png" />
</div>

贯彻一把唆的原则.**直接把base64流，还原成pdf**，**谷歌预览正常ie预览正常!!!**

解决方案:把pdf base64流转pdf 前提是原本就是pdf 且pdf.js只能预览pdf 以下代码添加到后台接口返回回调

#### base64格式的pdf，最终解决方案

```js

var base = res.pdfValue;//要传入的base64数据
var bstr = atob(base);
var n = bstr.length;
var u8arr = new Uint8Array(n);
while (n--) {u8arr[n] = bstr.charCodeAt(n);}
//确定解析格式，转换成pdf 可能可以变成img，但没有深入研究
var blob = new Blob([u8arr]， {type: "application/pdf;chartset=UTF-8"});
var url = window.URL.createObjectURL(blob);
//使用pdf.js 预览
console.log(url， "url");
var add = "../../assets/static/pdfjs/web/viewer.html?file=" + url;
let imageHTML = "<iframe id=\"imgStyle_pdf\" class=\"imgStyle_pdf\" style=\"width:100%;height:100%;min-height:100vh;\" src=\""+add+"\" scrolling=\"no\" frameborder=\"0\"></iframe>";
document.querySelector("#preview-div").innerHTML = imageHTML;

````

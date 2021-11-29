---
title: 移动端使用vant UI的上传组件上传图片时，回调函数返回的文件file对象有bug
author: DeepHug
index_img: /img/deep-hug-blog-wallpaper.JPG
category: 技术
tags:
  - H5 Vue Vant
excerpt: 在进行移动端开始的时候，经常会使用到UI库，我在移动端开发时用到的UI库是Vant UI(有赞UI)，在使用过程中发现了其上传组件在上传图片时的回调函数(before-read、after-read、before-delete)返回的文件(file)对象有bug。
date: 2021-11-29 18:13:56
---

有一次修改同事的bug，他开发上传图片的一个功能。开发完成的时候功能是ok，让同事去测试的时候，发现了其问题。问题就是上传图片这个功能并不能使用。当我本地启动项目用chrome浏览器打开，测试其功能的时候，发现一切也都是正常的，并没有测试所说的bug。然后我用手机去进行操作，发现确实上传不了，确实是存在bug的。

然后和另外一个同事一起去看这个bug，起初我对上传图片这个功能是没有接触过的，也不太懂里面的原理什么的。同事说之前的项目里面有用到上传图片的功能，让我找到并cv过来。cv完成之后，经过测试发现确实好了，直到现在也不是很清楚其问题出在哪里，让人很是费解。

<div>
    <img src="1.png" />
</div>

```js
  /**
 * compressImage 图片压缩
 * @param img    图片文件
 * @param width  要压缩的宽度
 * @param height 要压缩的高度
 * @param qualityArgument 要压缩的分辨率
 * @returns {*}
 */
export function compressImage(img, width, height, qualityArgument) {
    return new Promise(resolve => {
        // 缩放图片需要的canvas
        let canvas = document.createElement('canvas');
        let context = canvas.getContext('2d');

        // 图片原始尺寸
        let originWidth = img.width;
        let originHeight = img.height;

        // 最大尺寸限制
        let maxWidth = width,
            maxHeight = height;
        // 目标尺寸
        let targetWidth = originWidth,
            targetHeight = originHeight;
        // 图片尺寸超过的限制
        if (originWidth > maxWidth || originHeight > maxHeight) {
            if (originWidth / originHeight > maxWidth / maxHeight) {
                targetWidth = maxWidth;
                targetHeight = Math.round(
                    maxWidth * (originHeight / originWidth)
                );
            } else {
                targetHeight = maxHeight;
                targetWidth = Math.round(
                    maxHeight * (originWidth / originHeight)
                );
            }
        }
        // canvas对图片进行缩放
        canvas.width = targetWidth;
        canvas.height = targetHeight;
        // 清除画布
        context.clearRect(0, 0, targetWidth, targetHeight);
        // 图片压缩 第一个参数是创建的img对象；第二三个参数是左上角坐标，后面两个是画布区域宽高
        context.drawImage(img, 0, 0, targetWidth, targetHeight);

        //压缩后的图片转base64 url
        //qualityArgument表示导出的图片质量，只有导出为jpeg和webp格式的时候此参数才有效，默认值是0.92*
        qualityArgument = qualityArgument ? qualityArgument : 0.92;
        let newUrl = canvas.toDataURL('image/jpeg', qualityArgument); //base64 格式
        //也可以把压缩后的图片转blob格式用于上传
        if (canvas.toBlob) {
            canvas.toBlob(
                blob => {
                    resolve({
                        url: newUrl,
                        blob: blob
                    });
                },
                'image/jpeg',
                qualityArgument
            );
        } else {
            let blob = dataURItoBlob(newUrl);
            resolve({
                url: newUrl,
                blob: blob
            });
        }
    });
}

/**
 * 将base64转换为文件
 * @param {图片地址} dataurl
 * @param {图片名} filename
 */
export function dataURLtoFile(dataurl, filename) {
    var arr = dataurl.split(','),
        mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]),
        n = bstr.length,
        u8arr = new Uint8Array(n);
    while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
    }
    return new File([u8arr], filename, { type: mime });
}
```

上面的两个方法是图片中用到的方法，第一个方法是将图片文件转化为`base64 格式`，第二个方法是将`base64`转化为文件对象。

<div>
    <img src="2.png" />
</div>

**data.file 是vant的回调函数的文件对象，imgFile 是我自己生成的文件对象。**

我将vant回调函数返回的文件对象和我自己通过方法生成的文件对象都在控制台进行了打印，也没有找到哪里有区别。**但是vant返回的文件对象，使用的时候确实有问题，chrome表现正常，而手机端（ios和Android）都不能正常使用，可能是存在兼容问题吧**
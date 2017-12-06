---
title: 解决移动端网页上传图片自动旋转的问题
date: 2017-05-27 20:49:29
tags: 
 - js 
 - 实习
categories: 前端笔记
---
## 问题描述

之前在开发微信公众号的时候，遇到有一个需求是需要允许用户在移动端的网页上传图片。然而在初步开发后遇到一个bug：大多数情况下，用户上传的照片显示出来都会被旋转90度或者180度。
比如手机中选择的图片是这样子：
![](http://i4.buimg.com/588926/3426772102bd14c4.png)
而用户上传以后显示出来的照片是这些样子：
![](http://i1.piimg.com/588926/33c84cd831b36546.png) 
![](http://i1.piimg.com/588926/6f74aeb543a6b26e.png) 
![](http://i1.piimg.com/588926/4036ad33d2d8afb9.png)
<!-- more --> 
当时由于时间紧急，产品带着这个bug就直接上线给内部人员使用了，结果后台看到的照片四个角度的都有，看的我头都歪了。

## Exif.js简介

后来我在网上找到了这个Exif.js库：[Exif.js 读取图像的元数据](http://code.ciaoca.com/javascript/exif-js/)

> Exif.js 提供了 JavaScript 读取图像的原始数据的功能扩展，例如：拍照方向、相机设备型号、拍摄时间、ISO 感光度、GPS 地理位置等数据。

实际上用手机拍摄的照片文件中都会存有许多拍摄信息比如设备型号、拍摄时间、拍摄地点等等。通过Exif.js库我们可以从照片中方便地获取这些信息。

## 解决思路

所以接下来的思路就是在用户上传图片的时候对使用Exif.js读取图片的拍摄方向，根据用户的拍摄方向对图像进行相应的旋转，使之正常显示。

通过Exif.js我们可以得到图像的拍摄方向*Orientation*，它是一个整数，这个参数的值和对应的意义以及调整的方式如下表所示:

| Orientation        | 原图   |  调整方式  |
| --------   | -----:  | :----:  |
| 1     |![](http://i4.buimg.com/588926/3426772102bd14c4.png) |   不变     |
| 2     |![](http://i1.piimg.com/588926/2e1c78a3f483810b.png)|   左右翻转   |
| 3     |![](http://i1.piimg.com/588926/6f74aeb543a6b26e.png)    |  顺（逆）时针旋转180°  |
| 4     |![](http://i1.piimg.com/588926/88750989de6b9eb1.png)|   上下翻转     |
| 5     |![](http://i1.piimg.com/588926/933e40faf86a6b07.png)|   顺时针旋转90°，左右翻转   |
| 6     |![](http://i1.piimg.com/588926/4036ad33d2d8afb9.png)   |  顺时针旋转90°  |
| 7     |![](http://i1.piimg.com/588926/3b9cb58104cb0414.png)|   逆时针旋转90°，左右翻转    |
| 8     |![](http://i1.piimg.com/588926/33c84cd831b36546.png)   |   逆时针旋转90°   |

值得注意的是，在这个bug中我们只会遇到Orientation值为1，3，6，8的情况，因为我们只存在拍摄角度的问题，不存在翻转问题。

## 代码

最后附上代码，代码中使用canvas对照片进行旋转处理，这里就不赘述了。
``` javascript
$uploaderInput.on("change", function(e){
    var src, url = window.URL || window.webkitURL || window.mozURL, files = e.target.files;
    for (var i = 0, len = files.length; i < len; ++i) {
        var file = files[i];
        // Ensure it's an image
        if(file.type.match(/image.*/)) {
            console.log('An image has been loaded');
            // Load the image
            var reader = new FileReader();
            reader.onload = function (readerEvent) {
                var image = new Image();
                image.onload = function (imageEvent) {
                    EXIF.getData(image, function() {
                        EXIF.getAllTags(this);
                        Orientation = EXIF.getTag(this, 'Orientation');
                    });
                    var cxt = canvas.getContext('2d');
                    if(Orientation == 3) {
                        canvas.width = width;
                        canvas.height = height;
                        cxt.rotate(Math.PI);
                        cxt.drawImage(image, 0, 0, -width, -height);
                    }
                    else if(Orientation == 8) {
                        canvas.width = height;
                        canvas.height = width;
                        cxt.rotate(Math.PI * 3 / 2);
                        cxt.drawImage(image, 0, 0, -width, height);
                    }
                    else if(Orientation == 6) {
                        canvas.width = height;
                        canvas.height = width;
                        cxt.rotate(Math.PI / 2);
                        cxt.drawImage(image, 0, 0, width, -height);
                    }
                    else {
                        canvas.width = width;
                        canvas.height = height;
                        cxt.drawImage(image, 0, 0, width, height);
                    }
                    var dataUrl = canvas.toDataURL('image/jpeg');
                    var resizedImage = dataURLToBlob(dataUrl);
                    $.event.trigger({
                        type: "imageResized",
                        blob: resizedImage,
                        url: dataUrl
                    });
                };
                image.src = readerEvent.target.result;
            };
            reader.readAsDataURL(file);
        }
    }
});
```
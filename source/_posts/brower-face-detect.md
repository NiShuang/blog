---
title: 基于face-api.js的人脸实时跟踪
comments: true
date: 2020-03-07 19:54:41
tags: 
	 - js 
	 - HTML
category: 前端笔记 
---

疫情期间，公司选择让员工进行远程办公，却又难以监督员工保证他们的工时。所以老板想出了通过摄像头配合人脸识别算法，计算一天内员工在电脑面前的时间占比。

[项目的Github地址](https://github.com/NiShuang/browser_face_detect)

## 通过浏览器开启摄像头

这部分代码是在网上找的，需要兼容多种浏览器：
<!-- more --> 

```
if (navigator.mediaDevices === undefined) {
	navigator.mediaDevices = {};
}
if (navigator.mediaDevices.getUserMedia === undefined) {
	avigator.mediaDevices.getUserMedia = function (constraints) {	// 首先，如果有getUserMedia的话，就获得它
		var getUserMedia = navigator.webkitGetUserMedia || navigator.mozGetUserMedia || navigator.msGetUserMedia;
	 
	    // 一些浏览器根本没实现它 - 那么就返回一个error到promise的reject来保持一个统一的接口
	    if (!getUserMedia) {
	        return Promise.reject(new Error('getUserMedia is not implemented in this browser'));
	    }
	 
	    // 否则，为老的navigator.getUserMedia方法包裹一个Promise
	    return new Promise(function (resolve, reject) {
	         getUserMedia.call(navigator, constraints, resolve, reject);
	    });
	}
}
const constraints = {
    video: true,
    audio: false
};
let promise = navigator.mediaDevices.getUserMedia(constraints);
promise.then(stream => {
    let v = document.getElementById('v');
    // 旧的浏览器可能没有srcObject
    if ("srcObject" in v) {
         v.srcObject = stream;
    } else {
     	// 防止再新的浏览器里使用它，应为它已经不再支持了
         v.src = window.URL.createObjectURL(stream);
    }
    v.onloadedmetadata = function (e) {
        v.play();
    };}).catch(err => {
        console.error(err.name + ": " + err.message);
})
```

## 人脸识别

人脸识别使用的是Github上的一个人脸识别库 [face-api.js](https://github.com/justadudewhohacks/face-api.js) 。face-api.js可以识别出视频流中人脸的轮廓和表情，调整识别精度等，Github上有详细的使用教程。

```
// 初始化
faceapi.nets.ssdMobilenetv1.loadFromUri(dir),
// faceapi.nets.tinyFaceDetector.loadFromUri(dir),
faceapi.nets.faceLandmark68Net.loadFromUri(dir),
// faceapi.nets.faceRecognitionNet.loadFromUri(dir),
// faceapi.nets.faceExpressionNet.loadFromUri(dir)
            
var video = document.getElementById('video');
let canvas = faceapi.createCanvasFromMedia(video);
document.body.append(canvas);
faceapi.matchDimensions(canvas, displaySize);

// const options = new faceapi.TinyFaceDetectorOptions({ scoreThreshold: 0.2, inputSize: 608 });
const options = new faceapi.SsdMobilenetv1Options({ minConfidence: 0.5, maxResults: 3 });
let detections = await faceapi.detectAllFaces(video, options).withFaceLandmarks();

// 在画面中显示人脸轮廓描边
const resizedDetections = faceapi.resizeResults(detections, displaySize);
canvas.getContext('2d').clearRect(0, 0, canvas.width, canvas.height);
faceapi.draw.drawDetections(canvas, resizedDetections);
faceapi.draw.drawFaceLandmarks(canvas, resizedDetections);

```

## 实现实时跟踪
实现实时跟踪的思路就是，通过定时器，每隔1秒钟对当前的图像进行人脸识别并描边，这样间接实现了对视频的实时人脸跟踪，如果想要跟踪速度更加灵敏一点，可以把时间间隔改成0.1秒。


```        
video.addEventListener('play', () => {
    console.log('play lisetner')
    canvas = faceapi.createCanvasFromMedia(video);
    document.body.append(canvas);
    faceapi.matchDimensions(canvas, displaySize);
    takePhoto();
    setInterval(takePhoto,1000);
});

async function takePhoto(){
    if (!faceapiReady) {
        return;
    }
    let detections = await detect();
    draw(detections);
}

async function detect() {
    // const options = new faceapi.TinyFaceDetectorOptions({ scoreThreshold: 0.2, inputSize: 608 });
    const options = new faceapi.SsdMobilenetv1Options({ minConfidence: 0.5, maxResults: 3 });
    const detections = await faceapi.detectAllFaces(video, options).withFaceLandmarks();
    return detections;
}

function draw(detections) {
    const resizedDetections = faceapi.resizeResults(detections, displaySize);
    canvas.getContext('2d').clearRect(0, 0, canvas.width, canvas.height);
    faceapi.draw.drawDetections(canvas, resizedDetections);
    faceapi.draw.drawFaceLandmarks(canvas, resizedDetections);
}


```

最后，程序会每分钟发送一次识别结果到服务器，服务器最终会计算每个人在一天内第一次的识别时间和最后一次识别时间作为上下班打卡时间，然后计算一天内识别到人脸的比例，可以作为在岗率的参考。


## 参考资料


 - [justadudewhohacks/face-api.js](https://github.com/justadudewhohacks/face-api.js)
 
-------

 > 文章标题：[基于face-api.js的人脸实时跟踪](http://www.cielni.com/2020/03/07/brower-face-detect/)
> 文章作者：[Ciel Ni](http://www.cielni.com/about/)
> 文章链接：http://www.cielni.com/2020/03/07/brower-face-detect/
> 有问题或建议欢迎与我联系讨论，转载或引用希望标明出处，感激不尽！
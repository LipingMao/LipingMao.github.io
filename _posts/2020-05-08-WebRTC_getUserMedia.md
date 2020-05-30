---
title: WebRTC -- getUserMedia
---

> 记录和学习一些基本的音视频知识。



getUserMedia可以在浏览器中用来访问音视频设备基本格式如下：

```
navigator.mediaDevices.getUserMedia(constraints);
```



它的参数是MediaStreamConstraints:

```
dictionary MediaStreamConstraints { 
    (boolean or MediaTrackConstraints) video = false, 
    (boolean or MediaTrackConstraints) audio = false
};
```



这里可以制定是否采集video或者audio，例如都采集：

```
const mediaStreamContrains = { video: true, audio: true};
```



此处可以通过MediaTrackConstraints对video/audio采集参数进行细粒度控制,例如：

```
const mediaStreamContrains = {
    video: {
        frameRate: {min: 20},
        width: {min: 640, ideal: 1280},
        height: {min: 360, ideal: 720},
      aspectRatio: 16/9
    },
    audio: {
        echoCancellation: true,
        noiseSuppression: true,
        autoGainControl: true
    }
};
```





主要可以控制的参数如下：

![image-20200530204624459](https://raw.githubusercontent.com/LipingMao/LipingMao.github.io/master/_posts/picture/image-20200530204624459.png)


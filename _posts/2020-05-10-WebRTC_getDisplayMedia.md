---
title: WebRTC -- getDisplayMedia
---

> 记录和学习一些基本的音视频知识。

getDisplayMedia可以用来抓去桌面数据用于桌面共享：

```
...

//得到桌面数据流
function getDeskStream(stream){
        localStream = stream;
}

//抓取桌面
function shareDesktop(){

        //只有在 PC 下才能抓取桌面
        if(IsPC()){
                //开始捕获桌面数据
                navigator.mediaDevices.getDisplayMedia({video: true})
                        .then(getDeskStream)
                        .catch(handleError);

                return true;
        } 
         
        return false;
         
}  

...
```

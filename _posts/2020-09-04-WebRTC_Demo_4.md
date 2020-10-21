---
title: WebRTC -- Sample 4
---

Constraint可以控制很多媒体数据的元数据, 比如分辨率:

```
dictionary MediaTrackConstraintSet {
  ConstrainULong width;
  ConstrainULong height;
  ConstrainDouble aspectRatio;
  ConstrainDouble frameRate;
  ConstrainDOMString facingMode;
  ConstrainDOMString resizeMode;
  ConstrainULong sampleRate;
  ConstrainULong sampleSize;
  ConstrainBoolean echoCancellation;
  ConstrainBoolean autoGainControl;
  ConstrainBoolean noiseSuppression;
  ConstrainDouble latency;
  ConstrainULong channelCount;
  ConstrainDOMString deviceId;
  ConstrainDOMString groupId;
};
```

see: https://webrtc.github.io/samples/src/content/getusermedia/resolution/

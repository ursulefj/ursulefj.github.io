---
title: webAR —— AR.js demo
date: 2021-03-06 16:13:13
tags: AR
---

## 前言

最近有个新的调研需求，需要做到使用手机摄像扫在产品上的包装，实现一个展开的动画模型效果。初步的讨论后还是使用成本更低的webAR，在简单的了解后选择了使用更广泛更轻量的AR.js。

## AR.js

AR.js是一个用于Web上增强现实的轻量级库，具有基于标记和基于位置的AR等功能。
AR类型有三种

- **图像跟踪**，当摄像机找到2D图像时，可以在其顶部或附近显示某种内容。内容也可以是2D图像，GIF，3D模型（也为动画）和2D视频。
- **标记跟踪**，当摄像机找到标记时，可以显示一些内容（与图像跟踪相同）。标记非常稳定，但形状，颜色和大小受到限制。建议针对需要大量不同内容的标记的体验。
    标记有三种类型 
    - 宏（Hiro）
    - 条码
    - 图案
- **基于位置的AR**，这种AR使用真实世界的位置来在用户设备上显示增强现实内容。
  
AR.js使用jsartoolkit5进行跟踪，渲染库使用的是three.js或A-Frame。

## demo

demo使用的是图像跟踪的形式，将需要识别的图片通过`NFT Marker Creator`工具获取需要的识别文件。工具有两个版本，[web版](https://carnaux.github.io/NFT-Marker-Creator/#/)和[node.js](https://github.com/Carnaux/NFT-Marker-Creator)版本。需要处理的图片文件小数量少的话推荐使用web版的。特别注意的一点是图片dpi越高对后面识别的成功率越高。通过工具获取三个文件的图像描述符后放入相对应的文件夹。

这边方便起见，使用的是A-Frame，也可以使用three.js的方式

```
<script src="https://cdn.jsdelivr.net/gh/aframevr/aframe@1c2407b26c61958baa93967b5412487cd94b290b/dist/aframe-master.min.js"></script>

<style>
  .arjs-loader {
    height: 100%;
    width: 100%;
    position: absolute;
    top: 0;
    left: 0;
    background-color: rgba(0, 0, 0, 0.8);
    z-index: 9999;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .arjs-loader div {
    text-align: center;
    font-size: 1.25em;
    color: white;
  }
</style>

<!-- rawgithack development URL -->
<script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar-nft.js"></script>

<body style="margin: 0px; overflow: hidden">
  <!-- minimal loader shown until image descriptors are loaded -->
  <div class="arjs-loader">
    <div>Loading, please wait...</div>
  </div>
  <a-scene
    vr-mode-ui="enabled: false;"
    renderer="logarithmicDepthBuffer: true;"
    embedded
    arjs="trackingMethod: best; sourceType: webcam; debugUIEnabled: false;"
  >
    <!-- use rawgithack to retrieve the correct url for nft marker (see 'pinball' below) -->
    <a-nft type="nft" url="./trex/trex-image/trex" smooth="true" smoothCount="10" smoothTolerance="0.01" smoothThreshold="5">
      <a-entity
        gltf-model="https://arjs-cors-proxy.herokuapp.com/https://raw.githack.com/AR-js-org/AR.js/master/aframe/examples/image-tracking/nft/trex/scene.gltf"
        scale="5 5 5"
        position="50 150 -100"
      >
      </a-entity>
    </a-nft>
    <a-entity camera></a-entity>
  </a-scene>
</body>
```

效果如下
<!-- ![js引入效果](/images/202202/1.gif) -->
<img src="/images/202202/1.gif"  height="330" width="495">

## 总结

静态模型和动态模型呈现效果都还不错，但关键的问题是定位不太准确。镜头晃动或者换个角度，模型与定位图片的相对坐标变动太大。原因可能是图片的追踪无法像基于标记类型的那样准确。可能没办法在
---
title: iOS 页面渲染 - 流程
category:
  - iOS
  - UI
tags:
  - iOS
date: 2021-08-15 19:03:40
---

![Core Animation Pipeline](../../../../img/iOS/UI/iOS页面渲染/Core%20Animation%20Pipeline.jpeg)

* Handle Events：这个过程中会先处理点击事件，这个过程中有可能会需要改变页面的布局和界面层次。
* Commit Transaction：此时 app 会通过 CPU 处理显示内容的前置计算，比如布局计算、图片解码等任务，接下来会进行详细的讲解。之后将计算好的图层进行打包发给 Render Server。
* Decode：打包好的图层被传输到 Render Server 之后，首先会进行解码。注意完成解码之后需要等待下一个 RunLoop 才会执行下一步 Draw Calls。
* Draw Calls：解码完成后，Core Animation 会调用下层渲染框架（比如 OpenGL 或者 Metal）的方法进行绘制，进而调用到 GPU。
* Render：这一阶段主要由 GPU 进行渲染。
* Display：显示阶段，需要等 render 结束的下一个 RunLoop 触发显示。

一般开发当中能影响到的就是 Handle Events 和 Commit Transaction 这两个阶段，这也是开发者接触最多的部分。Handle Events 就是处理触摸事件，而 Commit Transaction 这部分中主要进行的是：Layout、Display、Prepare、Commit 等四个具体的操作。

**Layout：构建视图**
这个阶段主要处理视图的构建和布局，具体步骤包括：

调用重载的 layoutSubviews 方法
创建视图，并通过 addSubview 方法添加子视图
计算视图布局，即所有的 Layout Constraint

由于这个阶段是在 CPU 中进行，通常是 CPU 限制或者 IO 限制，所以我们应该尽量高效轻量地操作，减少这部分的时间，比如减少非必要的视图创建、简化布局计算、减少视图层级等。

**Display：绘制视图**
这个阶段主要是交给 Core Graphics 进行视图的绘制，注意不是真正的显示，而是得到前文所说的图元 primitives 数据：

根据上一阶段 Layout 的结果创建得到图元信息。
如果重写了 drawRect: 方法，那么会调用重载的 drawRect: 方法，在 drawRect: 方法中手动绘制得到 bitmap 数据，从而自定义视图的绘制。

注意正常情况下 Display 阶段只会得到图元 primitives 信息，而位图 bitmap 是在 GPU 中根据图元信息绘制得到的。但是如果重写了 drawRect: 方法，这个方法会直接调用 Core Graphics 绘制方法得到 bitmap 数据，同时系统会额外申请一块内存，用于暂存绘制好的 bitmap。
由于重写了  drawRect: 方法，导致绘制过程从 GPU 转移到了 CPU，这就导致了一定的效率损失。与此同时，这个过程会额外使用 CPU 和内存，因此需要高效绘制，否则容易造成 CPU 卡顿或者内存爆炸。

**Prepare：Core Animation 额外的工作**
这一步主要是：图片解码和转换

**Commit：打包并发送**
这一步主要是：图层打包并发送到 Render Server。
注意 commit 操作是依赖图层树递归执行的，所以如果图层树过于复杂，commit 的开销就会很大。这也是我们希望减少视图层级，从而降低图层树复杂度的原因。

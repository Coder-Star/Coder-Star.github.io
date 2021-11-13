---
title: iOS 优化 -UITableView
category:
  - iOS
  - 优化
tags:
  - iOS
  - 优化
date: 2021-08-03 20:59:49
---

## 前言


UI方面的通用优化也会是具体控件内部优化。

## 优化

### 预加载

```swift
protocol UITableViewDataSourcePrefetching {
    func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [NSIndexPath])
    optional func tableView(_ tableView: UITableView, cancelPrefetchingForRowsAt indexPaths:
                            [NSIndexPath])
}
class UITableView : UIScrollView {
    weak var prefetchDataSource: UITableViewDataSourcePrefetching?
}
```

mjrefresh也会有一个属性，用来设置其底部滚动时的偏移量。

### cell 复用

cell 复用时需要注意在 cell 上添加子视图导致重叠的问题；

为了使 cell 可以尽可能进行复用，需要尽量减少 tableview 的 cell 种类，可以将几种 cell 集中在一个 cell 上，然后通过控制子视图的显示与隐藏起到展示不同 cell 的效果；同理，尽量少用 addView 给 Cell 动态添加 View，可以初始化时就添加，然后通过 hide 来控制是否显示；

### 高度缓存

- 对于固定高度的 cell，我们可以直接给定 cell 高度；
- 对于高度不固定的 cell，我们需要将计算后的高度缓存下来，下次展示这个 cell 时，直接使用缓存的高度，减少高度计算

### 设计统一规格的 Cell，不要动态的 add 或者 remove 子控件，最好在初始化时就添加完，然后通过 hidden 来控制是否显示；

### 异步绘制

- AsyncDisplayKit

### 滑动时，按需加载

根据 runloop 的 mode，使图片下载等过程在 default 模式下再加载，滚动过程中不加载

### 提前布局

其实在从服务端拿到 JSON 数据的时候，关于视图的布局就已经确定了，包括每个控件的 frame、cell 的高度以及文本排版结果等等，在这个时候完全可以在后台线程计算并封装为对应的布局对象 XXXTableViewCellLayout，每个 cellLayout 的内存占用并不是很多，所以直接全部缓存到内存中。当列表滚动到某个 cell 的时候，直接拿到对应的 cellLayout 配置这个 cell 的对应属性即可。当然，该有的计算是免不了的，只是提前算好并缓存，免去了在滚动的时候计算和重复的计算。
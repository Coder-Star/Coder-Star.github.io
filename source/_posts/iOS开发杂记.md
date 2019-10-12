---
title: iOS开发杂记
date: 2019-10-12 14:12:08
categories: [iOS]
tags: [iOS]
---
# 1、UITapGestureRecognizer相关
* **一个手势添加到多个view上，只有最后一个view添加的有效**
>每一个Gesture Recognizer关联一个View，但是一个View可以关联多个Gesture Recognizer,一个View可以响应多种触控操作方式。当一个触控事件发生时，Gesture Recognizer接收一个动作消息要先于View本身，结果就是Gesture Recognizer作为View处理触控事件的代理。当Gesture Recognizer接收到指定的事件时，它就会发送一条动作消息(action message)给ViewController并处理。

```
//labelOne点击事件会失效
let singleTapGesture = UITapGestureRecognizer(target: self, action: #selector(changeAgreeState))

labelOne.addGestureRecognizer(singleTapGesture)
labelOne.isUserInteractionEnabled = true
        
labelTwo.addGestureRecognizer(singleTapGesture)
labelTwo.isUserInteractionEnabled = true
```
# 2、元素被导航栏遮挡
```
self.edgesForExtendedLayout =  UIRectEdge.init(rawValue: 0)
//设置edgesForExtendedLayout后会导致导航栏颜色变灰，需要人工改变导航栏颜色
self.navigationController?.navigationBar.backgroundColor = .white
```
# 3、页面向下偏移
当view的第一个页面是scrollView或者tableView时，页面会自动向下偏移64pt，如果已经设置了top约束，则页面就会错位，使用下面代码使scrollView或者tableView不要自动向下偏移。
```
if #available(iOS 11.0, *) {
   scrollView.contentInsetAdjustmentBehavior = UIScrollView.ContentInsetAdjustmentBehavior.never
  //tableView.contentInsetAdjustmentBehavior = UIScrollView.ContentInsetAdjustmentBehavior.never
}else{
   self.automaticallyAdjustsScrollViewInsets = false
}
```
# 4、控制页面view在安全区域内
```
baseScrollView.snp.makeConstraints{(make) in
      make.left.right.equalToSuperview()
      make.width.equalTo(ConstantsHelp.SCREENWITH)
      if #available(iOS 11.0, *) {
         make.top.equalTo(view.safeAreaLayoutGuide.snp.top)
         make.bottom.equalTo(view.safeAreaLayoutGuide.snp.bottom)
      } else {
         make.top.equalTo(topLayoutGuide.snp.bottom)
         make.bottom.equalTo(bottomLayoutGuide.snp.bottom)
      }
}
```
# 5、启动图配置方法说明
[启动图配置](https://www.jianshu.com/p/05b31f29b135)

# 6、ViewController的生命周期
```
initWithCoder：         通过 nib 文件初始化时触发。
awakeFromNib：          nib 文件被加载的时候，会发生一个 awakeFromNib 的消息到 nib 文件中的每个对象。      
loadView：              开始加载视图控制器自带的 view。
viewDidLoad：           视图控制器的 view 被加载完成。  
viewWillAppear：        视图控制器的 view 将要显示在 window 上。
updateViewConstraints： 视图控制器的 view 开始更新 AutoLayout 约束。
viewWillLayoutSubviews：视图控制器的 view 将要更新内容视图的位置。
viewDidLayoutSubviews： 视图控制器的 view 已经更新视图的位置。
viewDidAppear：         视图控制器的 view 已经展示到 window 上。 
viewWillDisappear：     视图控制器的 view 将要从 window 上消失。
viewDidDisappear：      视图控制器的 view 已经从 window 上消失。
```
# 7、启动图所有尺寸
![启动图尺寸1.png](/iOS开发杂记/启动图尺寸1.png)
![启动图尺寸2.png](/iOS开发杂记/启动图尺寸2.png)

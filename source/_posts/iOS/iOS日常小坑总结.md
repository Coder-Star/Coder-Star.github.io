---
title: iOS 日常小坑总结
category:
  - iOS
tags:
  - iOS
  - 坑
date: 2021-05-27 14:51:08
---

- iOS 请求权限回调，第一次用户选择时回调会发生在非主线程，如果在回调中进行 UI 操作，请注意切换到主线程。

- Xib 设置设置颜色使用的色域是 Generic RGB，而 API 设置颜色以及吸色计等大部分都是使用的 sRGB，所以我们需要注意将 xib 的色域换成 sRGB，

- WkWebview 字体过小，注入一段 js `var meta = document.createElement('meta'); meta.setAttribute('name', 'viewport'); meta.setAttribute('content', 'width=device-width'); document.getElementsByTagName('head')[0].appendChild(meta);`

- UIButton 通过 textLabel 设置颜色、标题不生效，因为这两个属性跟按钮本身状态挂钩，UIButton 只会显示跟状态对应的标题和颜色。

- `system`类型的 UIButton 点击高亮时表面会有一层灰色，修改为`custom`类型可解决；

- attributedText 设置后，text、lineBreakMode、textAlignment 等属性会被重置，虽然文档上说 font，textColor 也会被重置，但是根据我的测试，结果显示这两个属性不会被重置。

- UICollectionViewFlowLayout 设置`estimatedItemSize`后`sizeForItemAt`代理虽然会走，但是设置的值不会起效果

- UITextField 的 clearButtonMode 背后的删除按钮本身也是 UITextField rightView 的一种，如果手动设置了 rightView，clearButtonMode 的设置将不会生效。

- UITextField 直接设置 text，不会触发 editingChanged，如果需要实现触发目的，可以手动调用一下触发事件。

```swift
  field.text = nickName
  field.sendActions(for: .editingChanged)
```

- `selector-based` API 即 [addObserver(_:selector:name:object:)](https://developer.apple.com/documentation/foundation/notificationcenter/1415360-addobserver) 不再需要主动调用 `NotificationCenter.removeObserver(_:)` 方法（iOS 11.2 中测试）;
  `block-based` API 即 [addObserver(forName:object:queue:using:)](https://developer.apple.com/documentation/foundation/notificationcenter/1411723-addobserver) 还是需要主动调用 `NotificationCenter.removeObserver(_:)` 才能删除 callback Block（在 iOS 11.2 中测试）。

- UITableCell 初始化时 Width 不管是什么机型都会是 320，如果在该时间需要使用 Width，需要手动设置一下 Width；
- UILabel 设置渐变色后，文字看不见，建议换成 UIButton；
- UILabel 设置 frame 如果数值不为整数时，可能会有一个灰边。
- UILabel 调整基线准值，会导致不换行。
- 子 vc 也会影响父 vc 的导航栏样式

- UIButton设置imageView ishidden alpha 无反应，可以使用[UIButton's imageView property and hidden/alpha value](https://stackoverflow.com/questions/11673479/uibuttons-imageview-property-and-hidden-alpha-value)

- UIButton（可推及到UIControl）的state值不是一个enum，而是一个OptionSet。意味着状态间并不是互斥关系，而是一个叠加关系，其中遇到的实例为 UIButton 在已经是selected的前提时，长按时并不会出现高亮状态设置的内容，而是会出现normal状态下的内容。解决方式为：`setImage(UIImage(named: "XXX"), for: [.selected, .highlighted])`，即为该种状态单独设置一下值。

- UITableView 设置 group 样式 头部会有一段空白，可以设置`tableView.tableHeaderView = UIView(frame: CGRect(x: 0, y: 0, width: 0, height: 0.01))`


```swift
    /// 处理 UICollectionView 只有一个cell时，cell被自动居中问题
    /// 还有一个 _setRowAlignmentsOptions 私有API方案
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, insetForSectionAt section: Int) -> UIEdgeInsets {
        if dataArr.count == 1 {
            let right = collectionView.frame.width - self.collectionView(collectionView, layout: collectionViewLayout, sizeForItemAt: IndexPath(item: 0, section: section)).width
            return UIEdgeInsets(top: 0, left: 0, bottom: 0, right: right)
        } else {
            return UIEdgeInsets.zero
        }
    }
```
- UITableView cell中有文本框输入，文本框输入过程中需要调整cell高度，如果使用reloaddata会促使文本框丧失焦点，可以使用beginUpdate、endUpdate或者performBatchUpdates 两种方式处理刷新；
---
title: UITableView 相关
date: 2019-10-12 13:48:42
categories:
  - iOS
  - UI
tags: [iOS]
---

## 基础知识

### 样式

UITableView 有两种样式（plain，grouped），其中 plain 为普通列表样式，grouped 是分组样式。可以在实例化的时候进行设置，默认是 plain。

```swift
//tableview样式
let  tableView = UITableView()  //plain形式
let  tableview = UITableView(frame:frame, style:.grouped) //grouped形式

tableview.separatorStyle = .singleLine //设置是否有横线,默认有横线

//tablecell样式
tablecell.selectionStyle = .gray  //设置cell点击的样式，其中设置为none时候点击时候没有阴影效果
tablecell.accessoryType = .disclosureIndicator //设置cell尾部样式，箭头还是对号等等
```

### 数据源（UITableViewDataSource）

```swift
//行数（必须实现）
public func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int
//每个cell（必须实现）
public func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell
//组数，非必须实现 默认是1
optional public func numberOfSections(in tableView: UITableView) -> Int
```

### 代理（UITableViewDelegate）

UITableViewDelegate 继承 UIScrollViewDelegate，也就是说 UITableView 其实拥有很多 UIScrollView 的操作。

```swift
//cell点击监听
optional public func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath)
//取消点击后产生的阴影
optional public func tableView(_ tableView: UITableView, didDeselectRowAt indexPath: IndexPath)
//编辑操作，左滑
optional public func tableView(_ tableView: UITableView, editActionsForRowAt indexPath: IndexPath) -> [UITableViewRowAction]?
//cell即将展示
optional public func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath)
```

## 问题集锦

### 取消 TableView 前后默认空白部分

代码示例

```swift
    func tableView(_ tableView: UITableView, heightForHeaderInSection section: Int) -> CGFloat {
        return 0.01
    }
    func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView? {
        return nil
    }
    func tableView(_ tableView: UITableView, heightForFooterInSection section: Int) -> CGFloat {
        return 0.01
    }
    func tableView(_ tableView: UITableView, viewForFooterInSection section: Int) -> UIView? {
        return nil
    }
```

注意事项

1. 在设置 **tableView.delegate = self** 时注意将其放在 tableView 加入其 View 之前 ( **ParentView.addSubview(tableView)** )，否则上述代码不会生效。
2. 在 ios11 系统以下，如果上述 header 以及 footer 置为 0，则会认为其没有设置高度，还是会默认设置大约为 50 个像素的高度；ios11 系统以上可直接设置为 0；
3. 如果不设置 viewForHeaderInSection 或者设置其为 nil，直接设置 heightForHeaderInSection，不会走 heightForHeaderInSection 这个代理；

### TableView 高度自适应

设置 TableView 高度自适应一般需要设置 estimatedRowHeight 以及 rowHeight 两个属性, 并且 cell 的子控件布局要实现自动布局（即 cell 的子控件需要将 cell 撑满）。其中 estimatedRowHeight 是一个对 cell 的预估高度，为其设置一个非负的预估高度可以提高表视图的性能，将一些几何计算的成本从加载时间推迟到滚动时间（预估高度和实际高度差值越小越好）；rowHeight 设为 UITableView.automaticDimension；
**需要注意的是当 estimatedRowHeight 设为 0 时表示关闭预估高度（在实践中发现可能还会表示不使用自适应），ios11 之前默认值为 0，ios11 及以后默认值为 44，也就是说如果 app 需要兼容 ios10 及以下系统时，必须手动为 estimatedRowHeight 属性赋值，但是我觉得即使不是 ios10 及以下系统，还是建议为该属性手动赋值一个比较接近的值，提升流畅度**

> `cellForRowAtIndexPath`与`heightForRowAtIndexPath`调用顺序：

1. tableView 设置了预估行高
   `cellForRowAtIndexPath` 在 `heightForRowAtIndexPath` 之前调用
2. tableView 没有设置预估行高
   tableView 首先会把所有 IndexPath 的 heightForRowAtIndexPath 遍历一遍以计算 contentsize，然后又按照上述情况设置一次

```swift
//设置预估高度
tableView?.estimatedRowHeight = 44
//设置高度自适应
tableView?.rowHeight = UITableView.automaticDimension

//cell自动布局代码
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
     let tableCell:UITableViewCell = UITableViewCell(style: .default, reuseIdentifier: "cell")
     tableCell.accessoryType = .disclosureIndicator
     let itemUILabel = UILabel()
    //设置换行
     itemUILabel.lineBreakMode = .byWordWrapping
     itemUILabel.numberOfLines  = 0
     tableCell.addSubview(itemUILabel)
     itemUILabel.snp.makeConstraints { (make) in
       make.top.left.right.bottom.equalTo(tableCell) //其中bottom属性比较重要，因为这样cell才知道自己的底部约束，使cell撑开
     }
  return tableCell
 }
```

### TableView 滚动条常在（具体原理百度上很多，ScrollView 同理）

1. UIImageView 扩展 (oc 代理类文件如何新建自行百度)

- .h 文件

```OC
#import <UIKit/UIKit.h>
#define noDisableVerticalScrollTag 836913 //竖向滚动
#define noDisableHorizontalScrollTag 836914 //横向滚动
NS_ASSUME_NONNULL_BEGIN
@interface UIImageView (ForScrollView)
@end
NS_ASSUME_NONNULL_END
```

- .m 文件

```OC
#import "UIImageView+Scroll.h"
@implementation UIImageView (ForScrollView)
- (void)setAlpha:(CGFloat)alpha{

    if (self.superview.tag == noDisableVerticalScrollTag) {
        if (alpha == 0 && self.autoresizingMask == UIViewAutoresizingFlexibleLeftMargin) {
            if (self.frame.size.width < 10 && self.frame.size.height > self.frame.size.width) {
                UIScrollView *sc = (UIScrollView*)self.superview;
                if (sc.frame.size.height < sc.contentSize.height) {
                    return;
                }
            }
        }
    }

    if (self.superview.tag == noDisableHorizontalScrollTag) {
        if (alpha == 0 && self.autoresizingMask == UIViewAutoresizingFlexibleTopMargin) {
            if (self.frame.size.height < 10 && self.frame.size.height < self.frame.size.width) {
                UIScrollView *sc = (UIScrollView*)self.superview;
                if (sc.frame.size.width < sc.contentSize.width) {
                    return;
                }
            }
        }
    }

    [super setAlpha:alpha];
}
@end
```

#### 为 tableView 绑定标签

```swift
 tableView.tag = Int(noDisableVerticalScrollTag)
```

以上代码可以实现 tableView 人工滚动后，滚动条显示后不会再消失。但刚进入不会直接显示滚动条，如果进入页面就直接显示滚动条，则需要添上以下代码；

#### 进入页面直接显示滚动条

```swift
//当tableView绑定完数据后，使用flashScrollIndicators方法
 func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
        tableView.flashScrollIndicators() //该方法会自动显示滚动条1~2秒，再配合1，2步骤代码就可以实现滚动条常在了。
    }
```

### 提供一个获取 tableView 自适应高度比较暴力的方法

近期开发中遇到了一个 ScrollView 嵌套 tableView 的场景，因为 ScrollView 需要依靠其子 View 的相对约束来计算其 ContentSize, 所以需要获取到 tableView 的高度。下面是相关代码。大致思路为**cell 即将展示时，获取 tableView 的 contentSize，此 contentSize 便为 tableView 的高度，将此值更新为 tableView 的高度约束。(可以提前给 tableView 设置一个平均高度，然后去更新这个约束，这样即使没有获取到实际高度，也不会有错误)**

```swift
//在cell即将展示时，获取tableView的contentSize，此contentSize便为tableView的高度，将此值更新为tableView的高度约束
 func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
        tableView.layoutIfNeeded()
        tableView.snp.updateConstraints { make in
            make.height.equalTo(tableView.contentSize.height)
        }
    }
func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
        DispatchQueue.main.asyncAfter(deadline:.now()+0.5){
            QL1(tableView.contentSize.height)
        }
    }
```

### tableCell 点击之后 UIAlertController 延迟弹出问题处理

猜测原因：`点击事件触发后没有及时刷新UI，或者进入到了其他线程，主线程可能经过几次循环后之后才会发现UI变化，去刷新UI`

```swift
//方法1
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: false) //重点是 animated: false ，需要将是否显示动画设为false
}

//方法2，将弹出框操作放在主线程中进行
DispatchQueue.main.async {
  //弹出框操作
}

//方法三
tableCell.selectionStyle = .none  //不要将点击后颜色变化置为none
```

### 6、tablecell 复用导致页面数据错位

#### 1. 可在使用复用的 cell 前，先删除 cell 之前的子 view(代码如下)

#### 2. 为每一个 cell 设置一个特定的 reuseIdentifierId，去缓冲池取的时候直接取特定的 reuseIdentifierId(不推荐使用，如果这样做的话，其中就不存在复用了)

#### 3. 不再使用 dequeueReusableCell 从缓冲池中取，而是直接通过 cellForRow 直接定位到具体的 indexPath(不推荐，原因同上)

```swift
let id = "cellId"
var tableCell = tableView.dequeueReusableCell(withIdentifier: id)
if tableCell == nil{
       tableCell = UITableViewCell(style: .default, reuseIdentifier:id)
}else{
      if (tableCell?.subviews.count)! > 0{
          tableCell?.subviews.forEach{$0.removeFromSuperview()}
       }
}
```

### UITableViewCell 中的使用 cell 和 cell.contentView 的区别

- 进行编辑时，比如 cell 需要向左向右移动用以显示编辑按钮时，使用 cell 子视图不会自动移动；cell.contentView 会自动移动；
- iOS 13 之后，要求必须将视图放在 contentView 上，否则有些控件无法点击；

### UITableView 进行编辑多选模式的步骤

如果遇到进行编辑模式后，cell 没有自动右移的问题解决方式请见[UITableViewCell 中的使用 cell 和 cell.contentView 的区别](#7uitableviewcell%e4%b8%ad%e7%9a%84%e4%bd%bf%e7%94%a8cell%e5%92%8ccellcontentview%e7%9a%84%e5%8c%ba%e5%88%ab)；

#### 设置进行编辑模式的样式（删除、插入等）, 如果不设置，默认为删除形式, 或者 tableview 在未加在 view 上时设置其为编辑模式，也默认为圆圈样式

**cell.selectionStyle = .none** 切记不可将 selectionStyle 设置为 none，这样会使选择以及取消选择的样式都显示不出来。

```swift
/// 多选样式，前面有一个圆圈，
 func tableView(_ tableView: UITableView, editingStyleForRowAt indexPath: IndexPath) -> UITableViewCell.EditingStyle {
        return  UITableViewCell.EditingStyle(rawValue: UITableViewCell.EditingStyle.delete.rawValue | UITableViewCell.EditingStyle.insert.rawValue)!
    }
```

#### 进入编辑模式

```swift
tableView.isEditing = true //进入编辑模式
tableView.allowsMultipleSelectionDuringEditing = true //当编辑模式时允许多选,当开启该选项后，前面的样式不必要再
```

##### 选中及取消选中

```swift
// 选中
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    if tableView.isEditing {
        let cell = self.tableView.cellForRow(at: indexPath)
        cell?.accessoryType = .checkmark
    }
}
// 取消选中
func tableView(_ tableView: UITableView, didDeselectRowAt indexPath: IndexPath) {
    if tableView.isEditing {
        let cell = self.tableView.cellForRow(at: indexPath)
        cell?.accessoryType = .none
    }
}
```

##### 统计选中的项以及关闭编辑模式

```swift
//统计选中的项
let list = tableView.indexPathsForSelectedRows
// 关闭编辑模式
tableView.isEditing = false
关闭编辑后需要记得将所有tablecell的accessoryType复原到进行编辑模式之前的样式
```

##### 额外样式修改

多选时，去除选中情况下的蓝色浮层，只显示左侧圆圈

```swift
override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let tableCell:UITableViewCell = UITableViewCell(style: .default, reuseIdentifier: "cell")
        tableCell.multipleSelectionBackgroundView = UIView()
        return tableCell
    }

/// 设置完上述代码后，然后选中选中发现有的控件背景颜色不见了，需要在自定义cell上实现这样的方法
override func setSelected(_ selected: Bool, animated: Bool) {
         super.setSelected(selected, animated: animated)
         /// 在这里对所有需要设置背景颜色的控件背景颜色重新赋值
    }
```

##### 总结一下，tablecell 多选编辑效果不理想的原因

1. 自定义 cell 上面的控件是否加在 contentView 上

2. cell 的 selectionStyle 不可以设为 none，否则没有选中效果

### UITableView 懒加载中不可以设置 tableFooterView 以及 tableHeaderView，设置会导致崩溃（好像在 ios 13 上已经修复该问题）

```swift
 private lazy var tableView:UITableView = {
        let tableView = UITableView()
        tableView.delegate = self
        tableView.dataSource = self
        // tableView.tableFooterView = UIView() 不可这么写
        // tableView.tableHeaderView = UIView() 不可这么写
        return tableView
    }()
```

### cell 使用自动高度布局时，内部 view 约束高度过小

当 cell 使用自动高度布局时，内部 view 约束高度过小（小于 0.17）时，会出现设置的高度约束无效，cell 实际显示出的高度为 cell 默认高度 44.

### tableview,collectionView 数据 reload 之后的操作无法响应

我们如果要想实现在 reload 之后弹出 alertView，或者滚动到特定一行，可能会直接写：

```swift
tableView.reloadData()
tableView.scrollToRow(at: indexPath, at: .middle, animated: true)
```

复制代码看似没问题，但是滚动没起作用，因为 reloadData 是立即返回的，不会等 tableview 刷新完成。
解决办法就是需要等 reload 完成之后再做我们需要的操作，reload 是否完成有几种方式监听：

```swift
//collectionView
collectionView.performBatchUpdates(nil) { (finished) in
    //reload完成
}
//tableView方法只有iOS11可用
tableView.performBatchUpdates(nil) { (finished) in
    //reload完成
}//替代func beginUpdates()，func endUpdates()
//tableView等reload完成还可以使用
tableView.reloadData()
DispatchQueue.main.async {
    //reload完成
}
```

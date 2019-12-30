---
title: UITableView相关
date: 2019-10-12 13:48:42
categories: [iOS]
tags: [iOS]
---
## 1、基础知识

### 样式

UITableView有两种样式（plain，grouped），其中plain为普通列表样式，grouped是分组样式。可以在实例化的时候进行设置，默认是plain。

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

UITableViewDelegate继承UIScrollViewDelegate，也就是说UITableView其实拥有很多UIScrollView的操作。

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

## 2、问题集锦

### 1、取消TableView前后默认空白部分

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

1. 在设置 **tableView.delegate = self** 时注意将其放在tableView加入其View之前 ( **ParentView.addSubview(tableView)** )，否则上述代码不会生效。
2. 在ios11系统以下，如果上述header以及footer置为0，则会认为其没有设置高度，还是会默认设置大约为50个像素的高度；ios11系统以上可直接设置为0；

### 2、TableView高度自适应

设置TableView高度自适应一般需要设置estimatedRowHeight以及rowHeight两个属性,并且cell的子控件布局要实现自动布局（即cell的子控件需要将cell撑满）。其中estimatedRowHeight是一个对cell的预估高度，为其设置一个非负的预估高度可以提高表视图的性能，将一些几何计算的成本从加载时间推迟到滚动时间（预估高度和实际高度差值越小越好）；rowHeight设为UITableView.automaticDimension；  
**需要注意的是当estimatedRowHeight设为0时表示关闭预估高度（在实践中发现可能还会表示不使用自适应），ios11之前默认值为0，ios11及以后默认值为44，也就是说如果app需要兼容ios10及以下系统时，必须手动为estimatedRowHeight属性赋值，但是我觉得即使不是ios10及以下系统，还是建议为该属性手动赋值一个比较接近的值，提升流畅度**

>`cellForRowAtIndexPath`与`heightForRowAtIndexPath`调用顺序：

1. tableView设置了预估行高  
   `cellForRowAtIndexPath`  在 `heightForRowAtIndexPath` 之前调用
2. tableView没有设置预估行高  
   tableView首先会把所有IndexPath的heightForRowAtIndexPath遍历一遍以计算contentsize，然后又按照上述情况设置一次

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

### 3、TableView滚动条常在（具体原理百度上很多，ScrollView同理）

1. UIImageView扩展(oc代理类文件如何新建自行百度)

- .h文件（如果项目为swift工程，将该文件在桥接文件中声明以供swift使用）

```OC
#import <UIKit/UIKit.h>
#define noDisableVerticalScrollTag 836913 //竖向滚动
#define noDisableHorizontalScrollTag 836914 //横向滚动
NS_ASSUME_NONNULL_BEGIN
@interface UIImageView (ForScrollView)
@end
NS_ASSUME_NONNULL_END
```

- .m文件

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

#### 1. 为tableView绑定标签

```swift
 tableView.tag = Int(noDisableVerticalScrollTag)
```

以上代码可以实现tableView人工滚动后，滚动条显示后不会再消失。但刚进入不会直接显示滚动条，如果进入页面就直接显示滚动条，则需要添上以下代码；

#### 2. 进入页面直接显示滚动条

```swift
//当tableView绑定完数据后，使用flashScrollIndicators方法
 func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
        tableView.flashScrollIndicators() //该方法会自动显示滚动条1~2秒，再配合1，2步骤代码就可以实现滚动条常在了。
    }
```

### 4、提供一个获取tableView自适应高度的方法

近期开发中遇到了一个ScrollView嵌套tableView的场景，因为ScrollView需要依靠其子View的相对约束来计算其ContentSize,所以需要获取到tableView的高度。下面是相关代码。大致思路为**在最后一个cell即将展示完毕之后，获取tableView的contentSize，此contentSize便为tableView的高度，将此值更新为tableView的高度约束。(可以提前给tableView设置一个平均高度，然后去更新这个约束，这样即使没有获取到实际高度，也不会有错误)**

```swift
//在最后一个cell即将展示完毕之后，获取tableView的contentSize，此contentSize便为tableView的高度，将此值更新为tableView的高度约束
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

### 5、tableCell点击之后UIAlertController延迟弹出问题处理

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

### 6、tablecell复用导致页面数据错位

#### 1.可在使用复用的cell前，先删除cell之前的子view(代码如下)

#### 2.为每一个cell设置一个特定的reuseIdentifierId，去缓冲池取的时候直接取特定的reuseIdentifierId(不推荐使用，如果这样做的话，其中就不存在复用了)

#### 3.不再使用dequeueReusableCell从缓冲池中取，而是直接通过cellForRow直接定位到具体的indexPath(不推荐，原因同上)

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

### 7、UITableViewCell中的使用cell和cell.contentView的区别

**进行编辑时，比如cell需要向左向右移动用以显示编辑按钮时，使用cell子视图不会自动移动；cell.contentView会自动移动；** 其他情况下两者基本没有什么区别

### 8、UITableView进行编辑多选模式的步骤

如果遇到进行编辑模式后，cell没有自动右移的问题解决方式请见[UITableViewCell中的使用cell和cell.contentView的区别](#7uitableviewcell%e4%b8%ad%e7%9a%84%e4%bd%bf%e7%94%a8cell%e5%92%8ccellcontentview%e7%9a%84%e5%8c%ba%e5%88%ab)；

#### 1.设置进行编辑模式的样式（删除、插入等）,如果不设置，默认为删除形式,或者tableview在未加在view上时设置其为编辑模式，也默认为圆圈样式

**cell.selectionStyle = .none** 切记不可将selectionStyle设置为none，这样会使选择以及取消选择的样式都显示不出来。

```swift
/// 多选样式，前面有一个圆圈，
 func tableView(_ tableView: UITableView, editingStyleForRowAt indexPath: IndexPath) -> UITableViewCell.EditingStyle {
        return  UITableViewCell.EditingStyle(rawValue: UITableViewCell.EditingStyle.delete.rawValue | UITableViewCell.EditingStyle.insert.rawValue)!
    }
```

#### 2.进入编辑模式

```swift
tableView.isEditing = true //进入编辑模式
tableView.allowsMultipleSelectionDuringEditing = true //当编辑模式时允许多选
```

##### 1.选中及取消选中

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

##### 2.统计选中的项以及关闭编辑模式

```swift
//统计选中的项
let list = tableView.indexPathsForSelectedRows
// 关闭编辑模式
tableView.isEditing = false
关闭编辑后需要记得将所有tablecell的accessoryType复原到进行编辑模式之前的样式
```

##### 3.额外样式修改

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

##### 总结一下，tablecell多选编辑效果不理想的原因

1. 自定义cell上面的控件是否加在contentView上

2. cell的selectionStyle不可以设为none，否则没有选中效果

### 8、UITableView懒加载中不可以设置tableFooterView以及tableHeaderView，设置会导致崩溃

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

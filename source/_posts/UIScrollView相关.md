---
title: UIScrollView相关
date: 2019-10-12 14:45:16
categories: [iOS]
tags: [iOS]
---
# 一、UIScrollView
1. UIScrollView特殊点
- ios根View的焦点一般都是(0,0)，会从屏幕左上角开始布局，如果view第一个子视图是UIScrollView时，则view视图会自动下移大概64pt（状态栏20pt + 导航栏44pt）。如果不是第一个视图，并不会自动向下偏移，如果没有经过特殊布局，便会出现其内容被导航栏遮挡的问题出现（iOS 11.0系统以下），如果使用下面代码处理一下。
```
 self.edgesForExtendedLayout = UIRectEdge.init(rawValue: 0)
 self.automaticallyAdjustsScrollViewInsets = true

 self.navigationController?.navigationBar.backgroundColor = .white //使用上面语句处理后会出现导航栏变灰色问题，需要人工改变导航栏颜色
```
- 其他View利用snpkit布局时，一般子视图的约束会依赖于父视图，但是UIScrollView的contentSize是根据子视图来决定，意思就是父视图约束依赖于子视图，所以我们必须设置子视图的宽高；
2. 设置约束
利用snpkit自动布局库处理UIScrollView时，一般处理方法是：Scrollview里面放一个ContainView，然后子视图拉约束到ContainView，这样ContainView的大小就可以根据子视图来变化，Scrollview的大小根据ContainView来定。
其中UIScrollView如果设置是全局滚动时，一般可设置其约束为make.edges.equalToSuperview()，如果部分滚动时，宽高属性约束都不可少。（具体见示例代码）

# 二、代码示例
下面代码实现效果如下图所示，其中上面标题部分是一个固定view，下面红色部分是可滑动部分，可以根据以下代码体会以下UIScrollView在利用snpkit设置约束的不同之处；

![UIScrollView实现效果图.png](/UIScrollView相关/UIScrollView实现效果图.png)

```
import Foundation
import UIKit
import SnapKit

// ConstantsHelp.SCREENHEIGHT为屏幕高度，需要自己获取
// ConstantsHelp.SCREENWITH为屏幕宽度，需要自己获取
class UIScrollViewExampleViewController:UIViewController{
    
    var titleInfo = "UIScrollView"
    override func viewDidLoad() {
        super.viewDidLoad()
        title = titleInfo
        initView()
    }
    
    
    func  initView(){
        
        let navRect = self.navigationController?.navigationBar.frame
        let navigationItemHeight = navRect?.size.height //导航栏高度
        let statusBarHeight = UIApplication.shared.statusBarFrame.height //状态栏高度
        let topMarginHeight = navigationItemHeight! + statusBarHeight
        
        view.backgroundColor = UIColor.white
        
        let titleView = UIView()
        view.addSubview(titleView)
        titleView.snp.makeConstraints{ (make) in
            make.top.equalToSuperview().offset(topMarginHeight+10) //需要加上导航栏以及状态栏高度
            make.left.right.equalToSuperview()
        }
        
        let titleUILabel = UILabel()
        titleView.addSubview(titleUILabel)
        titleUILabel.snp.makeConstraints{ (make) in
            make.top.left.right.equalToSuperview()
        }
        titleUILabel.numberOfLines = 0
        titleUILabel.lineBreakMode = .byWordWrapping
        titleUILabel.text = "这是一个很长很长很长很长很长很长很长很长很长很长很长很长很长的标题"
        
        let lineView = CommonViews.getSeparatorUIView()
        titleView.addSubview(lineView)
        lineView.snp.makeConstraints{ (make) in
            make.bottom.equalTo(titleUILabel.snp.bottom).offset(ConstantsHelp.normalPadding)
        }
        
        titleView.snp.makeConstraints{ (make) in
            make.bottom.equalTo(lineView.snp.bottom).offset(ConstantsHelp.normalPadding)
        }
        
        let scrollView = UIScrollView()
        scrollView.delegate = self
        view.addSubview(scrollView)
        scrollView.alwaysBounceVertical = true
        scrollView.isScrollEnabled = true
        scrollView.backgroundColor = UIColor.white
//      scrollView.showsVerticalScrollIndicator = false //垂直滑动线隐藏
        
        // 设置时必须要设置宽高约束，一般需要全屏滚动的话可以使用make.edges.equalToSuperview()
        scrollView.snp.makeConstraints{(make) in
            make.top.equalTo(titleView.snp.bottom)
            make.width.equalToSuperview()
            make.bottom.equalToSuperview()
        }
        
        let contentView = UIView()
        scrollView.addSubview(contentView)
        contentView.backgroundColor = UIColor.red
        contentView.snp.makeConstraints{(make) in
            make.top.equalToSuperview()
            make.left.equalToSuperview().offset(ConstantsHelp.leftMargin)
            make.width.equalTo(ConstantsHelp.SCREENWITH - CGFloat(ConstantsHelp.leftMargin*2)) //重要，必须使用width属性而不是make.left.right.equalToSuperview() 形式
            make.height.equalTo(ConstantsHelp.SCREENHEIGHT) //设定高度，重要
        }
        
        scrollView.snp.makeConstraints{(make) in
            make.bottom.equalTo(contentView.snp.bottom)
        }
        
    }
    
}

extension UIScrollViewExampleViewController:UIScrollViewDelegate{
    func scrollViewWillBeginDragging(_ scrollView: UIScrollView) {
        print("将要开始拖动")
    }
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        print("已经滑动")
    }
    func scrollViewDidEndDragging(_ scrollView: UIScrollView, willDecelerate decelerate: Bool) {
        print("停止拖动")
    }
    func scrollViewShouldScrollToTop(_ scrollView: UIScrollView) -> Bool {
        print("将要滚动到顶部")
        return true
    }
    func scrollViewDidScrollToTop(_ scrollView: UIScrollView) {
        print("已经滚动到顶部")
    }
    func scrollViewDidZoom(_ scrollView: UIScrollView) {
        print("正在缩放")
    }
    func scrollViewDidEndZooming(_ scrollView: UIScrollView, with view: UIView?, atScale scale: CGFloat) {
        print("以及缩放完毕")
    }
}
```

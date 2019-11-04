---
title: iOS开发杂记
date: 2019-10-12 14:12:08
categories: [iOS]
tags: [iOS]
---
## 1、UITapGestureRecognizer相关

* **一个手势添加到多个view上，只有最后一个view添加的有效**

>每一个Gesture Recognizer关联一个View，但是一个View可以关联多个Gesture Recognizer,一个View可以响应多种触控操作方式。当一个触控事件发生时，Gesture Recognizer接收一个动作消息要先于View本身，结果就是Gesture Recognizer作为View处理触控事件的代理。当Gesture Recognizer接收到指定的事件时，它就会发送一条动作消息(action message)给ViewController并处理。

```swift
//labelOne点击事件会失效
let singleTapGesture = UITapGestureRecognizer(target: self, action: #selector(changeAgreeState))

labelOne.addGestureRecognizer(singleTapGesture)
labelOne.isUserInteractionEnabled = true

labelTwo.addGestureRecognizer(singleTapGesture)
labelTwo.isUserInteractionEnabled = true
```

## 2、元素被导航栏遮挡

```swift
self.edgesForExtendedLayout =  UIRectEdge.init(rawValue: 0)
//设置edgesForExtendedLayout后会导致导航栏颜色变灰，需要人工改变导航栏颜色
self.navigationController?.navigationBar.backgroundColor = .white
```

## 3、页面向下偏移

当view的第一个页面是scrollView或者tableView时，页面会自动向下偏移64pt，如果已经设置了top约束，则页面就会错位，使用下面代码使scrollView或者tableView不要自动向下偏移。

```swift
if #available(iOS 11.0, *) {
   scrollView.contentInsetAdjustmentBehavior = UIScrollView.ContentInsetAdjustmentBehavior.never
  //tableView.contentInsetAdjustmentBehavior = UIScrollView.ContentInsetAdjustmentBehavior.never
}else{
   self.automaticallyAdjustsScrollViewInsets = false
}
```

## 4、控制页面view在安全区域内

```swift
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

## 5、启动图配置方法说明

[启动图配置](https://www.jianshu.com/p/05b31f29b135)

## 6、ViewController的生命周期

```swift
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

## 7、启动图所有尺寸

![启动图尺寸1.png](/iOS开发杂记/启动图尺寸1.png)
![启动图尺寸2.png](/iOS开发杂记/启动图尺寸2.png)

## 8、view添加阴影

这种方式是使用离屏渲染的方式，性能会有很大的损耗

```swift
view.backgroundColor = .white //设置view背景色，如果不设置，会导致阴影显示不出来或者阴影加在子view上
view.layer.masksToBounds = false //设置子view允许超出父view
view.layer.shadowColor = UIColor.black.cgColor //设置阴影颜色
view.layer.shadowOffset = CGSize.init(width: 0, height: 1) //设置阴影偏移量
view.layer.shadowOpacity = 0.15 //设置阴影透明度
view.layer.shadowRadius = 3 //设置阴影半径
```

## 9、关于一些关键字的理解以及注意事项

主要关键字包括 **let var static lazy**等等

* lazy：延迟加载，一般用于修饰属性，只可以标识变量（var修饰的属性）；

lazy修饰属性后，当第一次访问这个属性时，会判断这个属性是否已经初始化，如果已经初始化会直接返回，如果没有，会进行初始化。一般lazy修饰的属性会采用闭包的方式进行初始化。(类初始化的时候不会初始化被lazy修饰的属性)

```swift
//修饰属性范例
lazy var titleLabel:UILabel = {
       let titleLabel = UILabel()
       titleLabel.text = "这是标题"
       return titleLabel
    }()
//与map filter等接受闭包运行的方法一起使用时
let arr = ["1","2"]
let list = arr.lazy.compactMap{ (info:String)->String in
   print("内部--\(info)")
   return "\(info)"
}
for item in list{
    print("外部--\(item)")
}
//结果
   内部--1
   外部--1
   内部--2
   外部--2
```

* let标识常量，var标识变量；

let修饰的属性为线程安全的。let修饰的属性只可以被赋值一次，如果在方法中，除了使用这种形式外，还可以使用`let number:Int;number = 1`这种形式，不过两种形式赋值一次之后都不可以再次赋值了。

* static 用于限定作用域，可以修饰方法以及属性。

```swift
class Info{
   static let name = "张三"
}
print(Info.name) //输出张三
print(Info().name) //报错，这点与Java不同，java中允许这种使用
```

**static let** 修饰属性时，属性会自动进行懒加载，并且是线程安全的
**static var** 修饰属性时，属性的初始化时机不确定

附上初始化的一部分写法

```swift
// 写法1
static let redLabel:UILabel = {
        $0.backgroundColor = .red
        $0.text = "红色"
        return $0
    }(UILabel())

// 写法2
static let redLabel:UILabel = {
        let redLabel = UILabel()
        redLabel.backgroundColor = .red
        redLabel.text = "红色"
        return redLabel
    }()

```

## 10、class和static的相同点和区别

### 1、相同点

* 都可以用来修饰方法，其中class修饰的叫做类方法，static修饰的叫做静态方法；
* 都可以用来修饰计算属性；

### 2、区别

* class只可以用于类中，修饰的方法以及计算属性可以被重写
* static可以用于类、结构体以及枚举中使用，修饰后的方法以及属性不可以被重写；static可以修饰存储属性，修改后的属性叫做静态变量（常量）；
* class、struct以及enum都可以实现protocol，其中在protocol定义时使用static，enum以及struct去实现时使用static，class去实现时class以及static都可以使用；

## 11、计算属性、存储属性以及类型属性

* 类型属性：被static修饰的属性；可以用在枚举、类中；
* 存储属性：存储变量或常量，可以用在结构体以及类中；
* 计算属性：不直接存储值，而是通过get以及set方法来赋值以及取值，同时对其他属性进行操作，类似一个方法；在类、结构体以及枚举中都可以使用；必须用var修饰、属性的类型不可以忽略、如果想修改属性的值，必须写set方法，否则只有一个get方法；

```swift
// 错误写法，这种写法会进入死循环
var name: String {
        get {
            return self.name
        }
        set {
            self.name = newValue
        }
    }
```

## 12、init构造方法总结

无论是结构体、类还是枚举，除非是在定义属性的时候已经给属性进行赋值，否则必须在init方法中对其进行赋值。

```swift
class Student{
   var sex:String! // var修饰的属性这种形式的写法 会 为属性自动设置一个nil的初始值，构造方法中不必要对其进行赋值了
   let name:String! //let修饰的属性这种形式的写法 不会 为属性设置初始值，必须通过构造方法进行赋值或者使用 let name:String! = nil这种方式的写法
}
```

便利构造函数中，一定不会有super，并且方法的第一句是调用自身的构造方法，然后再对属性赋值，只有self被初始化时候才可以对属性进行赋值，这也导致其赋值的属性不可以用let修饰。

* **结构体**
  * 对应结构体来说，会有一个默认的构造器，默认的构造器会带出所有的属性；当自定义初始化方法后，默认的初始化方法就没有了，必须使用自定义的初始化方法；
* **类**
  * 对于类来说，不会有默认的构造方法，如果想使用构造器对属性进行赋值就必须自定义构造方法；
  * 一旦自定义了一个构造方法，其原本的初始化方法（比如从父类继承而来）都不可以再使用，并且在自定义的构造方法中，必须先对属性进行赋值，然后再调用父类的构造方法。如果想使用默认的初始化方法，请重写这个方法，并进行相应的赋值后，调用父类的构造方法；
  
## 13、alpha、hidden、opaque、opacity以及isUserInteractionEnabled等属性的解析

### 1、isUserInteractionEnabled

用于控制view及其子view是否可以接收响应事件；当设为false时，该视图对象会从响应链中被移除，响应事件会传递到view的父视图。  

* UIImageView的isUserInteractionEnabled默认为false;

* UILabel的isUserInteractionEnabled默认为false;

* UIView的isUserInteractionEnabled默认为true;

### 2、hidden

控制view是否隐藏，当设置为true时，自身以及子view均会被隐藏，不管subView的hideen是否为true，并且当前view以及子view会从响应链中移除；

### 3、alpha

控制view的透明度，是一个浮点值，取值范围0~1.0,表示从完全透明到完全不透明；会影响自己的透明度，也会影响subView的透明度，当alpha为0，当前view以及子view会从响应链中移除；更改alpha默认是有动画效果的。当使用alpha属性来隐藏view，使用hidden比使用alpha性能好。

### 4、opacity

opacity是CALayer的属性，对应的是UIView的alpha。

### 5、opaque

表示view的不透明度，设为true表示不透明。但是它决定不了当前view是否不透明，只是为绘图系统提供一个性能优化开关，当设为true时，绘图系统在绘制该视图时会将整个视图当做不透明来对待。能将opaque设为true的尽量将opaque设为true。至于什么场景下会使用fasle，我也是不太清楚啊...

---
title: Swift语法相关
date: 2019-10-12 11:53:11
categories: [Swift]
tags: [Swift]
---
## 1、字符串

### 1、subString与String

- subString是一个全新的类型，看类名像是String的子类，但是大家千万别被误导了，Substring并不是String的子类，这是两个不同的类型，但是它们都继承了StringProtocol协议，因此存在一些共性；在开发中Substring并不常用(一般会在分割String中见到)，所以往往要转成String。
- subString最重要的作用的体现是在性能上，自身在分割时，其实重用了父String的内存，延长了父String的生命周期，当这个subString不被释放，其父String也不被释放。

### 2、字符串分割

我们可以使用两种方式对字符串进行分割，分别为split以及components方法，两种方法在类型以及分割结果上有着不同结果；

- split分割后的结果为[Substring]类型，如果想把分割的结果转换成[String]类型，可以使用compactMap进行转换，示例如下

```swift
let arryStr:[String] = str.split(",").compactMap{"\($0)"}
```

- components分割结果为[String]类型

结果比对

```swift
import Foundation

let x = ".1.2.3.4.5.."
print("使用split对字符串分割忽略空值结果：\(x.split(separator: "."))")
print("使用split对字符串分割不忽略空值结果：\(x.split(separator: ".",omittingEmptySubsequences:false))")
print("使用components对字符串进行分割结果：\(x.components(separatedBy: "."))")

//结果
使用split对字符串分割忽略空值结果：["1", "2", "3", "4", "5"]
使用split对字符串分割不忽略空值结果：["", "1", "2", "3", "4", "5", "", ""]
使用components对字符串进行分割结果：["", "1", "2", "3", "4", "5", "", ""]

//我们可以直接利用String()函数将subString类型转成String
```

## 2、Map、filter、reduce、flatMap以及compactMap的使用

首先明确这些高阶函数内部实现机制实际也是循环，是比较耗时的操作，大家使用时要考虑到这一点。

### 1、Map  

map方法用于对数据中的每一个元素进行处理，然后返回一个新数组，新数组可以是旧数组中的部分属性组成的；

**仿写原理**

```swift
extension Array{
    func customMap<T>(transform:((Element) -> T)) -> [T]{
        var rs:[T] = []
        for element in self {
            rs.append(transform(element))
        }
        return rs
    }
}
```

**示例Demo**

```swift
let str = [12,13,14,15,16,17]
let str1 = str.map{"\($0)" + "元"} //$0表示数组中每一个元素
print(str1)
//结果
["12元", "13元", "14元", "15元", "16元", "17元"]
```

### 2、filter

filter函数用于将列表中元素进行过滤，筛选出数组中满足某种文件的元素组成新数组；

**仿写原理**

```swift
extension Array{
    func customFilter(includeElement:(Element) -> Bool) -> [Element] {
        var rs:[Element] = []
        for x in self {
            if includeElement(x) == true {
                rs.append(x)
            }
        }
        return rs
    }
}
```

**示例Demo**
```swift
let str = [12,13,14,15,16,17]
let str2 = str.filter{$0 < 13 || $0 > 15} //$0表示数组中每一个元素
print(str2)
//结果
[12, 16, 17]
```

### 3、reduce

**仿写原理**

```swift
extension Array{
    func customReduce<T>(initial:T, combine:(T,Element)->T) -> T {
        var rs = initial
        for x in self {
            rs = combine(rs, x)
        }
        return rs
    }
}
```

**示例Demo**

reduce函数用于将数组元素组合计算为一个计算值，并且接受一个初始值，这个初始值的类型可以与数组元素的类型不同；

```swift
let str = [12,13,14,15,16,17]
let str3 = str.reduce(20){$0 + $1} //$0表示累积器，$1表示数组中每个元素
print(str3)
//结果
107
```

### 4、flatMap

flatMap函数与map函数作用类似，不同点在于它返回的数组中不含有nil（会自动将nil剔除），同时将Optional解包，并且它可以将多维数组拆分变成一维数组，但map不可以。该语法已经在Swift4.1中弃用了，不提供代码示例，代码示例见 5. compactMap，该方法只返回字符数组。

### 5、compactMap

swift4.1语法中，使用compactMap函数替代flatMap函数,同时返回的不仅仅是字符数组了，还可以是数字型数组了。

```swift
//自动去除nil以及解包
let array = ["Apple", "Orange", "Grape", ""]
let arr1 = array.map{ a ->Int?in
    let  length = a.count
    guard  length > 0 else{return  nil}
    return  length
}
let arr2 = array.compactMap{ a ->Int?in
    let  length = a.count
    guard  length > 0 else{return  nil}
    return  length
}
print(arr1)
print(arr2)
//结果
[Optional(5), Optional(6), Optional(5), nil]
[5, 6, 5]

//多维数组
let array1 = [[1,2,3],[7,8,9],[4,5,6]]
let arr11 = array1.map{ $0 }
let arr22 = array1.flatMap{ $0 }
print(arr11)
print(arr22)
//结果
[[1, 2, 3], [7, 8, 9], [4, 5, 6]]
[1, 2, 3, 7, 8, 9, 4, 5, 6]
```

>一般情况下可以使用简洁写法`list.compactMap{"\($0)"}`,即省略最外面的()以及return,但是当代码逻辑比较复杂时(比如遍历后的结果需要拼接之类的)，需要加上()

### 6、forEach

循环遍历list

```swift
let array = ["Apple", "Orange", "Grape", ""]
array.forEach{
            print($0)
        }
```

## 3、List数组处理

### 1、sort函数

Swift中自带sort函数，可为数组、字典等进行排序。(Swift4.1以后为sorted)，代码示例如下：

```swift
//数组字典排序
let nameList = [
    ["name":"李二","code":"0002","sort":"2"],
    ["name":"李一","code":"0001","sort":"1"],
    ["name":"李三","code":"0003","sort":"3"],
]
//正序
let sortList = nameList.sorted{ (list1,list2) ->Bool in //普通写法
    return list1["sort"]! < list2["sort"]! 
}
//倒序
let sortList1 = nameList.sorted{$0["sort"]! > $1["sort"]! } //稍高级写法，其实就是将两个闭包返回参数用$0以及$1代替了，下面两种写法也可以
// let sortList2 = nameList.sorted(){$0["sort"]! > $1["sort"]! }
// let sortList2 = nameList.sorted(by:{$0["sort"]! > $1["sort"]! })

print(sortList)
print(sortList1)
//正序结果
[["sort": "1", "name": "李一", "code": "0001"], ["sort": "2", "name": "李二", "code": "0002"], ["sort": "3", "name": "李三", "code": "0003"]]
//倒序结果
[["sort": "3", "name": "李三", "code": "0003"], ["sort": "2", "name": "李二", "code": "0002"], ["sort": "1", "name": "李一", "code": "0001"]]
```

字符串排序以及字典排序与上面方法类似，其中字符串排序可更加省略闭包参数，如 `let sortStr = str.sort(<)`,字典排序用0以及1代替key以及value，如根据key值排序代码， `let result = dic.sort {$0.0 < $1.0}`。

## 4、JSON相关

swift中json方面的处理一般有两种方式(资料参照:[资料](https://www.cnblogs.com/CoderEYLee/p/Object-C-0021.html))，一种是原生形式，一种是SwiftyJSON第三方库形式(git地址为：[SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON))；

1. json字符串转对象

```swift
//原生
//json字符串 -> Data -> 对象(Any类型)
let jsonString = "{\"name\":\"zhangsan\",\"age\": 18}"
let jsonData = jsonString.data(using: String.Encoding.utf8, allowLossyConversion: false) ?? Data()
guard let anyObject = try? JSONSerialization.jsonObject(with: jsonData, options: .mutableContainers) else {
     return
}

// SwiftJSON
// 特别注意，直接使用JSON(jsonString).dictionaryValue 这种形式转换无法转换出正常结果
// json字符串 -> Data -> 对象
let data = jsonString.data(using: String.Encoding.utf8, allowLossyConversion: false) ?? Data()
let json = JSON(data).dictionaryValue
```

## 5、限制符

swift的限制符分为五类，分别为`open` > `public`>`internal` > `fileprivate` > `private`，其中`internal`为默认限制符

### 1、 **private**  

对于属性来说, private 最为严格, 只能在声明的地方使用, 例如, 在 class 中声明一个 private 属性, 除了在这个类中能使用之外(swift 4后可以在extension中使用，不过这个类的extension必须和类在同一个物理文件), 其他地方无法使用。用于类时, 可以在当前模块实例化, 继承, 不可跨模块访问。值得注意的是, private 修饰的类, 依旧可以在外部, 别的文件中访问, 实例化。

### 2、**fileprivate**  

整体类似于 private, 区别在于限制范围由定义所在域 改为 当前文件。

### 3、**internal**

这个是默认访问控制符, 表示在当前模块内可访问, 超出此模块不可访问。

### 4、 **public**

用此方法申明的类, 可以跨模块访问, 不过不能跨模块继承。

### 5、**open**

无任何限制

## 6、extension扩展

* 子类如果想要重写父类的extension中的方法，需要在父类方法上加上@objc的修饰符
* 当用某一个限制符修饰扩展时，其扩展内部方法以及计算属性等默认都会使用该限制符，如果其中需要比较小的限制符，需要手动加上
* 扩展中不可以使用存储属性，只可以有计算属性以及方法

## 7、Swift以及OC

### 1、Swift与OC互相调用

1. **Swift调用OC**  
   Swift调用OC需要借助桥接文件，将OC的.h文件在桥接文件中引入，然后swift就可以直接使用在.h文件中定义的接口以及属性了；
2. **OC调用Swift**  
   OC调用swift时，需要在调用的的OC文件中引入一个.h头文件，其格式为 #import "CoderStar-Swift.h" ，其中CoderStar为整个工程的名称。引入后需要重新编译一下。（实际编译会将工程的所有.swift文件中涉及到的接口以及属性全都整合到这个.h文件中，进入该文件中就能明白了）。

## 8、Swift中的设计模式

### 1、单例模式

单例模式保证程序在运行过程中一个类只会生成一个实例。
单例模式中，类的构造方法必须进行重写，然后使其私有化，这样才能防止其他对象使用这个类的默认的'()'初始化方法来创建对象。
下面是单例模式的两种写法，其中原理差不多

- 写法1
  
```swift
class SingletonHelp: NSObject {
    private override init() {} // 构造方法必须私有化
    private static let singletonHelp = SingletonHelp()
    class func getSingletonHelp() -> SingletonHelp {
        return singletonHelp
    }
    var isTimeOut:Bool = false //属性
}

let isTimeOut = SingletonHelp.getSingletonHelp().isTimeOut // 获取单例属性
```

- 写法2
  
```swift
class SingletonHelp: NSObject {
    private override init() {} // 构造方法必须私有化
    static let singletonHelp = SingletonHelp()
    var isTimeOut:Bool = false //属性
}

let isTimeOut = SingletonHelp.singletonHelp.isTimeOut // 获取单例属性
```

## 9、OptionSets(选项集合)

swift的enum是枚举类型，只支持单选；要想构造选项集合的结构，需要使用struct来遵从OptionSet协议。代码示例

```swift
//因为 OptionSet中已经有了public init(rawValue: Self.RawValue)构造方法，所以我们只需要在Sex中建立rawValue就可以了
struct Sex:OptionSet{
    let rawValue:Int
    //rawValue的值需要注意，必须是2的整数次幂，因为集合的本质就是位运算
    static let man = Sex(rawValue:1)
    static let woman = Sex(rawValue:2)
}
class Test{
    static func showSex(sex:Sex){
        if sex.contains(.man){
            print("man")
        }
        if sex.contains(.woman){
            print("woman")
        }
      print(sex)
    }
}
Test.showSex(sex:[.man,.woman])
```

## 10、其他汇总

### 1、**class func 与static func 之间的区别**  

**static func** 禁止被子类重写，相当于 **class final func**

### 2、标识符使用关键字的写法

在swift，一些关键字可以通过加上``成为标识符，如下面这些写法

```swift
class 'class' {

}
enum sex {
    case `default`
}
```

## 11、isKindOf、isMemberOf、isEuqal、is以及==之间区别

**1、** isKindOf、isMemberOf以及isEuqal都是NSObjectProtocol协议中定义的方法，类需要继承自NSObject或者实现了NSObjectProtocol协议)才可以使用，其中isMemberOf判断某个对象是否是某一个类的实例，isKindOf判断某个对象是否为某个类及其子类的实例，isKindOf的范围更大点；isEuqal主要用于比较两个对象是否是同一个对象，hash值是否相同。

```swift
class Info:NSObject {

}
class SubInfo:Info {

}
let info = Info()
let subinfo = SubInfo()

print(subinfo.isKind(of: Info.self)) //true
print(subinfo.isKind(of: SubInfo.self)) //true
print(subinfo.isMember(of: Info.self)) //false
print(info.isMember(of: Info.self)) //true

```

**2、** ==是Equatable协议中的方法，如果想使用==这种方式比较两个对象之间的关系，其对应类就必须实现Equatable协议，结构体等一样。另外补充一下，Equatable协议是比较相等(里面有 == 以及 != )，还有一个Comparable协议是比较顺序，里面有>,<,<=,>=这些。现在大部分基础类就以已经实现了这些协议(NSObject,基础类型等)，而且大部分实现时比较两者值是否相等。

**3、** is用于任何类型的判断，包括对象类型(支持子类)以及非对象类型；判断实例是否是某一类型。

## 12、swift中闭包的相关理解

swift中的闭包（block）分为非逃匿闭包以及逃匿闭包两种。

## 13、关于一些关键字的理解以及注意事项

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

**static let** 修饰属性时，属性会自动进行懒加载，并且是线程安全的，只有第一次去访问它的时候才会进行赋值。
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

## 14、class和static的相同点和区别

### 1、相同点

* 都可以用来修饰方法，其中class修饰的叫做类方法，static修饰的叫做静态方法；
* 都可以用来修饰计算属性；

### 2、区别

* class只可以用于类中，修饰的方法以及计算属性可以被重写
* static可以用于类、结构体以及枚举中使用，修饰后的方法以及属性不可以被重写；static可以修饰存储属性，修改后的属性叫做静态变量（常量）；
* class、struct以及enum都可以实现protocol，其中在protocol定义时使用static，enum以及struct去实现时使用static，class去实现时class以及static都可以使用；

## 15、计算属性、存储属性以及类型属性

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

## 16、init构造方法总结

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

## 17、OC的Block与Swift的Closure之间互相转换

**Block转为Closure**

```swift
// block为获取的oc Block，后面(String) -> Void 为Block的类型 callback为转换后的Closure
typealias CallbackType = @convention(block) (String) -> Void
let blockPtr = UnsafeRawPointer(Unmanaged<AnyObject>.passUnretained(block as AnyObject).toOpaque())
let callback = unsafeBitCast(blockPtr, to: CallbackType.self)
```

**Closure转为Block**

``` swift
// callback为swift的Closure，block为一个常量，不太明白为什么要这么写
let callbackBlock = callback as @convention(block) (String) -> Void
let callbackBlockObject = unsafeBitCast(callbackBlock, to: AnyObject.self)
```

## 18、protocol相关

### 1、限制协议protocol只能有类去实现

```swift
protocol MyProtocol: class {

}
```
或者
```swift
protocol MyProtocol: AnyObject {

}
```

### 2、当protocol被struct实现
因为struct中的方法如果对属性进行修改时，需要在方法前面加上`mutating`关键字，所以当protocol可能被struct实现时，需要注意将protocol中的部分方法前面加上`mutating`关键字

### 3、可选protocol

OC方法

```swift
@objc protocol SomeProtocol {
    func requiredFunc()
    @objc optional func optionalFunc()
}
```

Swift方法

```swift
protocol SomeProtocol {
    func requiredFunc()
    func optionalFunc()
}
extension SomeProtocol {
    func optionalFunc() {
       print(“Dumb Implementation”)
    }
}
```
---
title: Xcode Tips
category:
  - iOS
  - Xcode
tags:
  - Xcode
date: 2022-03-19 16:57:47
---

## 前言

Hi Coder，我是 CoderStar！

今天我们不聊技术原理，咱们聊点简单的，也就是我们 `iOSer` 几乎每天都会用到的`Xbug`。`Xcode`虽然确实会有很多`Bug`，一些设计也不如`JB`家的做的好，但是还是有一些可取之处的，比如页面简洁...，嗯...，好像就这一个？

哈哈，简单开个玩笑，回到正题。虽然我们经常使用`Xcode`，但是有些功能还是需要我们自己特意去发现一下。今天我们就来聊聊`Xcode`的一些`Tips`。

> 有些`Tips`可能对于老司机们已经习以为常了，还望不要嫌太低级，如果还有一些文中没有体现的`Tips`，还望指教。

## 编辑相关

### Refactor

我们把光标放在类上或者方法上右键选中`Refactor`，其会显示出对其光标处可以进行的自动补全的一些操作；如下图所示：

![Refactor](../../../img/iOS/Xcode/XcodeTips/Refactor.png)

大家根据名字就能看出来支持的一些操作了，基本上都是些很常用的功能。比如说

- `Rename`：将光标选中处涉及到所有的统一进行改名；
- `Generate Memberwise Initialzer`：利用这功能，当我们利用非常多属性的类 / 结构体时，就可以使用这个快速生成构造函数了；
- `Add Missing Switch Cases`：填充缺失的`case`，虽然说`Xcode`会自动提示`Fix`补全，但是在电脑提示较慢的时候还是很难受的；
- ...

### Actions

还是将光标放到类或者方法上，然后 `Command` + `左键`，就会出现下列的`Actions`选项，看名字大家就知道大概支持哪些操作。

![Actions](../../../img/iOS/Xcode/XcodeTips/Actions.png)

> 之前还有小伙伴在群里抱怨`Xcode`没有`Callers`的功能，这不是来了嘛...

使用 `option` + `左键` ，会显示相应的`Quick Help`，也就是可以看到光标选中的相关提示，如果是系统相关的类或方法，还可以通过其进入到对应的`Developer Documentation`。

使用 `control` + `Command` + `左键`，会直接跳转到光标选中处的相关实现里面去。

### Navigate to Related Items

![Navigate to Related Items](../../../img/iOS/Xcode/XcodeTips/Navigate%20to%20Related%20Items.png)

大家可以很少关注上图所示的按钮，其实这个地方能提供很多有用的功能。

就不一一介绍了，主要介绍下`Generated Interface`功能，该功能可以查看 OC 的`.h`文件生成对应的`.swift`文件是什么样子，在处理混编时候比较常用；

### Check Spelling

这项功能为`Xcode`的拼写检查，开启方式为

`Edit` > `Format` > `Spelling and Grammar` > `Check Spelling While Typing`。

![Check Spelling](../../../img/iOS/Xcode/XcodeTips/Check%20Spelling.png)

当开启之后，我们在代码编辑过程中出现错误单词后，`Xcode`会将该单词下面加上红色波浪线，点击邮件并出现推荐的单词以及一些操作。

![Check Spelling Prompt](../../../img/iOS/Xcode/XcodeTips/Check%20Spelling%20prompt.png)

> 红色波浪线误单词为`Infoo`。

### Code Snippet

这是我们一定要利用起来的东西，良好、丰富的代码块可以有效提高我们代码的编写速度，

其中我们在保存一些代码段的时候可以需要留下一些变量等到使用时再填写上，我们可以使用`<#placeholder#>`这样的方式来留下待填充值。

我们编写的代码段保存路径为：

`~/Library/Developer/Xcode/UserData/CodeSnippets`

我们可以使用一些云同步方式对其进行同步，方便复用。

### 其他

- 可以使用 `Ctrl` + `I` 快捷建来重新调整所选代码的缩进，但其能力有限，如果你使用的是 Swift 开发语言，建议使用`SwiftFormat for Xcode`；
- 可以使用 `Command` + `+`/`-` 来调整编辑区域代码的字体大小，在代码演示时比较常用；
- 可以使用 `Command` + `option` + `[`/`]` 来向上或向下移动所选代码行；
- ...

## 导航

### `Open Quickly`

`Command` + `Shift` + `O`，该快捷键会打开一个`Open Quickly`窗口，使我们能够搜索几乎所有内容，包括文件、类型、方法、函数和属性。

![OpenQuickly](../../../img/iOS/Xcode/XcodeTips/OpenQuickly.png)

### `Reveal in Project Navigator`

`Command` + `Shift` + `J`，该快捷建会将你当前打开的文件在左侧导航定位到，方便查到该文件所在等级。

### 其他

## 模拟器

![Simulator](../../../img/iOS/Xcode/XcodeTips/Simulator.png)

模拟器`Debug`下这三个功能比较常用，其中从上到下依次：

1. 将动画变慢，可以更好看清动画的动作；
2. 检测图层混合；
3. 检测离屏渲染；


## 最后

要更加努力呀！

Let's be CoderStar!

---
title: 前端关于iOS的适配
category:
  - 前端
  - iOS
tags:
  - 前端
  - iOS
date: 2021-01-20 08:51:04
---

## Date 对象

可以通过下面四种形式创建 Date 对象；

```js
var d = new Date(); // 当前日期和时间
var d = new Date(milliseconds); // milliseconds指距离 1970 年 1 月 1 日至指定日期的毫秒数
var d = new Date(dateString);
var d = new Date(year, month, day, hours, minutes, seconds, milliseconds);
```

其中对于第三种，常用的 dateString 形式包含两种格式，一种为`yyyy-MM-dd hh:mm:ss`，另一种为`yyyy/MM/dd hh:mm:ss`。其中第一种在 iOS 不兼容，转换出来的 Date 为 NaN。

## input 标签在 disabled 情况下颜色偏淡

```css
input:disabled,
input[disabled] {
  color: red;
  -webkit-text-fill-color: red; /* 
  该属性比较重要 */
  -webkit-opacity: 1;
  opacity: 1;
}
```

## 滑动不流畅问题

```css
-webkit-overflow-scrolling: touch; /* 当手指从触摸屏上移开，会保持一段时间的滚动 */
-webkit-overflow-scrolling: auto; /* 当手指从触摸屏上移开，滚动会立即停止，这为默认值 */
```

采用第一种方式解决问题

## 点击事件 Click 延时

iOS 中的 safari，为了实现双击缩放操作，在单击 300ms 之后，如果未进行第二次点击，则执行 click 单击操作。也就是说来判断用户行为是否为双击产生的。但是，在 App 中，无论是否需要双击缩放这种行为，click 单击都会产生 300ms 延迟。

**使用 touchstart 替换 click**

不可以把所有的 click 换成 touchstart，因为在页面可以滚动情况下，滚动会触发 click 事件，在具有滚动的情况下，还是建议使用 click 处理。（事件触发顺序: touchstart, touchmove, touchend, click）

**使用 fastclick 库**

```
import FastClick from 'fastclick';
FastClick.attach(document.body, options);
```

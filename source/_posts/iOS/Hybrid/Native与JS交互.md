---
title: Native 与 JS 交互
category:
  - iOS
  - Hybrid
tags:
  - iOS
  - Hybrid
date: 2021-02-02 12:39:50
---

## 原生支持

### JS 调用 Native

#### url 拦截

一般情况下都是使用`iframe`跳转

* 连续续调用 location.href 会出现消息丢失，因为 WebView 限制了连续跳转，会过滤掉后续的请求；
* URL 会有长度限制，一旦过长就会出现信息丢失 因此，类似 WebViewJavaScriptBridge 这类库，就结合了注入 API 的形式一起使用；

#### 捕获 promt

UIWebView 不支持，WKWebView 支持，可以支持同步调用

#### 官方

UIWebView 可以使用`JSContext`；
WKWebView 因为是多线程，不存在`JSContext`，使用 MessageHandler 的方式；

## 三方库

- [DSBridge](https://github.com/wendux/DSBridge-IOS)
- [WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge)

**DSBridge**

JS 调用原生主要是利用拦截 prompt 输入框传递信息。其中同步调用是实现 promt 输入框来实现的。具体的原理可以自己去看下源码。

**WebViewJavascriptBridge**

主要原理是创建一个看不见的 iframe，然后原生部分通过拦截 url 进行解析传递信息。

**对比**

* 同步支持：WVJB 是不支持同步调用的，这意味着 js 中的 native 调用必须是异步的，如果调用过多，又会出现'回调地狱'。而对于一些简单的调用，同步调用会很方便。
* 初始化：WVJB 需要在 js 中定义一个函数，来异步获取 bridge 对象 (只有 webview 初始化成功后，bridge 才可以使用)，而 dsBridge 只需要引入一段初始化 js 后（dsbridge.js）就可以直接使用 , 而无论 webview 是否初始化成功，因为在 webview 在未初始化成功时，dsBridge 会将调用入队，等到初始化成功后再调用。
* API 管理：WVJB 的 API 管理是碎片化的，每一个 API 都需要单独注册，如果 API 较多 (如 web app)，代码耦合就会较乱。而 DSBridge 通过将 API 强制分离到一个类中, 这样一来可以解耦，二来也可以批量管理。

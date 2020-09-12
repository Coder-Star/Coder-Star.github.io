---
title: 关于WKWebView
date: 2019-10-12 14:08:29
categories: [iOS]
tags: [iOS]
---
## 1、代理

WKWebview代理主要是WKUIDelegate以及WKNavigationDelegate两部分。

### 1、WKNavigationDelegate可以分为页面跳转以及页面渲染两部分。

```swift
//页面调转部分
//发送请求前执行，决定是否跳转
optional public func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Void)
//用户认证
optional public func webView(_ webView: WKWebView, didReceive challenge: URLAuthenticationChallenge, completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void)
//收到回应，决定是否跳转
optional public func webView(_ webView: WKWebView, decidePolicyFor navigationResponse: WKNavigationResponse, decisionHandler: @escaping (WKNavigationResponsePolicy) -> Void)
//后台重定向
optional public func webView(_ webView: WKWebView, didReceiveServerRedirectForProvisionalNavigation navigation: WKNavigation!)
//失败
optional public func webView(_ webView: WKWebView, didFailProvisionalNavigation navigation: WKNavigation!, withError error: Error)

//页面渲染
//网页开始加载时调用 
optional public func webView(_ webView: WKWebView, didStartProvisionalNavigation navigation: WKNavigation!)
//网页正在加载
optional public func webView(_ webView: WKWebView, didCommit navigation: WKNavigation!)
//网页加载完成
optional public func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!)
//网页加载失败
optional public func webView(_ webView: WKWebView, didFail navigation: WKNavigation!, withError error: Error)

//关闭
optional public func webViewWebContentProcessDidTerminate(_ webView: WKWebView)
```

## 2、其他

### 1、js注入相关
  
```swift
//注入宽度自适应标签
 let js = "var oMeta = document.createElement('meta');oMeta.content = 'initial-scale=0.6,minimum-scale=0.5';oMeta.name = 'viewport';document.getElementsByTagName('head')[0].appendChild(oMeta);"
webView.evaluateJavaScript(js, completionHandler: nil)

//body滚动到顶部
webView.evaluateJavaScript("document.body.scrollTop = document.documentElement.scrollTop = 0;", completionHandler: nil)

//禁止长按出现菜单
webView.evaluateJavaScript("document.documentElement.style.webkitUserSelect='none';", completionHandler: nil)
webView.evaluateJavaScript("document.documentElement.style.webkitTouchCallout='none';", completionHandler: nil)

//放大文字
webView.evaluateJavaScript("document.getElementsByTagName('body')[0].style.webkitTextSizeAdjust= '266%'",completionHandler: nil)
```

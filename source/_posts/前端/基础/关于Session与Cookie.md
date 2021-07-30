---
title: 关于Session与Cookie
date: 2019-09-25 20:58:52
categories: 
   - 前端
tags: [web]
state: 发布
---

## 前言

有些时候技术群里面的群友会说一些涉及 Session 和 Cookie 的问题，但是通过他们的表述感觉其对两者并没有了解清楚，今天简单说明一下。

## Cookie

cookie 是存储在浏览器端的信息，以一串文本形式存在，存储的容量有限，大约为 4KB；我们可以通过 cookie 保存一些登录相关的账号密码等信息，并且我们可以人工的去编辑、阻止或者删除它，这种方式本身有着很大的风险性；
如果我们不设置 cookie 的过期时间时，cookie 信息保存在内存中，当浏览器关闭窗口时，就会自动将 cookie 删除；如果设定了过期时间，则浏览器会将 cookie 保存在硬盘中。存储在硬盘上的 cookie 可以在不同的浏览器进程间共享，比如两个浏览器窗口。而对于保存在内存里的 cookie，不同的浏览器有不同的处理方式。
cookie 在本地的存储格式为 key=value;key1=value1 有点类似于 map 的键值对，每个键值对之间用；号隔开，同时不允许键、值中出现分号（;）、逗号（,）、等号（=）以及空格；如果值中需要出现上述特殊字符，可以需要使用 escape()函数进行编码，取值时候通过 unescape()函数将取出的值进行解码。

前端持久化存储的方式除上述 Cookie 外还有以下几种形式：

- SessionStorage
- LocalStorage
- IndexDB
- WebSQL

对于 APP 来讲，也是存储有 Cookie 信息的，因为 `WKWebView`、`UIWebView` 其实本身也是一种浏览器壳子，拿 iOS 的 `WKWebView` 举个 🌰 ，使用下列代码可以获得 WkWebView 的 Cookie；

```swift
if #available(iOS 11.0, *) {
    webView.configuration.websiteDataStore.httpCookieStore.getAllCookies { cookies in
    }
}
```

### 1、新建、修改 cookie

新建 cookie，如果前面的 key 值不存在，则会新建 key 值为 username，value 值为李四的键值对；如果已存在，则会修改对应 key 的 value 值

```js
document.cookie = "username=李四"; //单个键值对
document.cookie = "username=李四;age=21"; //多个键值对，中间用；隔开
```

### 2、删除 cookie

删除 cookie 是通过设置 cookie 的过期时间为一个过去的时间来达到删除 cookie 的目的，真正的删除是由浏览器去自动删除的，其中删除时起作用的是对应 key 值，value 值可填可不填

```js
var date = new Date();
date.setTime(date.getTime() - 1); //设置一个过去时间
document.cookie = "password=;expires=" + date.toGMTString(); //设置过期时间
//document.cookie="password=123;expires="+date.toGMTString(); 如上代码效果相同
```

### 3、取 cookie 值

```js
var cookieInfo = document.cookie; // 我们可以使用split的方式将字符串cookieInfo转成数组，然后取出指定key的value值
```

## Session

session 是存储在服务器端的信息，称为会话信息。同一站点，每一个用户登录都会建立一个 session，会为每一个用户建立一个 id 作为访问的唯一标识，（比如 tomcat 容器的这个 id 叫做 Jsessionid，不同的容器名字会不同），当服务器访问人过多，服务器的压力就会很大。当关闭网站时，会话结束，session 就会失效，并不能长时间的保存；

#### 1.session 附加属性

```java
HttpSession session = request.getSession(); //获取session对象
session.setAttribute(String key,Object value);; //为session附加属性，其中value为对象类型
```

#### 2.获取 session 属性

```java
Object value=session.getAttribute(String key);
```

#### 3.session 删除

```java
session.invalidate(); //销毁session对象，所有属性删除
session.removeAttribute("currentName"); //删除某个session属性
```

在实际开发中，一般会将用户的信息封装成一个实体（属性可能包括账号、密码、权限、个人相关信息等等），然后将产生的对象放在 session，这样我们就可以根据登录用户的 session 获取这个用户的基本信息了；

## 两者联系

当后台程序需要为某个前台请求创建一个 session 时，服务器首先检查这个请求的 Cookie 值是否会有 session id，如果已包含则说明以前已经为此客户端创建过 session（一般是成功登录），服务器就按照 session id 把这个 session 检索出来使用，如果客户端请求不包含 session id，则为此客户端创建一个 session 并且生成一个与此 session 相关联的 session id，并且将该 id 放在 Response 中的 set-cookie 中，下一次客户端再请求就会自动将这个 id 放在 request header 中传给服务器，其中这个 id 一般是存在 cookie 中；

当客户端将 cookie 禁用时，我们无法将该 id 保存在 cookie 中，我们就需要对我们的请求 url 进行重写，(将 id 放在 url 后面，作为一个 get 请求的参数传递给后台)。

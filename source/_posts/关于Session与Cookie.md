---
title: 关于Session与Cookie
date: 2019-09-25 20:58:52
categories: [杂项]
tags: [web]
---
## 一、cookie

cookie是存储在浏览器端的信息，以一串文本形式存在，存储的容量有限，大约为4KB；我们可以通过cookie保存一些登录相关的账号密码等信息，并且我们可以人工的去编辑、阻止或者删除它，这种方式本身有着很大的风险性；
如果我们不设置cookie的过期时间时，cookie信息保存在内存中，当浏览器关闭窗口时，就会自动将cookie删除；如果设定了过期时间，则浏览器会将cookie保存在硬盘中。存储在硬盘上的cookie可以在不同的浏览器进程间共享，比如两个浏览器窗口。而对于保存在内存里的cookie，不同的浏览器有不同的处理方式。
cookie在本地的存储格式为  key=value;key1=value1  有点类似于map的键值对，每个键值对之间用；号隔开，同时不允许键、值中出现分号（;）、逗号（,）、等号（=）以及空格；如果值中需要出现上述特殊字符，可以需要使用escape()函数进行编码，取值时候通过unescape()函数将取出的值进行解码

### 1、使用JS操作cookie

### 1、新建、修改cookie

 新建cookie，如果前面的key值不存在，则会新建key值为username，value值为李四的键值对；如果已存在，则会修改对应key的value值

``` js
document.cookie = 'username=李四' //单个键值对 
document.cookie = 'username=李四;age=21'  //多个键值对，中间用；隔开
```

### 2、删除cookie

删除cookie是通过设置cookie的过期时间为一个过去的时间来达到删除cookie的目的，真正的删除是由浏览器去自动删除的，其中删除时起作用的是对应key值，value值可填可不填

```js
var date=new Date();
date.setTime(date.getTime()-1);//设置一个过去时间
document.cookie="password=;expires="+date.toGMTString();//设置过期时间
//document.cookie="password=123;expires="+date.toGMTString(); 如上代码效果相同
```

### 3、取cookie值

```js
var cookieInfo = document.cookie // 我们可以使用split的方式将字符串cookieInfo转成数组，然后取出指定key的value值
```

## 二、session

session是存储在服务器端的信息，称为会话信息。同一站点，每一个用户登录都会建立一个session，会为每一个用户建立一个id作为访问的唯一标识，（其中tomcat容器的这个id叫做Jsessionid，不同的容器名字会不同），当服务器访问人过多，服务器的压力就会很大。当关闭网站时，会话结束，session就会失效，并不能长时间的保存；

### 1、操作session代码

#### 1.session附加属性

```js
HttpSession session = request.getSession(); //获取session对象
session.setAttribute(String key,Object value);; //为session附加属性，其中value为对象类型
```

#### 2.获取session属性

```js
Object value=session.getAttribute(String key);
```

#### 3.session删除

```js
session.invalidate(); //销毁session对象，所有属性删除
session.removeAttribute("currentName"); //删除某个session属性
```

在实际开发中，一般会将用户的信息封装成一个实体（属性可能包括账号、密码、权限、个人相关信息等等），然后将产生的对象放在session，这样我们就可以根据登录用户的session获取这个用户的基本信息了；

###  三、两者联系

当后台程序需要为某个前台请求创建一个session时，服务器首先检查这个请求的Cookie值是否会有session id，如果已包含则说明以前已经为此客户端创建过session（一般是成功登录），服务器就按照session id把这个session检索出来使用，如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，并且将该id放在Response中的set-cookie中，下一次客户端再请求就会将这个id放在request header中传给服务器，其中这个id一般是存在cookie中；
当客户端将cookie禁用时，我们无法将该id保存在cookie中，我们就需要对我们的请求url进行重写，(将id放在url后面，作为一个get请求的参数传递给后台)，关键代码如下

```js
response.encodeURL("http://www.laozizhu.com");
// 重写后会在url后面加上 ;JSESSIONID=1234 （假如session id是1234）
```

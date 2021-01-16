---
title: Flutter-Native通信
category:
  - Flutter
tags:
  - Flutter
date: 2020-11-21 22:38:25
---
### Flutter与Native通信方式

* BasicMessageChannel：用于传递字符串和半结构化的信息,这个用的比较少
* MethodChannel：用于传递方法调用（method invocation）通常用来调用native中某个方法
* EventChannel: 用于数据流（event streams）的通信。有监听功能，比如电量变化之后直接推送数据给flutter端

Channel提供标准的消息解码器来为我们在发送及接收数据自动进行序列化及序列化；

支持的数据类型
![支持的数据类型](../../img/Flutter/Flutter-Native通信/MethodChannel数据类型)

codec，MessageCodecx协议
* JSONMessageCodec  
  JSONMessageCodec用于基础数据与二进制数据之间的编解码，其支持基础数据类型以及列表、字典。其在iOS端使用了NSJSONSerialization作为序列化的工具，而在Android端则使用了其自定义的JSONUtil与StringCodec作为序列化工具
* BinaryCodec  
  BinaryCodec是最为简单的一种Codec，因为其返回值类型和入参的类型相同，均为二进制格式（Android中为ByteBuffer，iOS中为NSData）。实际上，BinaryCodec在编解码过程中什么都没做，只是原封不动将二进制数据消息返回而已。或许你会因此觉得BinaryCodec没有意义，但是在某些情况下它非常有用，比如使用BinaryCodec可以使传递内存数据块时在编解码阶段免于内存拷贝。
* StringCodec  
  StringCodec用于字符串与二进制数据之间的编解码，其编码格式为UTF-8。
* StandardMessageCodec  
  是BasicMessageChannel的默认编解码器，其支持基础数据类型、二进制数据、列表、字典

  
### MethodChannel 




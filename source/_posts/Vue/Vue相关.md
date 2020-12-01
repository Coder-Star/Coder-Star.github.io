---
title: Vue相关
date: 2020-07-22 16:16:14
tags: [Vue]
---

### props

props是父组件给子组件传值的一种方式，其中可以包括

```Vue
props: {
    refFun: {
        type: Function,
        default: () => () => {}
    },
    refObj: {
        type: Object,
        default: () => ({})
    },
    refArr: {
        type: Array,
        default: () => { return [] }
    },
}
```

### 事件修饰符

* **.stop** 是阻止冒泡行为,不让当前元素的事件继续往外触发,如阻止点击div内部事件,触发div事件
* **.prevent** 是阻止事件本身行为,如阻止超链接的点击跳转,form表单的点击提交
* **.self** 是只有是自己触发的自己才会执行,如果接受到内部的冒泡事件传递信号触发,会忽略掉这个信号
* **.capture** 是改变js默认的事件机制,默认是冒泡,capture功能是将冒泡改为倾听模式
* **.once** 是将事件设置为只执行一次,如 .click.prevent.once 代表只阻止事件的默认行为一次,当第二次触发的时候事件本身的行为会执行
* **.passive** 滚动事件的默认行为 (即滚动行为) 将会立即触发，而不会等待 onScroll 完成。这个 .passive 修饰符尤其能够提升移动端的性能。
  
```vue
@click.stop
```


### 其他

this.$forceUpdate() 强制刷新

**关于$nextTick的说明**

vue的双向绑定，dom更新是异步更新的，也就是说值修改后之后，对应的dom并不会同步进行更新，如果此时立即去取dom的值，取到的还是更新之前的值，$nextTick会将回调延迟到下次DOM更新之后执行。

```vue
this.$nextTick(()=>{
    // 可以获取改变后dom的信息了
})
```

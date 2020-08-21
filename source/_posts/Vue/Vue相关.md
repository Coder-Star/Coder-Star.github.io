---
title: Vue相关
date: 2020-07-22 16:16:14
tags: [Vue]
---

## 1、props

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
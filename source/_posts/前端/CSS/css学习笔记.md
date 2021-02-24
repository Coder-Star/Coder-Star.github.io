---
title: css学习笔记
category:
  - CSS
  - 学习笔记
tags:
  - CSS
date: 2021-02-22 14:46:51
---

## 修改input - placeholder 和 readonly 的样式

```css
/* WebKit browsers */
::-webkit-input-placeholder { 
    color: #999999;
}
/* Mozilla Firefox 4 to 18 */
:-moz-placeholder { 
    color: #999999;
}
/* Mozilla Firefox 19+ */
::-moz-placeholder { 
    color: #999999;
}
/* Internet Explorer 10+ */
:-ms-input-placeholder { 
    color: #999999;
}

.div-name[readonly] {
    color: #999999;
}
```
---
title: 前端错误信息站
tags: code
date: 2017-07-04
---

**nodejs/webpack项目提示Invalid Host header**

新版的webpack-dev-server出于安全考虑，默认检查hostname，如果hostname不是配置内的，将中断访问。

<!-- more -->

————

disableHostCheck: true

```js
devServer: {
 contentBase: path.resolve(__dirname, 'build'),
 host: '0.0.0.0',
 port: process.env.PORT || 8601,
 disableHostCheck: true
 }
```


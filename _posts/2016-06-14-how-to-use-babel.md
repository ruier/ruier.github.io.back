---
layout: post
title:  "babel"
categories: JavaScript
tags:  ES2015 ES6 ES5 babel 
---

* content
{:toc}



## babel-cli

```json
{
    "asi": true,
    "esversion": 2015
}
```


## 配置`.jshintrc`

若编辑器中安装了 jshint 语法检查的插件。默认对于 ES2015 的代码可能会报错或者警告，看着可能会不爽。我们可以在配置文件中将它设置为允许 ES2015 的模式。

在项目根目录下创建文件`.jshintrc`。内容如下：

```json
{
    "asi": true,
    "esversion": 2015
}
```



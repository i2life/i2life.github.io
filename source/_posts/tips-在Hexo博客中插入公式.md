---
title: 'tips:在Hexo博客中插入公式'
date: 2019-10-03 12:45:31
tags: Hexo
categories: Hexo
---

写博客的时候，有的时候会需要插入公式，基于Hexo的next主题怎么编辑公式呢？

## 安装MathJax插件
原生hexo并不支持数学公式，需要安装插件 mathJax。mathJax 是一款运行于浏览器中的开源数学符号渲染引擎，使用 mathJax 可以方便的在浏览器中嵌入数学公式

安装也比较简单：
````xml
npm install hexo-math --save
```

## 修改站点配置文件
在站点配置文件中指定要使用的公式编辑引擎MathJax：
```xml
math:
  engine: 'mathjax' # or 'katex'
  mathjax:
    # src: custom_mathjax_source
    config:
      # MathJax config
```

## 修改next主题配置文件
在next主题配置文件中打开mathJax开关，修改默认配置为：
```xml
# MathJax Support
mathjax:
  enable: true
  per_page: false
  cdn: //cdn.bootcss.com/mathjax/2.7.1/latest.js?config=TeX-AMS-MML_HTMLorMML
```

## 本地公式预览
这个需要看使用的是什么markdown编辑器，我使用的是haroopad，为了在本地可以预览公式，所以需要对haroopad进行设置：
![](https://raw.githubusercontent.com/i2life/imageBed/master/haroopad.JPG)


进行完上述配置就可以愉快的进行公式编辑了：
```shell
行内公式：$公式$
行间公式：$$公式$$
```

**本地预览**
![](https://raw.githubusercontent.com/i2life/imageBed/master/mathJax1.JPG)
**发布到网页：**
这是一个行内公式：$f(x)=ax+b$，
这是一个行间公式：
$$f(x)=x^3+1$$





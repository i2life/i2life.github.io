---
title: hexo博客备份
date: 2019-10-01 08:40:45
tags: Hexo
categories: Hexo
---

其实很早之前就开始用hexo写博客了，hexo是一个快速、简洁且高效的博客框架。Node.js带来的超快生成速度，可以让上百个页面在几秒钟内完成渲染，而且hexo对markdown的友好支持，再加上其超强的插件系统，可以让用户很方便地搭建一套自己的博客站点，然后将博客页面托管到github page，一切都很优雅很方便。

然而直到有一天，我的老联想b460挂了，一块年久失修的硬盘停止了工作，里面除了一些美丽的老师的教学视频之外倒也没什么可惜，想着旧的不去新的不来，正好可以换个新电脑……

一拍脑门才想起来，我的hexo博客似乎也好死不死的搭在这块硬盘上，这可咋整，翻了一下github上的仓库只有静态的页面文件，source文件不会自动上传到页面，也就是之前写的博客都没了，肠子都悔青了，看来之前压根没有考虑到博客备份，重新再搭一个博客，又没法接着之前的博客地址继续写，要把之前的博客内容搬过来，又没有备份文件，除非去老博客的网页把内容复制下来，调成markdown格式，再重新部署到新博客。。。人肉成本太高了。

痛定思痛，重新搭了一个博客，搬了几篇老博客的内容，对于价值不大的一些博客就让他们跟那块古老的希捷硬盘一起牺牲吧，那么对新博客一定要做好备份工作。

看网上有的说复制到网盘啊，感觉都比价麻烦，后来看到一个通过github分支备份的方法，觉得比较靠谱，也比较简单，就采取了这种方法：

**stpe1：创建分支**
创建一个新分支hexo:
```shell
git checkout -b hexo
```
可以查看当前所在分支：
```shell
git branch
```

![](https://raw.githubusercontent.com/i2life/imageBed/master/git.JPG)

**step2：将github仓库的默认分支改成hexo分支**
理论上通过step1，在github仓库上应该可以看到hexo分支了，如果没有，在github仓库上手动创建一个hexo分支即可。

![](https://raw.githubusercontent.com/i2life/imageBed/master/hexo.JPG)
**step2：将本地修改备份到hexo分支**
```shell
git add ./
git commit -m "backup"
git push -u origin hexo
```

这个时候如果提示远程仓库跟本地仓库不一致，就把远程仓库先pull一下再push。

**step3：正常的hexo deploy**
上面将默认分支设置成了hexo，那hexo的操作会受到影响吗，其实不会，因为我们在blog的config文件中配置的deploy操作关联的是远程的master分支：
![](https://raw.githubusercontent.com/i2life/imageBed/master/config.JPG)
所以不需要进行分支切换，直接进行hexo操作，就会对master分支生效。

至此正常的博客备份，博客hexo部署都ok了。

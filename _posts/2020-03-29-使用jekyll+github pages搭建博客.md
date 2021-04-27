---
layout:     post
title:      使用jekyll+github pages搭建博客
subtitle:   
date:       2020-03-29
author:     Chris
header-img: img/post-bg-liuyifei.jpg
catalog: true
tags:
    - jekyll
    - github pages
---

# 1 背景
* 之前就已经使用jekyll+github pages将博客搭建好了，使用的主题是https://github.com/qiubaiying/qiubaiying.github.io。但是，使用过程中有一个bug：文章页面下滑或者上滑，滚动条会卡住，导致页面卡住。一直想修复这个问题。
* 想换一个页面主题，觉得这个主题不够好看。


# 2 调研
* 更换主题   
    更换主题可以解决主题不够好看的问题，也可以解决页面卡住的问题。然而，在更换的过程中发现了如下问题：   
    * 很多主题都不会自动生成文章目录，这点我无法接受
    * 博客主题好看的不多
* 继续使用当前主题   
    看了很多主题，花了大概3-4小时，才发现自己目前使用的主题是最好看的，支持目录。我只要解决滚动卡住的问题就可以了。根据issue里面的[解决办法](https://github.com/qiubaiying/qiubaiying.github.io/issues/52)，改了两行代码就解决了。

# 3 总结体会
* 尝试了不同的主题，才发现现在的主题最好。
* 修复了滚动的bug之后，感觉好爽快。

# 4 参考
[Jekyll Themes](http://jekyllthemes.org/)   
[Github+Jekyll 搭建个人网站详细教程](https://www.jianshu.com/p/9f71e260925d)
---
layout: post
title:  "搬家到新博客!"
date:   2016-02-03 23:27:01 +0800
categories: tech
tags:
- git
---

今天开始我将把博客搬家到这里，哈哈。 

先写些工作中常用的git命令吧，正好我也经常记不住...

今天闲来git log专题。[官方文档]

# git log

- git log -2

    显示最近的两条

- git log --stat

    显示修改的文件

- git log --pretty=oneline

    只显示commitid和memo，还可以用short,full,fuller

- git log --pretty=format:"%h %cd %s %an"
    
    按指定格式显示

- git log --graph

    按图形模式显示

- git log --since=2.weeks
    
    指定时间段，也可以--since="2016-02-01"

- git log --author=<pattern>

    指定作者：git log --author="zhao*"


[官方文档]: https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History

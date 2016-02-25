---
layout: post
title:  "实战Git之git reset"
date:   2016-02-25 11:25:01 +0800
categories: tech
tags:
- git
---

今天再来git reset专题。[官方文档]

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


[官方文档]: https://git-scm.com/docs/git-reset

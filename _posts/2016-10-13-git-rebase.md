---
layout:     post
title:      "Git 常用命令(一)"
subtitle:   "Git 的常用命令"
date:       2016-10-13
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - git
---

> 本篇将介绍 git 的常用命令
#git常用命令

### rebase 命令
在日常开发使用 git 时，不注意提交规范，导致 commit 提交很乱，特别是在团队开发中，这时我们可以使用 git rebase 命令，来使我们的提交纪录更为整洁。


我们先来看看git-rebase的命令格式：
git rebase [-i | --interactive] [options] [--onto ]  []
git rebase [-i | --interactive] [options] –onto   –root []
git rebase –continue | –skip | –abort


git pull --rebase origin master

可以先使用 git log 命令查看自己有几次提交记录。

git log

git rebase -i HEAD~次数

要合并的最近几次commit  例如：git rebase -i HEAD~3


然后把第一个
pick 下面的 pick 都换成 s

git branch --set-upstream-to=origin/远程分支 本地分支

使本地分支和远程分支相关联

git branch --set-upstream-to=origin/xx xx

最后提交
git push -f



### 分支重命名

本地分支重命名

git branch -m oldName newName



###推送本地分支到远端

将本地分支推送到远端

git push origin <分支名称>




---
layout:     post
title:      "Git 常用命令(二)"
subtitle:   "Git 的常用命令"
date:       2016-10-18
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - git
---

> 本篇将介绍 git 的常用命令


# git checkout

##### git 切换分支

git checkout branch_name

例如： 本地有个分支hello

git checkout hello


##### git 创建分支并切换到新建的分支

git checkout -b branch_name

该命令的 -b 就是 branch 的意思

例如: 创建 hello 分支并切换到 hello 分支

git checkout -b hello

# git branch

##### git 创建本地分支

git branch branch_name

例如： git branch hello

##### git 查看本地分支

git branch

git 查看远程分支

git branch -a



##### git 删除本地分支

git branch -d branch_name

例如：删除本地的 hello 分支

git branch -d hello



##### git 删除远程分支

git branch -d -r branch_name

例如：删除远程分支 hello

git branch -d -r hello




##### git 重命名分支

git branch -m oldbranch newbranch

例如：将分支 hello 分支重命名为 test

git branch -m hello test


##### git 强制重命名

git branch -M oldbranch newbranch

如果需要重命名的名字与其他分支重复了，但我们想强行修改分支名，那么我们需要使用 -M 命令，但并不推荐使用该命令。


例如：将本地分支 hello 重命名为 test

git branch -M hello test












































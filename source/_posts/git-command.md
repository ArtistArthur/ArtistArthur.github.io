---
title: git command
top: false
cover: false
toc: true
mathjax: true
date: 2020-07-09 11:16:33
password:
summary:
tags:
categories:
---

## git常用命令以及与github的结合使用

### git中的四个区域
<!--more-->
* workspace：写和修改代码的地方
* index/stage：暂存区，git add [file name]把修改从workspace提交到暂存区
* local repository：本地仓库，git commit -m [message] 把修改从暂存区提交到本地仓库
* remote ：远程仓库，git push 把修改从本地仓库推送到远程仓库
![git区域关系](https://i.bmp.ovh/imgs/2019/07/c49a713c776e6967.png)

* git pull 把远程仓库最新的更改拉取到本地
* git add . 把所有本地改动提交到暂存区
* git commit -m [message] 把修改从暂存区提交到本地仓库
* git push 把修改从本地仓库推送到远程仓库
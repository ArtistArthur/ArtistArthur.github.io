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
* workspace：写和修改代码的地方
* index/stage：暂存区，git add [file name]把修改从workspace提交到暂存区
* local repository：本地仓库，git commit -m [message] 把修改从暂存区提交到本地仓库
* remote ：远程仓库，git push 把修改从本地仓库推送到远程仓库
![git区域关系](https://i.bmp.ovh/imgs/2019/07/c49a713c776e6967.png)
<!--more-->

* git pull 把远程仓库最新的更改拉取到本地
* git add . 把所有本地改动提交到暂存区
* git commit -m [message] 把修改从暂存区提交到本地仓库
* git push 把修改从本地仓库推送到远程仓库
* git里有三个东西:origin master origin/master
  * master是一个本地分支
  * origin/master是远程分支,是名为origin的远程分支,里面有个本地master的副本,也叫master
  * `git push origin master`是`git push  <REMOTENAME> <BRANCHNAME> `的运用,指把本地内容推送到origin(远端)的master分支下,命令里的master指远端的master分支.
  * 名字不一定一定是origin也可以是其他名字:origin1 github gitlab等,master也一样.
* `git fetch origin master`指从远端origin拉取master分支,此时会会产生一个本地节点副本origin/master,`git merge origin/master`可以合并这个远端到本地,`git push origin master`可以把合并后的本地内容推送到远端.
* `git push --set-upstream origin <branchname>`,在远端建立分支并把本地分支推送过去.
    
github从本地推一个新的仓库到远端:  
1.在远端新建一个仓库,不要生成readme  
2.在本地生成rsa公钥和私钥,把公钥手动添加到远端仓库设置里(如果已经设置了可以跳过这一步).    
``` 
ssh是一种协议,有不同的具体实现.  
其中一种实现非对称密钥,在client本地生成rsa公钥和私钥,手动把公钥保存在服务器:这样可以让服务器验证client的身份,具体方法为,服务器产生一个随机数,通过公钥加密,发送给client,client通过私钥解密随机数,然后和sessionkey一起通过md5加密,发送给服务器,服务器也通过md5加密原先的随机数和sessionkey,验证client发来的两个md5签名是否相同.  
client的rsa密钥保存在`~/.ssh/`下,`id_rsa`保存私钥,`id_rsa.pub`保存公钥,`know_hosts`保存已验证的服务器(实际上client也需要验证服务器的身份以防止中间人攻击).  
`ssh-keygen -t rsa -C "your@email"`命令生成新的rsa公钥私钥,期间有个passphrase,即设置一个密码,是可选的,可以enter直接跳过.      
如果忘记密码了,只有重新生成了.    
```
3.在本地仓库`git remote add origin git@github.com:name/reponame.git`添加远程索引.如果显示错误:fatal: remote origin already exists.则说明本地仓库已经设置了远端origin的地址,使用`git remove remote origin`删除原来的设置.
4.`git push -u origin master`把本地master推送到远端.    

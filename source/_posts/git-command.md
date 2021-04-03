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
1. 在远端新建一个仓库,不要生成readme  
2. 在本地生成rsa公钥和私钥,把公钥手动添加到远端仓库设置里(如果已经设置了可以跳过这一步).    
``` 
ssh是一种协议,有不同的具体实现.  
其中一种实现非对称密钥,在client本地生成rsa公钥和私钥,手动把公钥保存在服务器:这样可以让服务器验证client的身份,具体方法为,服务器产生一个随机数,通过公钥加密,发送给client,client通过私钥解密随机数,然后和sessionkey一起通过md5加密,发送给服务器,服务器也通过md5加密原先的随机数和sessionkey,验证client发来的两个md5签名是否相同.  
client的rsa密钥保存在`~/.ssh/`下,`id_rsa`保存私钥,`id_rsa.pub`保存公钥,`know_hosts`保存已验证的服务器(实际上client也需要验证服务器的身份以防止中间人攻击).  
`ssh-keygen -t rsa -C "your@email"`命令生成新的rsa公钥私钥,期间有个passphrase,即设置一个密码,是可选的,可以enter直接跳过.      
如果忘记密码了,只有重新生成了.    
```

3. 在本地仓库`git remote add origin git@github.com:name/reponame.git`添加远程索引.如果显示错误:fatal: remote origin already exists.则说明本地仓库已经设置了远端origin的地址,使用`git remove remote origin`删除原来的设置.   
4. `git push -u origin master`把本地master推送到远端.    
5. 打标签
## git的分支
git中有三种特殊的对象：`blob`,`tree`,`commit object`。他们之间的关系很密切：
* `blob`对象是文件的快照，是最根本的数据
* `tree`对象是一个树结构，记录着目录结构和`blob`对象索引
* `commit object`包含一个指向暂存内容快照的指针（即指向一个`tree`对象的指针）。但不仅仅是这样，该提交对象还包含了作者的姓名和邮箱、提交时输入的信息以及指向它的父对象的指针。

git的分支，其实本质上仅仅是指向提交对象的可变指针。Git的默认分支名字是master。在多次提交操作之后，你其实已经有一个指向最后那个提交对象的 master 分支。它会在每次的提交操作中自动向
前移动。  
git 的 “master” 分支并不是一个特殊分支。它就跟其它分支完全没有区别。之所以几乎每一个仓库都
有 master 分支，是因为 git init 命令默认创建它，并且大多数人都懒得去改动它。   

### git 分支的创建
git分支的创建只是为你创建了一个可以移动的新的指针，比如创建一个test分支：`git branch test`  
这会在当前所在的提交对象上创建一个指针。  
在不同的分支上提交，会产生分叉
![在两个分支上分别提交会产生分叉](https://i.bmp.ovh/imgs/2021/03/a5ff0c16c7d518d7.png)  
你可以简单地使用`git log`命令查看分叉历史。运行`git log --oneline --decorate --graph --all` ，它会输出你的提交历史、各个分支的指向以及项目的分支分叉情况。  

### 分支切换
使用`git checkout test`切换到test分支上（当创建了新分支时，head并不会自动指向新分支）  
注意：支切换会改变你工作目录中的文件在切换分支时，一定要注意你工作目录里的文件会被改变。如果是切换到一个较旧的分支，你的工作目录会恢复到该分支最后一次提交时的样子。如果 Git 不能干净利落地完成这个任务，它将禁止切换分支。这就是为什么经常会报错。  
`git checkout -b test2`是两个命令的组合：`git branch test2` `git checkout test2`即这个命令创建了一个分支，并切换到这个分支。     
### 分支合并
`git merge branchname`把branchname分支合并到当前head指向的分支。  
如果将要合并的分支（branchname）是当前分支的直接下游，则这种合并叫fast-forward，意思是这个合并只是单纯的把当前分支的指针往前移动，到branchname一样的地方。  
合并之后可以把临时分支删除了`git branch -d tmpbranch`,只会删除分支指针，并不会删除blob对象   
三方合并：两个分支合并到同一个分支（把branchname合并到当前分支），如果他们是分叉的（一个不是另一个的祖先），那么git会找到最优的祖先，进行一个三方合并（把祖先作为参照，然后把改动的地方合并进来），合并完成后，git会自动产生一个提交对象，并把两个分支都向前移动一位指向它。    
如果两个分支相对于祖先来说，都在同一个文件的同一个位置进行了修改，git就不能自动合并，会产生冲突，此时应该手动解决冲突。  

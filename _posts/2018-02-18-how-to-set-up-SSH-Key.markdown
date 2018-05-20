---
layout:       post
title:        "电脑公钥未添加至GitHub导致本地操作失败的解决方法"
subtitle:     "--踩坑"
date:         2018-02-18
author:       "Chou"
header-img:   "img/post-bg-setup-SSH-Key.jpg"
catalog:      false
multilingual: false
tags:
    - 前端开发
    - GitHub
---

## GitHub中SSH Key的配置方法

​	由于最近换了电脑，在本地clone远程仓库的时候出现了如下的错误信息：



> nomacbook-puro:chioken zhangjian$ git clone git@github.com:chio-ken/chio-ken.GitHub.io.git
>
> Cloning into 'chio-ken.GitHub.io'...
>
> git@github.com: Permission denied (publickey).
>
> fatal: Could not read from remote repository.
>
> Please make sure you have the correct access rights and the repository exsits.  

​	简单看一下上面的错误信息，clone失败的原因可能有两个：远程仓库不存在或者没有权限访问。由于我们确定仓库是存在的，那就是权限的问题了。这时想到GitHub绑定的SSH Key是之前在另一台电脑设置的，这样就有解决的思路了。重新生成本电脑的秘钥，再将其添加到GitHub上即可。

​	所以把大象放冰箱里拢共分3步：

### 第一步

​	**把冰箱门打开**——在git中配置用户名和邮箱地址，如下：

> git config —global user.name "yourname"
>
> git config —global user.email "youremail"

​	上边的"yourname"和"youremail"分为对应GitHub的用户名和注册时绑定的邮箱地址。

### 第二步

​	**把大象放冰箱里**——生成SSH Key：

​	在终端输入以下命令：

> ssh-keygen -t rsa -C "youremail"

​	按3次回车，最后两次是设定密码，按回车代表密码设为空。

> Enter passphrase (empty for no passphrase):
>
> Enter the same passphrase again:

​	之后会显示类似以下的信息：

>Your identification has been saved in ……/.ssh/id_rsa.
>
>your public key has been saved in ……/.ssh/id_rsa.pub.

​	**注意**：在输入以上命令时，如果你的电脑之前存在SSH Key，会提示你是否覆盖原来的SSH Key（终端提示 yes/no）

​	现在得到了两个文件：id_rsa和id_rsa.pub

​	打开id_rsa.pub，里面的公钥就是我们要添加到GitHub上的秘钥，打开该文件的指令如下：

> ~ zhangjian$ cd ~/.ssh
>
> .ssh zhangjian$ ls
>
> Id_rsa		id_rsa.hub		known_hosts
>
> .ssh zhangjian$ vim id_rsa.pub

### 第三步 

​	**把冰箱门关上**——在GitHub中添加私钥

​	按如下的顺序进入个人设置Settings—SSH and GPG keys-new SSH key 添加即可。

​	到这为止整个设置就完成了，可以在本地对GitHub进行操作了。










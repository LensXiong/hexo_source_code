---
layout: post
title: Git命令-1
date: 2016-10-21 08:32:24.000000000 +09:00
---
---
##1 Git SSH Key 生成步骤
```
1.设置Git的user name和email：
$ git config --global user.name "wangxiong"
$ git config --global user.email "363127224@qq.com"
$ git config --list //查看配置信息

2.生成密钥：
$ ssh-keygen -t rsa -C "363127224@qq.com"
按3个回车，密码为空。
最后得到了两个文件：id_rsa和id_rsa.pub

3.把id_rsa.pub上传至git服务器
/Users/wangxiongpaymew/.ssh/id_rsa //私钥位置
/Users/wangxiongpaymew/.ssh/id_rsa.pub //公钥位置

3.添加密钥到ssh：ssh-add 文件名
需要之前输入密码。
4.在github上添加ssh密钥，这要添加的是“id_rsa.pub”里面的公钥。
打开https://github.com/ ，登陆363127224@qq.com，然后添加ssh。

5.测试：ssh git@github.com
The authenticity of host ‘github.com (207.97.227.239)’ can’t be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added ‘github.com,207.97.227.239′ (RSA) to the list of known hosts.
ERROR: Hi tekkub! You’ve successfully authenticated, but GitHub does not provide shell access
Connection to github.com closed.
```

##2 使用Github
```
1.获取源码：
$ git clone git@github.com:billyanyteen/github-services.git
2.这样你的机器上就有一个repo了。
3.git于svn所不同的是git是分布式的，没有服务器概念。所有的人的机器上都有一个repo，每次提交都是给自己机器的repo
仓库初始化：
git init
生成快照并存入项目索引：
git add
文件,还有git rm,git mv等等…
项目索引提交：
git commit
4.协作编程：
将本地repo于远程的origin的repo合并，
推送本地更新到远程：
git push origin master
更新远程更新到本地：
git pull origin master
补充：
添加远端repo：
$ git remote add upstream git://github.com/pjhyett/github-services.git
重命名远端repo：
$ git://github.com/pjhyett/github-services.git为“upstream”
```

## 3、Git命令
```
1、$ mkdir learngit //创建文件夹
2、$ pwd //显示当前目录
3、$ git init // 初始化Git
4、$ ls -ah //显示隐藏的目录
5、$ git add readme.txt //把文件添加到仓库
6、$ git commit -m "wrote a readme file" //把文件提交到仓库
7、$ git status //工作区的状态
8、$ git diff //查看修改内容
9、$ git log //查看提交历史
10、$ git log --pretty=oneline //--pretty=oneline 单行查看
11、$ git reflog //查看命令历史
12、$ git reset --hard commit_id //回退到版本
13、HEAD当前版本 HEAD^上一个版本 HEAD^^上上一个版本 HEAD~100往上100个版本
14、工作区和版本库(.git)
版本库中add后有为stage（或者叫index）的暂存区，commit后会创建第一个分支master，以及指向master的一个指针叫HEAD
git add //把文件修改添加到暂存区
git commit //把暂存区的所有内容提交到当前分支
简单讲：需要提交的文件修改通通放到暂存区，然后，一次性提交暂存区的所有修改
15、Git跟踪并管理的是修改，而非文件
16、$ git diff HEAD -- readme.txt //查看工作区和版本库里面最新版本的区别
17、git如何跟踪修改的 //每次修改，如果不add到暂存区，那就不会加入到commit中
18、$ git checkout -- readme.txt //丢弃工作区
19、$ git reset HEAD readme.txt //丢弃暂存区
20、$ rm test.txt //删除文件(可以从版本库中checkout)
21、$ git rm test.txt //从版本库中删除文件
22、$ git checkout -- test.txt
23、$ git remote add origin https://github.com/LensXiong/learngit.git //关联一个远程库
24、$ git push -u origin master //首次推送
25、$ git push origin master //非首次推送
26、$ git clone git@github.com:michaelliao/gitskills.git//远程克隆
```

## 4、相关命令
① 撤销误操作
```
git status -s  -s表示short 第一列M表示staging 第二列M表示working
git add --file 工作区到暂存区 (working->staging)
git commit -m '' 暂存区到历史区(staging->histroy)
git commit -am 工作区到历史区(working->staging)
git checkout --file
从暂存区到工作区(staing->working)
git reset --file 从历史区到暂存区（histroy->staging）
git checkout HEAD --file 从历史区到工作区(histroy->working)
```
② 移除及重命名文件
```
git rm file
git rm --cached file
git mv oldfile newfile 文件重命名

```
③ 暂存工作区
```
git stach 把当前的改动压入一个栈
git stach list 显示这个栈的list
git  stach pop
```
④ 添加到暂存区的区别
```
git add -A   保存所有的修改
git add .    保存新的添加和修改，但是不包括删除
git add -u   保存修改和删除，但是不包括新建文件。
```

⑤ 创建分支

```
创建分支： $ git branch feature/xiong-test
切换分支： $ git checkout feature/xiong-test
创建并切换分支： $ git checkout -b feature/xiong-test
推送本地更新到远程：git push origin develop
更新远程更新到本地：git pull origin develop
删除dev-xiong分支之前需要切换到别的分支develop上：git checkout develop
删除分支： $ git branch -d dev-xiong
强制删除分支： $ git branch -D dev-xiong
回退到某个版本：git reset --hard HEAD
更新版本：git pull origin develop
```

⑥ [远程操作](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)
![http://image.beekka.com/blog/2014/bg2014061202.jpg](http://image.beekka.com/blog/2014/bg2014061202.jpg)

```
git clone <版本库的网址> <本地目录名>:
克隆后重命名目录名：git clone https://github.com/LensXiong/Yii_advanced.git yii2.0
克隆后重命名远程主机(默认origin):git clone -o Yii https://github.com/LensXiong/Yii_advanced.git yii2.0
查看远程主机的网址:git remote -v
添加远程主机：git remote add <主机名> <网址>
远程主机改名:git remote rename <原主机名> <新主机名>
取回分支更新：git fetch <远程主机名> <分支名>
git fetch origin master
查看远程分支：git branch -r
查看所有分支：git branch -a
本地分支上合并远程分支(1)：git merge origin/master
本地分支上合并远程分支(2)：git rebase origin/master
取回远程主机某个分支的更新，再与本地的指定分支合并:
git pull <远程主机名> <远程分支名>:<本地分支名>
git pull origin next:master
如果远程分支是与当前分支合并，则冒号后面的部分可以省略:git pull origin next
git pull origin next 等同于
① git fetch origin  //先做git fetch
② git merge origin/next //再做git merge
远程推送：git push <远程主机名> <本地分支名>:<远程分支名>
指定默认主机的推送：git push -u origin master
将本地的所有分支都推送到远程主机：git push --all origin
```
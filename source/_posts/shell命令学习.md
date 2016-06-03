---
title: shell命令学习
date: 2016-05-03 14:49:51
tags: shell
---

提交Jenkins的时候，会用到部分命令：

```
df -h

Filesystem      Size   Used  Avail Capacity  iused   ifree %iused  Mounted on
/dev/disk1     233Gi  199Gi   33Gi    86% 52321616 8659602   86%   /
devfs          187Ki  187Ki    0Bi   100%      648       0  100%   /dev
map -hosts       0Bi    0Bi    0Bi   100%        0       0  100%   /net
map auto_home    0Bi    0Bi    0Bi   100%        0       0  100%   /home
```


- ***df 命令：***
linux中df命令的功能是用来检查linux服务器的文件系统的磁盘空间占用情况。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

- ***date命令***
修改linux的时间可以使用date指令 

```
date
Tue May  3 14:53:22 CST 2016
➜  _posts git:(master) ✗ date '+Current time: %H:%M:%S'
Current time: 14:53:24 //可见这里会自定义输出date的格式

修改日期： 
时间设定成2009年5月10日的命令如下：
date -s 05/10/2009 

修改时间： 
将系统时间设定成上午10点18分0秒的命令如下。 
date -s 10:18:00 

```

- **git命令****

git branch newBranch   创建新分支 git br 

git checkout newBranch	切换到新分支 git co

合并： git checkout -b newBranch
简写： git co -b newBranch

git br nnew [commit] 以某次commit创建分支

git checkout -B <branch>

这个命令，可以强制创建新的分支，为什么加-B呢？如果当前仓库中，已经存在一个跟你新建分支同名的分支，那么使用普通的git checkout -b <branch>这个命令，是会报错的，且同名分支无法创建。如果使用-B参数，那么就可以强制创建新的分支，并会覆盖掉原来的分支。


---
title: iOS长连接的实现--python搭建服务及端口
date: 2016-09-22 09:57:15
tags: 
-iOS
---
###端口学习
80端口，主要是提供http服务用的，一般用户的设备上是不会被占用，服务端开启80端口，监听客户端的连接。

通过命令

	lsof -i tcp:80
可以有如下打印：

<!--more-->

```
COMMAND   PID    FD   TYPE             DEVICE SIZE/OFF NODE NAME
Google    382    92u  IPv4 0x2daafbe57dd15977      0t0  TCP 172.22.32.145:59650->221.228.218.142:http (CLOSED)
com.apple 414    25u  IPv4 0x2daafbe58dbe5d57      0t0  TCP 172.22.32.145:60738->42.121.252.58:http (ESTABLISHED)
com.apple 414    26u  IPv4 0x2daafbe58dbe5d57      0t0  TCP 172.22.32.145:60738->42.121.252.58:http (ESTABLISHED)
SourceTre 380  	 228u  IPv4 0x2daafbe589512d57      0t0  TCP 172.22.32.145:60764->131.103.28.14:http (ESTABLISHED)
SourceTre 380    35u  IPv4 0x2daafbe589512d57      0t0  TCP 172.22.32.145:60764->131.103.28.14:http (ESTABLISHED)
Google    382    92u  IPv4 0x2daafbe57dd15977      0t0  TCP 172.22.32.145:59650->221.228.218.142:http (CLOSED)
com.apple 414    22u  IPv4 0x2daafbe5895e8787      0t0  TCP 172.22.32.145:60772->101.226.196.38:http (ESTABLISHED)
com.apple 414    24u  IPv4 0x2daafbe5895e8787      0t0  TCP 172.22.32.145:60772->101.226.196.38:http (ESTABLISHED)
com.apple 414    33u  IPv4 0x2daafbe588e8e977      0t0  TCP 172.22.32.145:60782->103.37.152.63:http (CLOSE_WAIT)
com.apple 414    34u  IPv4 0x2daafbe588e8e977      0t0  TCP 172.22.32.145:60782->103.37.152.63:http (CLOSE_WAIT)
```
- 可以看到自己的源地址都是开启一些临时的端口：比如 59650， 60738等，不同的应用会创建不一样的端口，然后目的端口都是http默认的80端口，这里没有写出来。

- 同一个设备，一个端口只能被一个应用使用。

- 那什么情况下自己的电脑的80端口会被占用呢？
一般是在自己的机器上安装IIS,NGINX或者APACHE，才会冲突

windows中经常会被进程为inetinfo.exe占用。（稍后解释inetinfo.exe进程）如果你现在直接结束掉这个进程，无论如何inetinfo.exe都会自动重新运行，只是这个时候的PID就改变了。所以这样不能完全的释放80端口。
遇到这种情况，先分析是否是IIS，因为我之前有做网站，需要安装IIS，并且创建了一个站点，在控制面板-管理工具-internet 信息服务-网站下面可以看到这个站点，只需要把这个站点停止掉就可以了，然后你再到开始-运行-输入cmd（回车）-打开命令提示符——netstat -ano，可以看到已经没有80端口，这个时候在安装软件，就一切顺利了。

> 进程PID是可以变化的，就是说不同时间运行同一个程序，它的PID号就不同。不同计算机同一个进程的PID号多数情况也是不同的。因此，在结束inetinfo.exe之前的PID是一个三位数，重新启动后，它的PID可能为四位数了。这点不影响问题的解决，我只是好奇，便拿出来说罢了。

用python写一个简答的服务端程序，可以参考：
[Networking Tutorial for iOS: How To Create A Socket Based iPhone App and Server](https://www.raywenderlich.com/3932/networking-tutorial-for-ios-how-to-create-a-socket-based-iphone-app-and-server)
它里面主要是创建了一个80端口的监听，但是我自己的电脑实际上80端口还是被占用的，执行命令

	sudo lsof -i:80 | grep LISTEN
	输出
	httpd     66226      root    5u  IPv6 0x2daafbe58b3cc967      0t0  TCP *:http (LISTEN)
	httpd     66230      _www    5u  IPv6 0x2daafbe58b3cc967      0t0  TCP *:http (LISTEN)

这是因为80端口被apache给占用了，默认是开机启动的。所以可以执行

	sudo apachectl stop
	sudo python server.py

之后就可以看到了 Iphone Chat server started，后续可以把客户端给搭建起来。
> [Python twisted reactor - address already in use](http://stackoverflow.com/questions/14640711/python-twisted-reactor-address-already-in-use)
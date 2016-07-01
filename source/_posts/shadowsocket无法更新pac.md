---
title: shadowsocks无法更新pac
date: 2016-07-01 13:06:05
tags: shadowsocks
---

这两天上网发现facebook竟然不能登录，于是首先怀疑了是不是ss账号的问题，换了几个服务器，自己的电脑还是不行，但是其他人的都可以，于是就想是不是gfwlist需要更新先，然后就操作**·Update pac from GFWList·**，但是发现竟然还是request error 404. 之前自己也遇到过这样的事情，但是读没有怎么在意，今天因为要登陆facebook，所以才决定解决下的。

<!--more-->

google一下：[不能更新 PAC 文件](https://github.com/shadowsocks/shadowsocks-iOS/issues/212)发现已经在shadowsocks的issues中提交了问题，下面分析了具体的原因，自己一路看下来，决定使用

	sudo easy_install pip  //已经安装则不需要了
	sudo pip install gfwlist2pac
	sh update_gfwlist.sh
	
但是当打开连接[https://gist.github.com/VincentSit/b5b112d273513f153caf23a9da112b3a](https://gist.github.com/VincentSit/b5b112d273513f153caf23a9da112b3a)取源码的时候，发现这个都打不开，真是无语（我竟然忍受了这么久。。。）。于是在github中搜索，发现还是有蛮多好心人把这个存下来的，然后自己就顺便copy了下来，代码如下:

```
#!/bin/bash
# update_gfwlist.sh
# Author : VincentSit
# Copyright (c) http://xuexuefeng.com
#
# Example usage
#
# ./whatever-you-name-this.sh
#
# Task Scheduling (Optional)
#
#	crontab -e
#
# add:
# 30 9 * * * sh /path/whatever-you-name-this.sh
#
# Now it will update the PAC at 9:30 every day.
#
# Remember to chmod +x the script.


GFWLIST="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"
PROXY="127.0.0.1:1080"
USER_RULE_NAME="user-rule.txt"

check_module_installed()
{
	pip list | grep gfwlist2pac &> /dev/null

	if [ $? -eq 1 ]; then
		echo "Installing gfwlist2pac."

		pip install gfwlist2pac
	fi
}

update_gfwlist()
{
	echo "Downloading gfwlist."

	curl -s "$GFWLIST" --fail --socks5-hostname "$PROXY" --output /tmp/gfwlist.txt

	if [[ $? -ne 0 ]]; then
		echo "abort due to error occurred."
    exit 1
	fi

	cd ~/.ShadowsocksX || exit 1

	if [ -f "gfwlist.js" ]; then
		mv gfwlist.js ~/.Trash
	fi

	if [ ! -f $USER_RULE_NAME ]; then
		touch $USER_RULE_NAME
	fi

	/usr/local/bin/gfwlist2pac \
    --input /tmp/gfwlist.txt \
    --file gfwlist.js \
    --proxy "SOCKS5 $PROXY; SOCKS $PROXY; DIRECT" \
    --user-rule $USER_RULE_NAME \
    --precise

  rm -f /tmp/gfwlist.txt

  echo "Updated."
}

check_module_installed
update_gfwlist
```

以后需要更新rule的时候，就自己执行以下就可以了。
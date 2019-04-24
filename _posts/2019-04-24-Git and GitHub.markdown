---
layout: post
title:  "Git and GitHub"
date:   2019-04-24 16:05:00 +0800
categories: linux
---
所有操作默认非root用户，本文中用`lz`用户

Ubuntu安装git:
```shell
$ sudo apt install git
```

生成`ssh-key`:

```shell
$ ssh-keygen -t rsa -C "注释"
```

接下来会有提示：

```shell
Enter file in which to save the key (/home/lz/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again:
```

一路回车默认即可。

系统会在`~/.ssh/`下生成密钥对，`id_rsa`为私钥，`id_rsa.pub`为公钥。私钥用于表明主机身份，公钥发给对方让对方用于鉴别主机身份。

接下来登陆GitHub，鼠标右上角


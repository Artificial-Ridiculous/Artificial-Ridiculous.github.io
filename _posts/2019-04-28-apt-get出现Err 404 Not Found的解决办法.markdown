---
layout: post
title:  "apt-get出现Err 404 Not Found的解决办法"
date:   2019-04-26 15:35:00 +0800
category: linux
---
***所有操作默认非root用户，本文中用 `lz` 用户。***

每个Ubuntu版本都有生命结束周期（EOL）时间；常规的Ubuntu发行版提供18个月的支持，而LTS（长期支持）版本则长达3年（服务器版本）和5年（桌面版本）。当某个Ubuntu版本达到生命结束周期时，其仓库就不能再访问了，你也不能再从Canonical获取任何维护更新和安全补丁。

如果你所使用的Ubuntu系统已经被结束生命周期，你就会从apt-get或aptitude得到以下404错误，因为它的仓库已经被遗弃了：

```sh
$ sudo apt-get update
Err http://archive.ubuntu.com vivid/main amd64 Packages
  404  Not Found [IP: 91.189.88.149 80]
Err http://archive.ubuntu.com vivid/restricted amd64 Packages
  404  Not Found [IP: 91.189.88.149 80]
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/vivid/main/source/Sources  404  Not Found [IP: 91.189.88.149 80]

W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/vivid/restricted/source/Sources  404  Not Found [IP: 91.189.88.149 80]

E: Some index files failed to download. They have been ignored, or old ones used instead.
```

首先有一个小知识点，每个Ubuntu发行版本都有一个代号。

查看当前环境Ubuntu版本及代号：

```sh
$ lsb_release -a
sb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:  Ubuntu 18.04.2 LTS
Release:  18.04
Codename: bionic
```

Ubuntu历代版本及其代号：

版本号 | 版本代号 |  发行时间  
-|-|-
4.1|Warty Warthog(长疣的疣猪)|2004年10月20日
5.04|Hoary Hedgehog(灰白的刺猬)|2005年4月8日
5.1|Breezy Badger(活泼的獾)|2005年10月13日
6.06|Dapper Drake(整洁的公鸭)|2006年6月1日(LTS)
6.1|Edgy Eft(急躁的水蜥)|2006年10月6日
7.04|Feisty Fawn(坏脾气的小鹿)|2007年4月19日
7.1|Gutsy Gibbon(勇敢的长臂猿)|2007年10月18日
8.04|Hardy Heron(耐寒的苍鹭)|2008年4月24日(LTS)
8.1|Intrepid Ibex (勇敢的野山羊)|2008年10月30日
9.04|Jaunty Jackalope(得意洋洋的怀俄明野兔)|2009年4月23日
9.1|Karmic Koala(幸运的考拉)|2009年10月29日
10.04|Lucid Lynx(清醒的猞猁)|2010年4月29日
11.1|Oneiric Ocelot(梦幻的豹猫)|2010年10月13日
11.04|Natty Narwhal(敏捷的独角鲸)|2011年4月28日
12.04|Precise Pangolin(精准的穿山甲)|2012年的4月26日(LTS)
12.1|Quantal Quetzal(量子的绿咬鹃)|2012年的10月20日
13.04|Raring Ringtail(铆足了劲的猫熊)|2013年4月25日
13.1|Saucy Salamander(活泼的蝾螈)|2013年10月17日
14.04|Trusty Tahr (可靠的塔尔羊)|2014年4月18日 (LTS)
14.1|Utopic Unicorn(乌托邦独角兽)|2014年10月23日
15.04|Vivid Vervet (活泼的小猴)|2015年4月
15.1|Wily Werewolf (狡猾的狼人)|2015年10月
16.04|Xenial Xerus (好客的非洲地松鼠)|2016年4月 (LTS)
16.1|Yakkety Yak（牦牛）|2016年10月
17.04|Zesty Zapus(开心的跳鼠)|2017年4月
17.1|Artful Aardvark(机灵的土豚)|2017年10月
18.04|Bionic Beaver(仿生海狸)|2018年4月 (LTS)
19.04|Disco Dingo(蹦迪小狗)|2019年4月

对于使用旧版本的Ubuntu的用户，Canonical会维护 old-releases.ubuntu.com ，这是一个过期库的归档。因此，当前版本Ubuntu过期后，必须把源切换到 old-releases.ubuntu.com。  

下面的方法通过切换到旧版本的源来解决“404 Not Found”错误：

```sh
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak  # 备份原始文件
sudo sed -i -r 's/([a-z]{2}\.)?archive.ubuntu.com/old-releases.ubuntu.com/g' /etc/apt/sources.list
sudo sed -i -r 's/security.ubuntu.com/old-releases.ubuntu.com/g' /etc/apt/sources.list
sudo apt-get update
```

---

对于还在支持期内的ubuntu版本（如18.04LTS或16.04LTS），如果境外的源更新缓慢，可以将其替换为阿里云的源：

```sh
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak  # 备份原始文件
sudo sed -i -r 's/([a-z]{2}\.)?archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
sudo sed -i -r 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
sudo apt-get update
```

---
layout: post
title:  "Mysql出现Access denied for user root@localhost的解决办法"
date:   2019-04-25 16:59:00 +0800
categories: linux
---
***所有操作默认非root用户，本文中用 `lz` 用户。***

安装Mysql后打开时报错：

```shell
$ sudo apt-get install mysql-server
$ service mysql start
$ mysql -u root -p
Enter password:  # 无论输入什么密码
ERROR 1698 (28000): Access denied for user 'root'@'localhost'
```

root用户被随机设置了一个密码，但是这个密码无从得知，因而无法进入，解决办法：

```shell
$ sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

在该配置文件中的[mysqld]块中加入一行`skip-grant-tables`：
```conf
[mysqld]
#
# * Basic Settings
#
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
lc-messages-dir = /usr/share/mysql
skip-external-locking
skip-grant-tables
```

该语句作用为免密码登录Mysql。

重启Mysql服务：

```shell
$ service mysql restart
```

进入Mysql：

```shell
$ mysql -u root -p
Enter password:　　＃ 直接回车
```

```SQL
mysql> use mysql;
mysql> update user set authentication_string=password("新密码") where user="root";
mysql> flush privileges;
mysql> quit;
```

回到`mysqld.cnf`中将免密登录的那一条语句注释掉：

```conf
[mysqld]
#
# * Basic Settings
#
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
lc-messages-dir = /usr/share/mysql
skip-external-locking
# skip-grant-tables
```

重启Mysql服务：

```shell
$ service mysql restart
```

再返回终端输入`$ mysql -u root -p`，应该就可以进入数据库了。

若依然报错：

```shell
$ mysql -u root -p
Enter password:  # 输入密码
ERROR 1524 (HY000): Plugin 'auth_socket' is not loaded
```

则需要返回`mysqld.cnf`文件将注释掉的语句重新生效：

```conf
[mysqld]
#
# * Basic Settings
#
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
lc-messages-dir = /usr/share/mysql
skip-external-locking
skip-grant-tables
```

重启Mysql服务：

```shell
$ service mysql restart
```

重新免密进入Mysql：

```shell
$ mysql -u root -p
Enter password:  ＃ 直接回车
```

进入mysql数据库，查看user表的`user`,`plugin`：

```SQL
mysql> use mysql;
mysql> select user,plugin from user;
```

应该显示如下结果：

```
+------------------+-----------------------+
| user             | plugin                |
+------------------+-----------------------+
| root             | auth_socket           |
| mysql.session    | mysql_native_password |
| mysql.sys        | mysql_native_password |
| debian-sys-maint | mysql_native_password |
+------------------+-----------------------+
4 rows in set (0.00 sec)
```

错误原因为`plugin root`的字段是`auth_socket`，改掉它为下面的`mysql_native_password`即可：

```SQL
mysql> update user set authentication_string=password("新密码"),plugin='mysql_native_password' where user='root';
```

此时再查看user表的`user`,`plugin`应该为：

```
+------------------+-----------------------+
| user             | plugin                |
+------------------+-----------------------+
| root             | mysql_native_password |
| mysql.session    | mysql_native_password |
| mysql.sys        | mysql_native_password |
| debian-sys-maint | mysql_native_password |
+------------------+-----------------------+
4 rows in set (0.01 sec)
```

最后再把`mysqld.cnf`的那行再次注释掉：

```conf
[mysqld]
#
# * Basic Settings
#
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
lc-messages-dir = /usr/share/mysql
skip-external-locking
# skip-grant-tables
```

重启Mysql服务：

```shell
$ service mysql restart
```

这次使用 `$ mysql -u root -p`应该可以正常输入密码登陆了：

```shell
$ mysql -u root -p
Enter password:  # 输入密码
Welcome to the MySQL monitor.  Commands end with ; or \g.

...

mysql > 
```
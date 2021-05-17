---
title: MySQL安装
date: 2021-05-17 17:17:19
tags: mysql
category: 数据库
---

# mysql安装

## 安装环境

### 系统版本

LSB Version:	n/a
Distributor ID:	ManjaroLinux
Description:	Manjaro Linux
Release:	17.1.11
Codename:	Hakoila

### mysql版本

8.0.12

## 安装前准备

### 下载 mysql

下载地址 ：https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.12-linux-glibc2.12-x86_64.tar.xz

下载下来先将压缩文件解压到**/usr/local，**将长长的文件夹名称改为mysql



## 安装mysql

### 创建用户及用户组

```shell
#创建用户组mysql
groupadd mysql 
#-r参数表示mysql用户是系统用户，不可用于登录系统，创建用户mysql并将其添加到用户组mysql中
useradd -r -g mysql mysql
#改变myql拥有者
chown -R mysql mysql/
#改变mysql用户组
chgrp -R mysql mysql/
```



### 创建配置文件

文件位置/etc/my.cnf

```ini
[client]
default-character-set=utf8
port = 3306
socket = /home/data/mysql/tmp/mysql.sock

[mysql]
default-character-set=utf8
port = 3306
socket = /home/data/mysql/tmp/mysql.sock

[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'
basedir=/usr/local/mysql
#因为我的数据盘挂载在/home目录，所以数据放在data下
#需要授权/home/data/mysql目录
#chown -R mysql /home/data/mysql
#chgrp -R mysql /home/data/mysql
datadir=/home/data/mysql/data
socket=/home/data/mysql/tmp/mysql.sock
log-error=/home/data/mysql/var/log/mysqld.log
pid-file=/home/data/mysql/var/run/mysqld/mysqld.pid
#不区分大小写
lower_case_table_names = 1
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
max_connections=5000
default-time_zone = '+8:00'

# skip-grant-tables
```

### 初始化数据库

用下面语句初始化数据库(需要用root权限执行)

```shell
sudo ./mysqld --initialize --user=mysql
```



### 启动数据库

 启动mysql 需要权限

```shell
sudo /usr/local/mysql/support-files/mysql.server start
```



### 连接数据库

```shell
./mysql -uroot -p
```

**首次连接，可能会报错如下：**

```
./mysql: error while loading shared libraries: libtinfo.so.5: cannot open shared object file: No such file or directory
```

原因：缺少libncurses.so.5

在arch里是ncurses5-compat-libs，同样进行安装：

```shell
sudo pacman -S ncurses5-compat-libs　
```

centos

```shell
 yum install  libncurses.so.5 
```

查看默认密码（默认密码为vpet9Ph;d7al）

```shell
cat /home/data/mysql/var/log/mysqld.log | grep "A temporary password"
2018-08-12T07:14:18.140346Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: vpet9Ph;d7al
```

修改密码

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码'; 
```



### 添加用户

```mysql
-- 使用mysql 数据库
USE mysql
-- 为mysql创建用户：ljs 密码为：ljs
CREATE USER ljs@'%' IDENTIFIED BY 'ljs';
-- 授权
GRANT EXECUTE,INSERT,SELECT,UPDATE ON test.* TO 'ljs'@'%';
-- 刷新权限
FLUSH PRIVILEGES;
-- 再次查询 下权限
SELECT *  FROM USER WHERE USER='ljs' ;
SHOW GRANTS FOR ljs;
```











> 参考 https://www.cnblogs.com/ralap7/p/9034879.html
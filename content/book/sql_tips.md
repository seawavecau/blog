---
title: "Sql相关命令"
date: 2019-11-01T13:21:21+08:00
draft: flase
categories: [Sql]
tags: [Sql,Mysql,创建用户,修改密码]
---

# 1. Mysql
## 创建用户命令：
```
mysql> CREATE USER yy IDENTIFIED BY '123';
```
* yy表示你要建立的用户名，后面的123表示密码

* 上面建立的用户可以在任何地方登陆。

* 如果要限制在固定地址登陆，比如localhost 登陆：

```
mysql> CREATE USER yy@localhost IDENTIFIED BY '123';
```

## grant命令:

```
mysql> GRANT ALL PRIVILEGES ON *.* TO user;@localhost
# 格式：grant select on 数据库.* to 用户名@登录主机 identified by "密码"
mysql> grant select,insert,update,delete on *.* to test1@"%" Identified by "abc";
```
  
## 修改密码：
```
mysql> grant   all   privileges   on   pureftpd.*   to   koko@localhost   identified   by   'mimi';  
# flush:
mysql> flush privileges;
# 查看用户信息：
mysql> select host,user from mysql.user; 
```

## 登陆
```
# 普通登录
mysql -uroot -p
# 如果忘记密码，通过下面启动命令启动，则root密码为空。
mysqld --console

```
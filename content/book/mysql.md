---
title: "Mysql"
date: 2019-11-27T11:34:36+08:00
draft: true
---

# Mysql基础
## 安装
    后台登录 mysql -u root -p 123456
    创建备用mysql账号 CREATE USER yy@localhost IDENTIFIED BY '123';
    授权访问 GRANT ALL PRIVILEGES ON *.* TO hd@%;
    flush flush privileges;
    Mysql8密码加密切换为native ALTER USER 'hd'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

# Mysql高阶
## MySQL 数据库双向同步复制
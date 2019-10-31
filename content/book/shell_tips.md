---
title: "Shell相关命令"
date: 2019-10-31T09:20:44+08:00
draft: false
categories: [Shell]
tags: [Shell,SSH,ssh-keygen,免密登录]
---

# 1. SSH
## 1.1 SSH 免密登录
* 终端Host产生RSA公私钥
``` shell
# 产生key
ssh-keygen -i rsa -C "root@aliyun"
# copy公钥到远程机器
scp ~/.ssh/id_rsa.pub root@xxxxx:
# 登陆远程机器
ssh root@xxxxx
# 把上传的公钥追加到authorized_keys文件
cat id_rsa.pub >> .ssh/authorized_keys
# 退出登陆，再ssh一下检验是否成功
```

## 1.2 SSH 别名登陆
```
# 在.ssh目录下创建config
touch config

# 编辑config文件
vi config
```

* Host是服务别名，HostName你的IP，文件内容如下
```
Host g
HostName 35.200.115.242
User root
```
访问服务器用`ssh g`就可以登录了。
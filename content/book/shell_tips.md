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
# copy公钥到远程机器（google云在浏览器ssh下，用sudo su切换到root）
scp ~/.ssh/id_rsa.pub root@xxxxx:
# 登陆远程机器
ssh root@xxxxx
# 把上传的公钥追加到authorized_keys文件
cat id_rsa.pub >> .ssh/authorized_keys
# 退出登陆，再ssh一下检验是否成功
```
## 1.2 SSH root启用远程登录
``` shell
# 修改/etc/ssh/sshd_config文件，启用下面属性
PermitRootLogin yes
PubkeyAuthentication yes
PasswordAuthentication yes
```
## 重启sshd
```systemctl resart sshd``` 使配置生效

## 1.3 SSH 别名登陆
``` shell
# 在.ssh目录下创建config
touch config

# 编辑config文件
vi config
```

* Host是服务别名，HostName你的IP，文件内容如下
``` shell
Host g
HostName 35.200.115.242
User root
```
访问服务器用`ssh g`就可以登录了。

# 2. 系统用户
## 2.1 创建系统用户
```
# 创建
adduser dev
# 设置密码
passwd dev  #输入新密码
```

## 2.2 修改文件夹owner
``` shell
# 创建一个es新用户，并运行程序
adduser es
chown -R es:es elasticsearch
su es
./elasticsearch -d  # -d 表示后台运行
```

# 3. tree命令技巧
## 3.1 常见命令
```
tree
tree folder     
tree -L 2       # 遍历2层目录显示
tree -C         # 加颜色
tree -N folder  # 显示中文乱码
```
## 3.2 其他选项：
`
-a：显示所有文件和目录；
-A：使用ASNI绘图字符显示树状图而非以ASCII字符组合；
-C：在文件和目录清单加上色彩，便于区分各种类型；
-d：先是目录名称而非内容；
-D：列出文件或目录的更改时间；
-f：在每个文件或目录之前，显示完整的相对路径名称；
-F：在执行文件，目录，Socket，符号连接，管道名称名称，各自加上”*”，”/”，”@”，”|”号；
-g：列出文件或目录的所属群组名称，没有对应的名称时，则显示群组识别码；
-i：不以阶梯状列出文件和目录名称；
-l：<范本样式> 不显示符号范本样式的文件或目录名称；
-l：如遇到性质为符号连接的目录，直接列出该连接所指向的原始目录；
-n：不在文件和目录清单加上色彩；
-N：直接列出文件和目录名称，包括控制字符；
-p：列出权限标示；
-P：<范本样式> 只显示符合范本样式的文件和目录名称；
-q：用“？”号取代控制字符，列出文件和目录名称；
-s：列出文件和目录大小；
-t：用文件和目录的更改时间排序；
-u：列出文件或目录的拥有者名称，没有对应的名称时，则显示用户识别码；
-x：将范围局限在现行的文件系统中，若指定目录下的某些子目录，其存放于另一个文件系统上，则将该目录予以排除在寻找范围外。
`

# 4. 网络命令
## 4.1 ab 命令
### 安装
``` shell
# centos 安装
yum -y install httpd-tools
```
### 使用
```shell
# ab -n 请求次数 -c 并发数 url
ab -n 10 -c 2 http://baidu.com
# 并发测试
ab -r -n 150000 -c 10000 http://url
# post文件
ab -n 100 -c 10 -k -T "application/json" -p data.txt http://xxxxx
# post 字符串
ab -n 100 -c 10 -k -T "application/json" -p "{'key1': 'value1', 'key2': 'value2'}" "http://test.com/post" >> report.html
# post 参数值
ab -n 100 -c 10 -k -p "k1=value1&k2=value2" http://xxxx
```
* 结合python/shell操作使用,可以套用for循环
* 参数
```
-n ：总共的请求执行数，缺省是1；
-c： 并发数，缺省是1；
-t：测试所进行的总时间，秒为单位，缺省50000s；
-k：发送keep-alive指令到服务器；
-T：header头内容类型，T是大写的，内容是字符串形式；
-p：post请求，后面加post参数文档路径，默认为当前路径或者-p后可以是json格式、可以是&连接参数，放在""双引号之间；
-w：以HTML表格输出结果，默认是两列宽度的一个表；
-C：请求附加一个cookie，典型形式是cookieName = value；C大写
-H：请求附加多个cookie；-H  “Cookie: key1=value1; key2=value2”
-i：执行的是head请求不是get；
```
* 延伸阅读：[wrk,ab,jmeter,lutos压测工具比较](https://testerhome.com/topics/17068)

## 4.2 netstat 命令

``` shell
# centos安装目录
yum install net-tools
# 使用命令,通过端口查看程序是否启动
netstat -an | grep 端口
#结果如下：
# tcp        0      0 0.0.0.0:11800           0.0.0.0:*               LISTEN
# 还可以这么用
netstat -anltp|grep 22
# 还可以 这么用
netstat -anp|grep java
```

## 4.3 Mac下查看对方服务端口是否开启命令
``` nc -vz -w 2 192.168.50.8 12800 ```
* 等同于 `telnet ip 端口`



# 5. SHELL语法
## 5.1 while循环
``` shell
while true
    do
    .....    #执行命令
    sleep 3  #单位s
    done
```

# 6. Centos系统设置
## 修改系统时间和时区
### 手动修改
``` shell
# 修改时区
cp  /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
# 修改时间
date -s  ‘2019-11-11 10:50:00’
# 使用date命令查看是否成功
date
> 2019年 11月 05日 星期二 10:29:01 CST
# 将时间写入bios避免重启失效
hwclock -w

```
### 修改时区另一种办法，需重启生效
修改/etc/sysconfig/clock文件，把ZONE的值改为Asia/Shanghai，UTC值改为false，改完后的文件如下：
```
# The time zone of the system is defined by the contents of /etc/localtime.
# This file is only for evaluation by system-config-date, do not rely on its
# contents elsewhere.
ZONE="Asia/Shanghai"
UTC=false
ARC=false
```

### 同步网络
* 外网同步
```shell
ntpdate -u time.windows.com 
```

* 局域网内同步
选择局域网中的一台机器作为ntp服务器，ntpd 配置.

配置开机自启动
``` shell 
chkconfig ntpd on
```
# 7. crontab定时任务
* crontab就是一个自定义定时器
## crontab 服务命令
```shell
sudo service crond start     #启动服务
sudo service crond stop      #关闭服务
sudo service crond restart   #重启服务
sudo service crond reload    #重新载入配置
sudo service crond status    #查看服务状态
```
## 定时任务设定
```shell
# 添加定时任务
# 方法一
crontab update_blog.cron
# 方法二
crontab -e
# 添加 0 12,23 * * * /etc/cron.d/update_blog.sh >> /etc/cron.d/update_blog.log
# 查看定时任务
crontab -l
```

# 8. zsh
    按方向键↑和↓来查看之前执行过
    可以用 !!来执行上一条命令
    使用 ctrl-r 来搜索命令历史记录
    命令和文件补全(按tab键)
    .zshrc 中添加 alias shortcut='this is the origin command'
    j hado 即可正确跳转。j --stat 可以看你的历史路径库。
    输入 d，即可列出你在这个会话里访问的目录列表
    通配符搜索：ls -l */.sh 文件少时可以代替 find
## 文件查找
```find . -name 'Loan*Mapper.java'```
## 插件
* 命令行google: web-search
* 应用使用命令行:
    - 在bash 中添加IDEA： Tools->Create Command-Line Launcher...；添加后使用 idea . 命令打开
## 扩展阅读
[玩转Mac常用命令、zsh等技巧](https://www.jianshu.com/p/28de342f5ecc)


# 9. Brew 
* cat高亮插件:```brew install ccat```
* tree命令
* 查字典dict computer:```sudo pip install dict-cli```
* open 打开目录
* wc 查看文件多少行:```wc -l out.txt```
* 查看前N行:```head -10 xx.txt ```
* tail -100 xxx.txt,more xxx.txt,
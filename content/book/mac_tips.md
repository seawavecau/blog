---
title: "Mac 技巧"
date: 2019-11-01T11:11:37+08:00
draft: flase
categories: [Mac]
tags: [Mac,OS X,TimeMachine]
---

# 1. TimeMachine备份
* 解决存储空间一般会排除一些大空间的应用，例如MS Office家族
* 还想要排除Library下的XCode，但是这些文件在Finder里是隐藏的，使用下面命令显示它：
``` shell
chflags nohidden ~/Library  #显示
chflags hidden ~/Library #用完后再隐藏
```
* 最后限制在~50G

# 2. 代理
* 有时候使用全局Http代理，访问局域网的时候找不到路径
* 使用网络设置里的代理，配置“忽略主机和域”的代理设置。
![network_1.png](/img/02_mac_tips/01.jpg)
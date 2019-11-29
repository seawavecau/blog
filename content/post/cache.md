---
title: "缓存使用"
date: 2019-11-28T10:09:06+08:00
draft: true
---



使用缓存的正确方式:
* 先更新到数据库,然后再删除缓存.
* 缓存要设置失效时间(几秒到几分钟.特殊的可以设置长一些)
* 对数据实时性要求高的场景,
 
## 参考
[使用缓存的正确姿势](https://juejin.im/post/5af5b2c36fb9a07ac65318bd)
[springboot 使用缓存](https://blog.csdn.net/tianyaleixiaowu/article/details/70314277)
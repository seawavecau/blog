---
title: "Springboot中的tomcat"
date: 2019-11-07T11:38:24+08:00
draft: false
---

## 概述
*Tomcat连接器Connector的主要功能是<strong>接收连接请求，创建request和response对象用于和请求端交换数据；然后分配线程给engine（servlet容器）来处理请求，并把request和response对象传递给engine引擎，当engine处理完请求后，也会通过Connector将响应返回给客户端。</strong>

在accept队列中接收连接（当客户端向服务器发送请求时，如果客户端与OS完成三次握手建立了连接，则OS将该连接放入accept队列）；在连接中获取请求的数据，生成request；调用servlet容器处理请求；返回response。为了便于后面的说明，首先明确一下连接与请求的关系：连接是TCP层面的（传输层），对应socket；请求是HTTP层面的（应用层），必须依赖于TCP的连接实现；一个TCP连接中可能传输多个HTTP请求。

在BIO实现的Connector中，处理请求的主要实体是JIoEndpoint对象。JIoEndpoint维护了Acceptor和Worker：Acceptor接收socket，然后从Worker线程池中找出空闲的线程处理socket，如果worker线程池没有空闲线程，则Acceptor将阻塞。其中Worker是Tomcat自带的线程池，如果通过<Executor>配置了其他线程池，原理与Worker类似。

在NIO实现的Connector中，处理请求的主要实体是NIoEndpoint对象。NIoEndpoint中除了包含Acceptor和Worker外，还是用了Poller，处理流程如下图所示（图片来源：http://gearever.iteye.com/blog/1844203）。

## Springboot自带Tomcat的最大连接数和并发数

* 默认最大连接数和并发数是10000和200
* 可以通过工程下的application.yml配置文件来改变这个值
``` yml
server: 
  port: 18080
  tomcat:
    max_threads: 500         #最大并发数
    max-connections: 20000   #最大连接数
    min-SpareThreads: 20     #初始化时创建的线程数
    acceptCount: 700         #可以放到处理队列中的请求数
```
## 参数说明
maxThreads="1000" 最大并发数 
minSpareThreads="100"///初始化时创建的线程数
maxSpareThreads="500"///一旦创建的线程超过这个值，Tomcat就会关闭不再需要的socket线程。
acceptCount="700"// 指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理

maxThreads 客户请求最大线程数 
minSpareThreads Tomcat初始化时创建的 socket 线程数 
maxSpareThreads Tomcat连接器的最大空闲 socket 线程数 
enableLookups 若设为true, 则支持域名解析，可把 ip 地址解析为主机名 
redirectPort 在需要基于安全通道的场合，把客户请求转发到基于SSL 的 redirectPort 端口 
acceptAccount 监听端口队列最大数，满了之后客户请求会被拒绝（不能小于maxSpareThreads ） 
connectionTimeout 连接超时 
minProcessors 服务器创建时的最小处理线程数 
maxProcessors 服务器同时最大处理线程数 
URIEncoding URL统一编码

 

maxThreads：处理的最大并发请求数，默认值200
minSpareThreads：最小线程数始终保持运行，默认值10
maxConnections：在给定时间接受和处理的最大连接数，默认值10000


## 深入阅读
[tomcat高并发下优化详解及连接数和线程池](https://blog.csdn.net/jaryle/article/details/83616146)
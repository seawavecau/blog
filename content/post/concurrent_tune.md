---
title: "高并发调优"
date: 2019-11-23T16:34:48+08:00
draft: true
---
# 高并发架构图
略 （Nginx->Tomcat->(Redis、Kafka)->Dubbo Service->Database)

# Nginx调优
* 打印linux系统配置参数
```sysctl -a > ~/sys.config ```
## 操作系统设置
1) 修改socket最大连接数
   ``` shell
   #系统默认的值是128，现在改成50000
   echo 50000 > /proc/sys/net/core/somaxconn
   ```
2) 加快系统的tcp回收机制
   ```shell
   # 系统默认tcp在断开后还会存活一段时间,TCP连接立即回收、回用（recycle、reuse）
   # 系统默认是0，修改为1
   echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse
   echo 1 > /proc/sys/net/ipv4/tcp_tw_recycle
   ```
3) 让系统不做洪水抵御
   * 当系统检测到80端口在大量的请求时，会自动给返回信息中增加 cookie ,还验证客户端身份，从而避免受到攻击，但这时只是高并发，并不是攻击，所以要把这个抵御机制给关闭。
   ```shell
   # 系统默认为1，修改为0
   echo 0 >/proc/sys/net/ipv4/tcp_syncookie 
   ```
4) 调整同时打开文件数量
   ```ulimit -n 65536 ```

使用：sysctl -p 生效
```sysctl -p ```

## Nginx层面
### 优化Nginx Worker进程数
* nginx 配置子进程可以打开的文件个数和连接数。worker processes、workder connections
   ```shell
   # 修改nginx配置文件，nginx.conf
   # 增加work_rlimit_nofile和worker_connections数量，并禁用keepalive_timeout。
   worker_processes  1; #nginx 进程数，建议CPU总核数*2(#LinuxCPU查看)，一般为当前机器总cpu核心数的1到2倍
   worker_rlimit_nofile 65535; #一个nginx 进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx 进程数相除，但是nginx 分配请求并不是那么均匀，所以最好与ulimit -n 的值保持一致

   events {
        #使用epoll的I/O模型，一个可扩展的I/O事件通知机制来触发事件
        use epoll;
        # 每个进程允许的最多连接数， 理论上每台nginx 服务器的最大连接数为,worker_processes*worker_connections
        # 等于 ulimit -n 输出的值
        worker_connections 65536;
        # 使worker一次接受所有新连接
        multi_accept on;
    }
    http {
        keepalive_timeout 65;
        client_header_timeout 15;   # 用于设置读取客户端请求头数据的超时时间
        client_body_timeout 15;     # 用于设置读取客户端请求主体数据的超时时间
        send_timeout 25;        # 用于指定响应客户端的超时时间
        # 设置客户端最大的请求主体大小为 8 M
        client_max_body_size 8m;    
    }
   ```

1) 重启nginx
```shell
systemctl restart nginx
```

### ab压力测试
```shell
ab -r -n 150000 -c 10000 "http://blog.wacai.ml"
```
* 报告
```
Server Software:        nginx/1.16.1
Server Hostname:        blog.wacai.ml
Server Port:            80

Document Path:          /
Document Length:        26600 bytes

Concurrency Level:      10000               并发数量
Time taken for tests:   307.169 seconds     测试时间
Complete requests:      150000              总请求数
Failed requests:        33826
   (Connect: 0, Receive: 10996, Length: 11834, Exceptions: 10996)
Write errors:           0
Total transferred:      3786580664 bytes
HTML transferred:       3752525864 bytes
Requests per second:    488.33 [#/sec] (mean)   吞吐量
Time per request:       20477.906 [ms] (mean)   每个并发组请求的平均时间
Time per request:       2.048 [ms] (mean, across all concurrent requests)   服务器处理每个请求的时间
Transfer rate:          12038.45 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0 1095 4964.5     80   66644
Processing:   150 14021 32954.8   3078  262640
Waiting:        0  788 2388.2     82  111092
Total:        225 15116 33228.1   3695  269736
```
### 优化参考
[Nginx服务器性能调优](https://www.centos.bz/2016/10/tunning-nginx-server-performance/)
[Nginx 配置和性能调优](https://blog.csdn.net/lamp_yang_3533/article/details/80383039)
[nginx高并发优化](https://blog.csdn.net/nuli888/article/details/51865267)
[nginx单机150w的QPS](https://blog.csdn.net/mergerly/article/details/71198712)


# 参考
[天猫双十一架构演变，亿级流量](https://cloud.tencent.com/developer/article/1539546)
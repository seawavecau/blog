---
title: "Java应用接入Skywalking 6"
date: 2019-11-02T09:32:04+08:00
draft: false
categories: [架构]
tags: [skywalking,架构,oap,监控,性能,全链路]
---

# 概述
  SkyWalking 是观察性分析平台和应用性能管理系统。

  提供分布式追踪、服务网格遥测分析、度量聚合和可视化一体化解决方案.

## 特性：
* 多种监控手段，语言探针和service mesh
* 多语言自动探针，Java，.NET Core和Node.JS
* 轻量高效，不需要大数据
* 模块化，UI、存储、集群管理多种机制可选
* 支持告警
* 优秀的可视化方案

## 架构图：
![skywalking架构图.png](/img/04_skywalking/01.jpeg)

SkyWalking 逻辑上分为四部分: 探针, 平台后端, 存储和用户界面。

* 探针
    主要负责从客户端收集数据，将数据转换成SkyWalking适用的格式，探针对客户端程序没有任何代码侵入，使用起来简单方便，使用如下命令即可完成对应用的监控java -javaagent:/path/skywalking-agent.jar -jar youApp.jar
* 平台后端（OAP Server）
    主要用于数据聚合, 数据分析以及驱动数据流从探针到用户界面的流程。通过 gRPC/Http 收集客户端Agent的采集信息 ，Http默认端口 12800，gRPC默认端口 11800。
* 存储
    SkyWalking支持很多存储：H2（用作演示环境）、MySQL（当数据量大时检索性能下降很厉害）、ES（主流生产级别的存储）
* 用户界面
    精美的后台，监控详情一目了然。


# 本地环境安装

## 在centos7下安装skywalking6.4.0
``` sh
# 下载skywalking
wget http://mirrors.tuna.tsinghua.edu.cn/apache/skywalking/6.4.0/apache-skywalking-apm-6.4.0.tar.gz

# 解压缩
tar -zxvf apache-skywalking-apm-6.4.0.tar.gz

# 运行startup.sh，会启动Collector和Webapp
bin/startup.sh
```

* 经过以上步骤，skywalking就启动起来了。主要一个是Collector用来采集数据，还有一个Web app，是Skywalking的UI后台。
* 端口说明：
    - 11800：Collector的gRPC协议的端口，采集端口
    - 12800：Collector开放REST接口
    - 8080： webapp后台端口
    - 18080：测试用web端口

### oapServer和webapp日志查看
``` shell
# 查看oapServer 日志
tail -f logs/skywalking-oap-server.log
# 查看webapp日志
tail -f logs/webapp.log
```
观察日志，如果没有错误，我们接着run应用来上传数据
  
## 应用接入Skywalking
skywalking使用探针技术，对应用无侵入。启动时加上`-javaagent`参数即可
``` shell
# 启动生产者
java -jar -javaagent:$AGENT_PATH/skywalking-agent.jar  dubbo-provider.jar

# 启动消费者
java -jar -javaagent:$AGENT_PATH/skywalking-agent.jar  dubbo-consumer.jar 

# 查看agent日志,如果是status:CONNECTED 表示OK
tail -f logs/skywalking-api.log
```
  
使用前，要配置`skywalking/agent/config`目录下的`agent.config`文件, 指向Collector服务器地址
``` shell
# 设置代理应用名称
agent.service_name=${SW_AGENT_NAME:Demo}

# 设置collector服务器地址
collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:192.168.50.8:11800}
```

# 监控后台

* 服务可用性指标 SLA
* 每分钟平均响应数
* 平均响应时间
* 服务进程 PID
* 服务所在物理机的 IP、Host、OS
* 运行时 CPU 使用率
* 运行时堆内存使用率
* 运行时非堆内存使用率
* GC 情况
  
## Trace ID展开详细信息

# Alarm配置

# 生产环境安装
生产环境使用ElasticSearch做存储。先安装ES
## ElasticSearch集群安装
* 安装步骤略过，自行google
* 修改ES配置，`/usr/local/etc/elasticsearch/elasticsearch.yml`
``` shell
# 设置cluster.name设置成CollectorDBCluster
cluster.name: CollectorDBCluster
network.host: 0.0.0.0
# ---------------------------------- elasticsearch - 允许跨域 ----------------------
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-methods: OPTIONS, HEAD, GET, POST, PUT, DELETE
http.cors.allow-headers: "X-Requested-With, Content-Type, Content-Length, X-User"
```
* 通过`curl http://localhost:9200` 验证

## 部署skywalking
* 修改skywalking配置`vim apache-skywalking-apm-bin/config/application.yml`
* 将h2内容注释掉，启用elasticsearch内容，使用elasticsearch存储数据。
``` shell
storage:
  elasticsearch:
    nameSpace: ${SW_NAMESPACE:"CollectorDBCluster"}                 # 和ES的cluster name一致
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:192.168.50.143:9200}# 指向es服务器
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
#    trustStorePath: ${SW_SW_STORAGE_ES_SSL_JKS_PATH:"../es_keystore.jks"}
#    trustStorePass: ${SW_SW_STORAGE_ES_SSL_JKS_PASS:""}
#    user: ${SW_ES_USER:""}
#    password: ${SW_ES_PASSWORD:""}
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
    # Those data TTL settings will override the same settings in core module.
#    recordDataTTL: ${SW_STORAGE_ES_RECORD_DATA_TTL:7} # Unit is day
#    otherMetricsDataTTL: ${SW_STORAGE_ES_OTHER_METRIC_DATA_TTL:45} # Unit is day
#    monthMetricsDataTTL: ${SW_STORAGE_ES_MONTH_METRIC_DATA_TTL:18} # Unit is month
    # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:1000} # Execute the bulk every 1000 requests
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
    metadataQueryMaxSize: ${SW_STORAGE_ES_QUERY_MAX_SIZE:5000}
    segmentQueryMaxSize: ${SW_STORAGE_ES_QUERY_SEGMENT_SIZE:200}
#  h2:
#    driver: ${SW_STORAGE_H2_DRIVER:org.h2.jdbcx.JdbcDataSource}
#    url: ${SW_STORAGE_H2_URL:jdbc:h2:mem:skywalking-oap-db}
#    user: ${SW_STORAGE_H2_USER:sa}
#    metadataQueryMaxSize: ${SW_STORAGE_H2_QUERY_MAX_SIZE:5000}
```
启动SkyWalking `bin/startup.sh`。通过浏览器访问`http://localhost:8080`验证

# 高级
## Agent 配置文件详解
``` conf
# 当前的应用编码，最终会显示在webui上。
# 建议一个应用的多个实例，使用有相同的application_code。请使用英文
agent.application_code=Your_ApplicationName

# 每三秒采样的Trace数量
# 默认为负数，代表在保证不超过内存Buffer区的前提下，采集所有的Trace
# agent.sample_n_per_3_secs=-1

# 设置需要忽略的请求地址
# 默认配置如下
# agent.ignore_suffix=.jpg,.jpeg,.js,.css,.png,.bmp,.gif,.ico,.mp3,.mp4,.html,.svg

# 探针调试开关，如果设置为true，探针会将所有操作字节码的类输出到/debugging目录下
# skywalking团队可能在调试，需要此文件
# agent.is_open_debugging_class = true

# 对应Collector的config/application.yml配置文件中 agent_server/jetty/port 配置内容
# 例如：
# 单节点配置：SERVERS="127.0.0.1:8080" 
# 集群配置：SERVERS="10.2.45.126:8080,10.2.45.127:7600" 
collector.servers=127.0.0.1:10800

# 日志文件名称前缀
logging.file_name=skywalking-agent.log

# 日志文件最大大小
# 如果超过此大小，则会生成新文件。
# 默认为300M
logging.max_file_size=314572800

# 日志级别，默认为DEBUG。
logging.level=DEBUG
```

# 参考和扩展阅读
* [skywalking P.K. Pinpoint](https://skywalking.apache.org/zh/blog/2019-02-24-skywalking-pk-pinpoint.html)
* [全链路监控](https://juejin.im/post/5a7a9e0af265da4e914b46f1#heading-14)
* [存储设置详解](https://github.com/apache/skywalking/blob/v6.0.0-GA/docs/en/setup/backend/backend-storage.md)
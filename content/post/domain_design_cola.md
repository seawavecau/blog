---
title: "领域设计框架COLA"
date: 2019-11-06T15:28:51+08:00
draft: false
categories: [领域设计]
tags: [领域设计,cola,DDD,架构,架构分层,二方]
---

# COLA 2.0概述
COLA是Clean Object-Oriented and Layered Architecture的缩写，代表“整洁面向对象分层架构”，也叫“可乐”架构，目前COLA已经发展到COLA 2.0。

# COLA 2.0 框架
* COLA作为框架主要是提供应用和架构需要的通用组件支撑，比如对CQRS和扩展点功能的支持。COLA框架主要由cola-framework这个项目来实现。 在这个项目里面包含3个Module：cola-core, cola-common和cola-test。
* 架构图
![架构图.png](https://i.loli.net/2019/11/12/zHEDYPN6gLja27T.jpg)

## cola-core
该Module是整个框架的核心，里面的主要功能和Package如下：
``` shell
com
└── alibaba
    └── cola
        ├── assembler  \\提供Assembler标准
        ├── boot \\这是框架的核心启动包，负责框架组件的注册、发现
        ├── command  \\提供Command标准
        ├── common
        ├── context  \\提供框架执行所需要的上下文
        ├── domain  \\提供Domain Entity标准
        ├── event
        ├── exception \\提供Exception标准
        ├── extension  \\负责扩展机制中的重要概念-扩展(Extension)的定义和执行
        ├── logger  \\提供DIP的日志接口
        ├── repository  \\提供仓储（Repository）的标准
```
## cola-common
* 该Module提供了框架中Client Object, Entity Object和Data Object的定义，二方库会依赖该Module。

## cola-test
* 该Module主要是提供一些开发测试的工具，可以使用TDD来进行开发。
  
# COLA 2.0 架构图
![domain_design_cola_archtect](/img/07_domain_design/01.png)


# COLA 2.0使用
* 官方提供了两个Archetype，分别是cola-archetype-service和cola-archetype-web
## 安装archetype
* 方法一是使用安装命令，直接生产demo工程：
``` shell
mvn archetype:generate  -DgroupId=com.alibaba.demo -DartifactId=demo -Dversion=1.0.0-SNAPSHOT -Dpackage=com.alibaba.demo -DarchetypeArtifactId=cola-framework-archetype-service -DarchetypeGroupId=com.alibaba.cola -DarchetypeVersion=2.0.0
```

* 这里介绍本地install archetype,然后再生成代码。
``` shell
# 下载源码
git clone https://github.com/alibaba/COLA
# 生成archetype骨架模板，并安装到本地
cd cola-archetype-service
# 执行install
mvn install
# 用crawl命令，在本地仓库生成archetype-catalog.xml骨架配置文件
mvn archetype:crawl
# 通过vi命令验证文件内容
vi ~/.m2/repository/archetype-catalog.xml
# 对 cola-archetype-web 同样操作
```
## 使用archetype生成Demo工程
使用命令生成工程
```shell
# 从本地archeType模板中创建项目
mvn archetype:generate -DarchetypeCatalog=local
```
命令交互如下：
```shell
Choose archetype:
1: local -> com.alibaba.cola:cola-framework-archetype-service (demo)
2: local -> com.alibaba.cola:cola-framework-archetype-web (demo)
3: local -> org.apache.maven.archetypes:maven-archetype-quickstart (quickstart)
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): 3: 1
Define value for property 'groupId': com.seawave
Define value for property 'artifactId': demo
Define value for property 'version' 1.0-SNAPSHOT: :
Define value for property 'package' com.seawave: :
Confirm properties configuration:
groupId: com.seawave
artifactId: demo
version: 1.0-SNAPSHOT
package: com.seawave
 Y: :
 ```
* 使用`idea demo`打开，验证项目是否成功
* 生成的目录结构如下
``` shell
├── demo-app        #application
├── demo-client
├── demo-domain
├── demo-infrastructure
├── pom.xml
└── start
```



# 参考
[应用架构 COLA 2.0](https://toutiao.io/posts/9b2wg1d/preview)
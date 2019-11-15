---
title: "Maven"
date: 2019-11-06T13:36:40+08:00
draft: false
categories: [maven]
tags: [maven,install,archetype]
---

# Maven 安装


# maven配置和添加阿里云镜像
* 在settings.xml文件的mirrors节点里，添加
``` xml
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
```

# Maven自定义archetype
## 1. 什么是archetype.
* 当我们用IDEA创建maven项目时，会列出archetype列表。
* 这个等同于`mvn archetype:generate`命令来快速创建maven项目。
![idea_archetype_1.png](/img/05_maven/01.png)
* 每个模板里其实就是附带不同的依赖和插件。一般在公司私服里都会有属于本公司的一套archeType模板，里面有着调试好的项目用到的依赖包和版本号。

## 2. 创建archetype
假如自己已经有了一个maven项目，想给该项目创建一个archeType模板。
cd 到项目根目录下执行(pom.xml同级目录)
``` shell
mvn archetype:create-from-project 
```
* 扫描代码，__xxx__参数会做替换，把文件拷贝到target目录下。
* 此时会在项目target下生成generated-sources文件
最终产生的结构如[阿里COLA Github](https://github.com/alibaba/COLA.git )下的`cola-archetype-web`，就是一个典型的archetype目录结构。把generated-source/archetype目录下的代码作为工程上传的。

## 3. 生成archetype模板
``` shell
# 进入archetype目录
cd target/generated-sources/archetype/
# 执行install
mvn install
# 用crawl命令，在本地仓库生成archetype-catalog.xml骨架配置文件
mvn archetype:crawl
# 通过ls命令验证文件
ls ~/.m2/repository/archetype-catalog.xml
```
archetype-catalog.xml的文件内容如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<archetype-catalog xsi:schemaLocation="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-catalog/1.0.0 http://maven.apache.org/xsd/archetype-catalog-1.0.0.xsd"
    xmlns="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-catalog/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <archetypes>
    <archetype>
      <groupId>com.alibaba.cola</groupId>
      <artifactId>cola-framework-archetype-web-archetype</artifactId>
      <version>2.0.0</version>
      <description>cola-framework-archetype-web</description>
    </archetype>
  </archetypes>
</archetype-catalog>
```

## 4. 使用archetype模板
```s
# 从本地archeType模板中创建项目
mvn archetype:generate -DarchetypeCatalog=local
```
命令后会有交互，选择相应的模板，然后定义要创建项目的groupId、artifactId和version。
```log
Choose archetype:
1: local -> org.apache.maven.archetypes:maven-archetype-quickstart (quickstart)
2: local -> com.alibaba.cola:cola-framework-archetype-web-archetype (cola-framework-archetype-web)
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): 1: 2
Define value for property 'groupId': com.seawave
Define value for property 'artifactId': demo-test
Define value for property 'version' 1.0-SNAPSHOT: :
Define value for property 'package' com.seawave: :
Confirm properties configuration:
groupId: com.seawave
artifactId: demo-test
version: 1.0-SNAPSHOT
package: com.seawave
 Y: : Y
 ```

 # 4. Maven 命令
查看jar包依赖:
 ```shell
 mvn dependency:tree > dep.log 
 ```
 maven-dependency-plugin最大的用途是帮助分析项目依赖。dependency:list能够列出项目最终解析到的依赖列表，dependency:tree能进一步的描绘项目依赖树，dependency:analyze可以告诉你项目依赖潜在的问题
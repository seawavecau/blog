---
title: "linux常用软件安装"
date: 2019-11-01T19:37:14+08:00
draft: false
---

# Centos7安装Java8
* [下载JDK8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
* 安装JDK
```
# 在/usr/local下创建java文件夹
mkdir /usr/local/java
# 解压
tar -zxvf jdk-8u231-linux-x64.tar.gz
#把解压文件放到java目录下
mv jdk1.8.0_171 /usr/local/java/
# 设置环境变量
sudo vim /etc/profile
```
文件中添加
```
#java
export JAVA_HOME=/usr/local/java/jdk1.8.0_171
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```
保存后推出，让环境变量生效
```
# 生效
source /etc/profile

# 检查是否安装成功
java -version
```

# Mac安装ElasticSearch
Mac支持brew安装
`brew install elasticsearch`

安装好后，用命令启动
``` sh
# 后台启动
brew services start elasticsearch

# 控制台直接启动
elasticsearch
```

安装后的路径用`brew info elasticsearch`命令查看安装信息
```
elasticsearch:  /usr/local/Cellar/elasticsearch/x.x.x
Data:    /usr/local/var/lib/elasticsearch/
Logs:    /usr/local/var/log/elasticsearch/elasticsearch_seawave.log
Plugins: /usr/local/var/elasticsearch/plugins/
Config:  /usr/local/etc/elasticsearch/
plugin script: /usr/local/opt/elasticsearch/libexec/bin/elasticsearch-plugin
```

在浏览器上直接输入`http://127.0.0.1:9200/`可以看到返回内容
``` json
{
"name": "w3V6OEY",
"cluster_name": "elasticsearch_seawave",
"cluster_uuid": "cr0lyWKIRR2GWq4Aoqmnxg",
"version": {
"number": "6.8.4",
"build_flavor": "oss",
"build_type": "tar",
"build_hash": "bca0c8d",
"build_date": "2019-10-16T06:19:49.319352Z",
"build_snapshot": false,
"lucene_version": "7.7.2",
"minimum_wire_compatibility_version": "5.6.0",
"minimum_index_compatibility_version": "5.0.0"
},
"tagline": "You Know, for Search"
}
```
* ElasticSearch管理工具的安装见下节

简单CRUD

# 安装ElasticSearch-head
独立webapp运行
安装node.js
``` shell
# 先安装nvm （node版本管理器）
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash

# 激活nvm
source ~/.nvm/nvm.sh

# 安装node
nvm install node

# 安装完，切换到该版本
nvm use node

# 安装grunt，是构建工具，类似Java的maven工具。安装完用grunt -version检查
cd /usr/local/elk/elasticsearch/elasticsearch-head
npm install -g grunt-cli

# 安装Head插件
cd /usr/local/elk/elasticsearch/
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head/
npm install
```

安装完后，修改服务器监听地址。目录：elasticsearch-head/Gruntfile.js,增加hostname属性，设置为*
``` shell

connect: {
    server: {
        options: {
            port: 9100,
            hostname: '*',
            base: '.',
            keepalive: true
        }
    }
}
```

启动Head插件
使用`grunt server` 或 `npm run start`启动插件
访问http://localhost:9100/ 出现以下画面,代表启动成功

目录：elasticsearch-head/_site/app.js,把localhost修改成你es的服务器地址,如:
```
this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://192.168.50.8:9200";
```



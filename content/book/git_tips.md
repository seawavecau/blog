---
title: "Git & MVN使用技巧"
date: 2019-10-31T09:36:40+08:00
draft: true
---

* 以远程为准，抹掉本地
`
$ git fetch --all
$ git reset --hard origin/master 
$ git pull
`

`
# 查看修改
git  diff 文件
`


# Maven
``` shell
# 跳过测试
mvn install -Dmaven.test.skip=true
```

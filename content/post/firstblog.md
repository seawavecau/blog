---
title: "Hugo + Nginx: 利用webhook自动部署"
date: 2019-10-30T21:20:36+08:00
draft: false
categories: [Website]
tags: [Website,Nginx,CentOS,Github,Webhook,自动部署]
---

# 利用Hugo生产静态网站
## 1. 创建blog目录
```
~> hugo new site blog
```
* 会在你的用户下创建blog目录，生产hugo项目文件
```
~> cd blog
```
* 接下来我们使用git做文件管理，执行下列命令行
``` shell
# git 初始化
git init

#将主题maupassant作为外链加入本地仓库
git submodule add https://github.com/rujews/maupassant-hugo.git themes/maupassant

# 修改hugo配置文件，使用下载的博客主题
echo 'theme = "maupassant"' >> config.toml

# 将public作为外链加入本地仓库
git submodule add -b https://github.com/seawavecau/seawavecau.github.io.git public

# 项目关联到你的blog远程代码仓库
git remote add origin https://github.com/seawavecau/blog.git

# 添加博客
hugo new post/firstblog.md
# 创建的第一篇博客方赞content/post目录下，使用编辑工具打开目录下的文件

sublime content/post
```
* 编辑博客内容，使用markdown风格。
* 保存文本内容后，执行hugo提供的调试命令查看效果，检查博文是否生效
* ``` shell
* hugo server -D

# 然后再浏览器访问
http://localhost:1313/
```

* 配置发布脚本
完成博客编写后，执行`hugo` 命令，项目会编译成静态网站资源，保存在`public`文件夹下。然后直接发布到服务器完成部署。
网上有人提供部署脚本，借用如下
``` shell
# cd 到项目根目录
cd ~/blog
~> vim deploy.sh
```
保存一下脚本内容：
``` shell
    echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

    # Build the project.
    hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

    # Go To Public folder
    cd public
    # Add changes to git.
    git add .

    # Commit changes.
    msg="rebuilding site `date`"
    if [ $# -eq 1 ]
    then msg="$1"
    fi
    git commit -m "$msg"

    # Push source and build repos.
    git push origin master

    # Come Back up to the Project Root
    cd ..

```
   以后更新博客，直接执行deploy 部署更新
```
./deploy.sh "git comment"
```
以上完成本地网站源码和静态文件同步到Github仓库。
接下来，服务器还需要一些配置，满足自动化部署

# 服务器配置

## 1. 网站根目录
``` shell
#  创建网站根目录
mkdir /usr/www/blog

# cd 到目录blog2
cd blog

# git 初始化
git init
git remote add origin https://github.com/seawavecau/seawavecau.github.io.git

# 从 Github第一次抓取
git pull origin master
```
## 2.webhook自动更新
利用Github提供的webhook功能，需要再服务端启动一个监听服务，接收github的push event。当你提交工程文件到github后，webhook会往你服务器发送一个push event请求。我们就可以在服务器上执行脚本，从该仓库抓取最新资源，达到博客内容实时更新的目的。

### 2.1 创建webhook目录，创建脚本git_pull.sh
``` shell
#创建目录
mkdir /usr/www/webhook
#进入目录
cd webhook
# 创建脚本
vim git_pull.sh

```
在脚本中保存以下内容：

``` shell
cd /usr/www/blog
git pull origin master
exit 0
```

### 2.2 github_webhook.js脚本
* 我们需要用到nodejs来跑监听服务，需要用到中间件 [github_webhook_handler](https://github.com/rvagg/github-webhook-handler), 我们使用npm 来安装, 如果有问题可以从github上clone一个到本地：
```
npm install github_webhook_handler
```
* 新建`github_webhook.js`脚本
 ```
    vim github_webhook.js
  ```
  写入以下脚本内容：

  ``` shell
  var http = require('http')
  var exec = require('child_process').exec
  var createHandler = require('github-webhook-handler')
  var handler = createHandler({ path: '/webhook', secret: 'xxxx' })

  http.createServer(function (req, res) {
  handler(req, res, function (err) {
      res.statusCode = 404
      res.end('no such location')
    })
  }).listen(7777)

  handler.on('push', function (event) {
      let currentTime = new Date();
      console.log('\n--> ' + currentTime.toLocaleString());
      console.log('Received a push event for %s to %s', event.payload.repository.name, event.payload.ref);
      exec('sh ./git_pull.sh', function (error, stdout, stderr) {
          if(error) {
              console.error('error:\n' + error);
              return;
          }
          console.log('stdout:\n' + stdout);
          console.log('stderr:\n' + stderr);
      });
  })
  ```
* ### pm2启动脚本
先用`node github-webhook.js`启动调试，等调试通过了，我们用pm2来启动：`pm2 start github-webhook.js`，使用`pm2 startup`命令来设置脚本开机启动。

# 3. Nginx配置更新
* 在/etc/nginx/conf.d目录下新建blog.conf
```
cd /etc/nginx/conf.d
vim blog.conf
```
``` shell
server {
        listen       80;
        server_name  blog.xxxxx.com;
        root         /usr/www/blog;

        location / {
		index index.html index.htm index.php;
		autoindex on;
        }
        location /webhook {
		alias /usr/www/webhook/;
		proxy_pass http://127.0.0.1:7777;
        }
        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
``` 
* 重启Nginx
```
systemctl restart nginx
```


# 4 Github配置
* 到目前为止，还差最后一步才能完成自动化部署。要Github上创建一个webhook指向你的服务器。

登录Github,进入` seawavecau.github.io`仓库，点击`Setting`
  ![hugo_1.png](/img/01_hugo_server/01.jpg)
注意事项：

``` shell
第2步：'webhook'即 ‘github_webhook.js’ 中配置的path
第3步：必须选择‘application/json’,否则不起作用
第4步：‘Secret’即 ‘github_webhook.js’中配置的‘ secret_key’
一定要和脚本中设置一致，否则，你懂的！
```

好了，到目前为止，配置工作都已完成。
剩下的只是我们日常的简单操作了，如果我们某天又心血来潮，想写篇博客消遣消遣，该怎么做呢？
辛苦这么久的配置，就是为了这个时刻。

``` shell
cd /xxx/xxx/BlogTest
hugo new post/MySecondBlog.md
# 呃～～编辑博客：想用什么编辑工具都可以， Markdown语法
# 编辑完了？嗯！
./deploy.sh
# 执行完后，不出意外的话，服务器已经已经更新了！ 完美！

# ------------------
# 这样就完了？ ～～
# nooooo~~
# 细心的你肯定已经发现，我们的脚本只是把网站相关的东西push到了xxx.github.io这个仓库
# 源码还没提交备份呢？！！！
git add .
git commit -m '备注：要提交源码啊！'
git push -u origin master
# ALL DONE, 这下服务器也更新了，github源码也提交了，新更新的内容也备份到另一个github仓库。
#再也不怕了。
```
`文章原型参考bones的[博客](https://www.harddone.com/post/hugo_nginx_1/)：https://github.com/LazyBonesLZ/MyBlog 我是搬运工 👍`
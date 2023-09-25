# 一步步实现Nginx +Docker + Jenkins前端自动化部署

实验室每次项目发布测试时，都要手动本地打包好了然后上传到服务器，替换原来nginx下面的目录文件，十分麻烦和繁琐。这次就来优化一下，通过Dockerfile + Jenkins实现自动化部署

# Nginx相关

如果不了解Nginx的同学，可以看一下我之前的实践，了解一下只用Nginx是怎么实现部署的，这样看后面比较好接受

1.  [在一个裸的Ubuntu server中我做了哪些 - 掘金 (juejin.cn)](https://juejin.cn/post/7166502397173301278#heading-4)
2.  [nginx部署前端页面 - 掘金 (juejin.cn)](https://juejin.cn/post/7169064271425306654)
3.  [nginx部署前后端分离项目(二) - 掘金 (juejin.cn)](https://juejin.cn/post/7174200706004418591)

和本篇文章搭配食用，味道更佳哦！！！这里我就简单描述一下

## 安装nginx

一定要按照官方的安装方式来安装nginx，不然可能会出现一些幺蛾子。笔者之前就直接`apt install nginx`，结果反向代理一直不成功，最后还是卸了重新安装的。

官方下载地址：<http://nginx.org/en/linux_packages.html#Ubuntu>

我的服务器是`Ubuntu`的，所以挨个复制粘贴如下指令就好了
![image-20230925195702077](http://120.53.221.17/blog/1695643024370.png)

## 了解nginx配置

`whereis nginx`查看nginx目录

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77cadb82b49d489591a576c09abb328e~tplv-k3u1fbpfcp-watermark.image?)

主要涉及到这个目录文件`/etc/nginx`

![image-20230925200120962](http://120.53.221.17/blog/1695643281208.png)

注意这个目录下面的两个文件`nginx.conf`和`conf.d`

我们先查看`nginx.conf`的文件内容

```bash
cat nginx.conf 
```

结果：

```bash
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a61a2c9488a74f2b80eb791dac86484e~tplv-k3u1fbpfcp-watermark.image?)

所以我们去`/etc/nginx/conf.d/`的目录下面看一下配置文件

![image-20230925200147504](http://120.53.221.17/blog/1695643307747.png)

```bash
cat default.conf 
```

`default.conf`的文件内容：

```bash
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

![image-20230925200215853](http://120.53.221.17/blog/1695643336107.png)

这时候我们输入我们的ip地址，看到的就是nginx的默认页面

![image-20230925200228638](http://120.53.221.17/blog/1695643348856.png)

## nginx部署

#### 简单实现

输入ip地址默认访问的就是`80`端口，所以我们只要将`80`端口下面的静态文件替换成我们打包好的文件的就可以了

#### 稍微正规做法

1.  在`/etc/nginx/conf.d`新建我们自己的`.conf`文件

```
    server{
        # 监听本地的3000端口
        listen 3000 default_server;
        listen [::]:3000 default_server;
            
            # 对应的打包文件目录
            root /usr/share/nginx/City;
            
            # history路由需要配置，不然刷新会404
            try_files  $uri $uri/ /index.html;

        # 反向代理，解决跨域
        location /api/ {
            proxy_pass  http://xx.xx.xxx.x:[port]/;
         }
    }
```
在将打包好的文件上传的对应的目录，我`.conf`文件写的` root /usr/share/nginx/City`这个目录

配置文件修改完成之后，运行下面三条指令

```bash
sudo /usr/sbin/nginx -t # 检查配置是否正确

sudo /usr/sbin/nginx  # 启动服务

sudo /usr/sbin/nginx -s reload # 重新载入配置
```

## nginx基础知识

#### 反向代理

我们之前做的工作，就是使用反向代理来解决跨域行为

#### 动静分离

nginx的并发能力公式：

worker\_processs\* worker\_connections / 4|2 = nginx最终的并发能力

动态资源需要/4，静态资源需要/2

nginx通过动静分离，来提升nginx的并发能力，更快的给用户响应

动态资源：
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c09e3ffb55d4b919a2a7034b57c9c07~tplv-k3u1fbpfcp-watermark.image?)

静态资源：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e724b335189047f8844b02430f82f24e~tplv-k3u1fbpfcp-watermark.image?)

**为什么需要动静分离：**

通过上图，我们明显可以看出，请求动态资源的时候，nginx需要做一层转发，去服务端拿取数据，再返回客户端。如果实现动静分离，就可以直接返回给客户端，没必要做转发，提升了响应速度和并发能力。

比如我有一个静态资源的图片，可以这样实现动静分离

```js
<img src='/images/saoqi.jpg'>
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64294417d4c84e3c9c27f22cb792efb9~tplv-k3u1fbpfcp-watermark.image?)

#### 负载均衡

nginx为我们提供了三种负载均衡的策略：

*   轮询：将客户端发起的请求，平均的分配给每一台服务器，默认策略

*   权重：会将客户端的请求，根据服务器的权重不同，分配不同的数量

*   ip\_hash：基于发起请求的客户端的ip地址不同，他始终会将请求发送到指定的服务器上。根据ip地址计算出一个结果，根据这个结果找到对应的服务器

# Docker

## 实践

Nginx手动部署需要我们本地执行`npm run build` 生成打包好的文件上传到服务器，编写Nginx配置文件。

下面以一个常用的Nginx配置文件为例

```bash
server{
    # 监听本地的3000端口
    listen 3000 default_server;
    listen [::]:3000 default_server;
        
        # 对应的打包文件目录
        root /usr/share/nginx/City;
        
        # history路由需要配置，不然刷新会404
        try_files  $uri $uri/ /index.html;

    # 反向代理，解决跨域
    location /api/ {
        proxy_pass  http://xx.xx.xxx.x:[port]/;
     }
}
```

在项目中需要实现nginx + docker部署，需要我们在工程目录下新建两个文件：`Dockerfile` 和 `default.conf`

`Dockerfile`是Docker根据Dockerfile生成Docker image用的，`default.conf`这个文件名字可以随便取，也可以叫`nginx.conf`，不过你需要和你下面编写的Dockerfile配置文件保持一致

接下来我们一起尝试编写一下这两个文件

**default.conf**

和上面一样，就不详细说了

```bash
server{
    listen 3000 default_server;
    listen [::]:3000 default_server;
        root /usr/share/nginx/City;
        try_files  $uri $uri/ /index.html;

    location /api/ {
        proxy_pass  http://47.92.112.6:8022/;
     }
}
```

**Dockerfile**

然后我们来编写一个脚本文件，模拟我们打包上传的过程

```bash

# 第一阶段
# 使用alpine镜像，可以减少构建后docker镜像文件的体积
FROM node:14-alpine

# 设置工作目录
WORKDIR /project

# 先copy package.json文件到工作目录，一般我们的项目中依赖是不会变的，这样可以充分利用缓存减少部署时的构建时间
COPY package*.json /project/

# 安装node_modules
RUN npm install

# 将所有文件copy到工作目录
COPY . /project

# 开始打包
RUN npm run build

# 第二阶段
# 拉取nginx镜像文件
FROM nginx
# 这里的dist文件就是打包好的文件，project是我们上面设置的工作目录
COPY --from=0 /project/dist /usr/share/nginx/City
# default.conf就是我们项目下面的nginx配置文件，我们需要copy到nginx的相应目录
COPY --from=0 /project/default.conf /etc/nginx/conf.d/City.conf
```

编写好这两个文件之后，我们只需要上传这个我们这个项目就好了，不需要单独打包上传。这里我将我的文件放到了这个目录下面

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e86a7dde6a94aab81d1f4e940baaeb9~tplv-k3u1fbpfcp-watermark.image?)

然后我们就可以根据Dockerfile生成docker image，再通过image 生成container，直接运行container做端口映射就可以了

```bash
# 镜像名设为front-project
docker build -t front-project .

# 运行容器，将本地的8081端口映射到docker的3000端口
docker run -itd -p 8081:3000 --name frontend-container front-project
```

然后我们直接访问ip+port就可以看到效果啦

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9efd2451a4f94e28ad20beb0d0a54e83~tplv-k3u1fbpfcp-watermark.image?)

注意：映射的3000端口可不能随便乱写哦，这是根据你的Nginx配置文件来得，我上面的`default.conf`监听的就是3000端口，所以写的3000

## 基础知识点

如果你看上面有点懵的话，不如先来了解一下Docker的基础知识点

### 基本命令

查看所有的容器

    docker ps -a

删除容器

    docker rm [containerID]

进入容器

    docker exec -it [containerId]  /bin/bash

生成镜像文件

    docker build -t [image-name] .

<!---->

启动容器

    docker run -itd -p 8081:80 --name [containerId] [image]

删除镜像前，要停掉所有与镜像相关的容器

**镜像迁移相关**

将已有的镜像保存为新的镜像

```js
docker commit mynginx mynginx_i
```

将镜像 mynginx\_i 镜像保存为 mynginx.tar 文件

```js
docker save -o mynginx.tar mynginx_i
```

将镜像保存为 .tar 文件后就可以将它放到其他服务器进行部署了，首先将其恢复；

    docker load -i mynginx.tar
    docker images

### Dockerfile多阶段构建

Dockerfile 文件，其内包含了一条条的指令，每一条指令构建一层，因此每一条指令的内容，就是描述该层如何构建

Docker镜像并不是由一个文件构成，而是由一堆文件构成，最主要的文件就是层，每一层构建完就不会再发生改变，后一层的改变只发生在自己的那一层，镜像层会被缓存和复用

镜像是由多层文件系统组成，想要优化它的大小，就需要去减少层数、每一层尽量只包含该层需要的东西，任何额外的东西应该在该层构建结束前清理掉，下面开始正文

镜像上传到Docker Hub， 体积小拉的化就会很快，相当于一个docker构建部署的一个优化

### 镜像优化

1.  使用node的Alpine版本,减少镜像文件体积
2.  减少层数，不经常变动的层移到前面去
3.  env设置多个环境变量，我们只需要dependencies的依赖，而devDependencies 依赖只是编译阶段用的，比如eslint,ts,也就是说最终镜像只需要我们需要的东西

**总结**：镜像构建的几个阶段
我把这个构建分成了三个阶段：

1.  第一阶段：构建基础镜像

    安装依赖、编译、运行等等阶段，就是所有阶段共用的东西都在第一阶段封到一个基础镜像里供其它阶段使用，比如设置环境变量、设置工作目录、安装 nodejs、yarn 等等

2.  第二阶段：安装依赖阶段

    在这个阶段，安装依赖，如果项目需要编译，可以在这个阶段安装依赖编译好

    `npm install`命令安装依赖比较耗时，大部分时我们只是改变代码不改变依赖，package.json可以缓存起来提高编译速度

    这里再说下装依赖的小细节，就是执行 `yarn --production` 加个 production 参数或者环境变量 `NODE_ENV` 为 `production`，yarn 将不会安装 devDependencies 中列出的任何软件包，[点我查看官方文档说明](https://link.juejin.cn?target=https%3A%2F%2Fclassic.yarnpkg.com%2Fen%2Fdocs%2Fcli%2Finstall%23toc-yarn-install-production-true-false "https://classic.yarnpkg.com/en/docs/cli/install#toc-yarn-install-production-true-false")

3.  第三阶段：最终使用镜像

    拷贝第二阶段安装的好的依赖文件夹，然后在拷贝代码文件到工作目录，执行启动命令，第二阶段装依赖多出的一些垃圾我们不需要，我们就只拷贝我们要用的东西，大大减少镜像的大小

    如果项目需要编译，在拷贝编译后的文件夹，不需要拷贝编译前的代码，有编译后的代码和依赖就可以跑起项目

多阶段构建，最后生成的镜像只能是最后一个阶段的结果，但是，能够将前置阶段中的文件拷贝到后边的阶段中，这就是多阶段构建的最大意义。

通过多阶段构建，充分利用缓存，可以将二次构建的速度提升到3.6秒

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/630d07da4c6a421bbef6744a14484099~tplv-k3u1fbpfcp-watermark.image?)

# jenkins

了解完Docker之后，我们就来学习一下怎么将这两者结合，最终实现自动部署

## 安装jenkins

jenkins安装也挺麻烦的，还涉及到Java版本等其他问题，这里我也是直接拉取jenkins镜像来安装jenkins

镜像文件地址：<https://hub.docker.com/r/jenkinsci/blueocean>

```bash
# 拉取镜像
docker pull jenkinsci/blueocean
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27a855ffb76b4bfaa1eab5d4ec905de7~tplv-k3u1fbpfcp-watermark.image?)

根据镜像运行容器，并进行数据映射

因为之后我们需要在jenkins中操作docker容器，生成镜像文件并运行容器，所以要做一下数据映射

在jenkins中执行docker有三种方式:

1.  Jenkins中安装docker客户端，使用第三方的docker 需要付费
2.  Jenkins中安装docker客户端，自己搭建docker，也就是 docker in docker， 比较麻烦，需要特权模式，或者使用第三方工具，比较复杂
3.  Jenkins中什么都不装，直接使用宿主机的docker服务

这里我们就是采用的第三种方案：

*   优势：简单，方便，直观

*   缺点：jenkins直接使用宿主机的docker服务，Jenkins可以全权的管理所有的容器(包括Jenkins自己)，有隐含的安全问题

```bash
# 运行容器
docker run -itd \
  -u root \
  --name jenkins_auto \
  -p 5002:8080 \
  -v /root/docker/jenkins:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkinsci/blueocean
```

然后我们进去jenkins容器里面，看看是不是能操作docker

```bash
# 进入容器
docker exec -it afbd2729c2af bash

# 是否有docker命令
docker --help
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ea56b8020314bd68a61ded43d8f5761~tplv-k3u1fbpfcp-watermark.image?)

我们输一下`docker images`，发现确实可以操作宿主机上的所有容器

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70e714b98b1e4e55bfc63ded11052c28~tplv-k3u1fbpfcp-watermark.image?)

## 运行Jenkins

第一次登陆需要密码，因为我们做了数据映射，所以直接到我们映射过的目录下面找就好了

    cat  root/docker/jenkins/secrets/initiaLAdminPassword

然后跟着引导创建一个新的用户就好，安装推荐的创建，失败了也不要紧，到时候需要啥插件我们自己装就好

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/431e666f7e934c06bbf96b5034b387cd~tplv-k3u1fbpfcp-watermark.image?)

## 安装插件

因为jenkins版本和插件版本的原因，安装了好几次插件都报错，最后选择了直接下载到本地然后上传插件的方式

插件下载地址：[Index of /download/plugins (jenkins-ci.org)](https://updates.jenkins-ci.org/download/plugins/)

安装git插件

点击插件管理 -> 高级

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81d4eaad76a54287af9e4f3fb055538d~tplv-k3u1fbpfcp-watermark.image?)

上传我们之前下载到本地的git插件
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc062247f7db46599fe4bf706eaf1b70~tplv-k3u1fbpfcp-watermark.image?)

## 新建任务

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63863246fc46466fb13d18fdc37f92a0~tplv-k3u1fbpfcp-watermark.image?)

点击配置

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8261789082d4619b107372638d37021~tplv-k3u1fbpfcp-watermark.image?)

输入我们的仓库地址和账号密码(点击添加，可以新增一个账号)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35f9e6c7fcbc406fbabf43324ed4a78b~tplv-k3u1fbpfcp-watermark.image?)

构建的时候选择执行shell

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3efd26a763f4c66bcf89999cd3a1b34~tplv-k3u1fbpfcp-watermark.image?)

输入我们的shell脚本

```bash
# 先删除之前的容器和镜像文件
if [ "$(docker ps -a | grep jenkins-front-city)" ]; then
    docker stop jenkins-front-city
    docker rm jenkins-front-city
fi
if [ "$(docker images -q jenkins-front-city)" ]; then
    docker rmi jenkins-front-city
fi

# 重新生成
docker build -t jenkins-front-city .
docker run -itd -p 8866:3000 --name jenkins-front-city jenkins-front-city
```

最后我们点击保存就大功告成了！！！点击构建之后就能看到最后效果了！！！

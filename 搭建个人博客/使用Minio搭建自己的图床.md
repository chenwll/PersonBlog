# 使用Minio搭建自己的图床

最近自己想搭建一下自己的个人博客，然后发布到各个平台上面，包括自己的私欲流量，掘金，微信公众号等平台。目前的一个想法是先Typora本地写好文档，然后git上存一份，并且通过脚本统一发布到各个平台。

我们先来实现第一步，搭建一个个人的图床，这样就不会有各个平台的水印了

实验条件：

1. 一台个人服务器
2. Typora
3. 了解docker,nginx的使用

# 创建Minio

1. 通过docker直接安装Minio

```bash
docker pull minio/minio
```

2. 创建目录

我们创建了两个目录，用类做数据映射

```bash
# 存储文件位置
mkdir -p /usr/local/minio/data

# 配置文件位置
mkdir -p /usr/local/minio/config
```

3. 启动容器 运行Minio

在做数据映射的时候，记得映射一下登录的账号和密码，比如我们将用户名设置为admin，密码为123456

```bash
docker run \ 
 --name minio \  #docker 镜像名称
  -p 9000:9000  \ #服务端口号
  -p 9001:9001  \ #映射端口号
  -d --restart=always \ #docker设置容器随系统开机启动 minio
  -e "MINIO_ACCESS_KEY=admin"  \ #登录用户名
  -e "MINIO_SECRET_KEY=123456"  \ #登录密码
  -v "/usr/local/minio/data":"/data" \  # 存储文件位置
  -v "/usr/local/minio/config":"/root/.minio"  \ # 配置文件位置
  minio/minio server /data --console-address ":9001"  \  #启动服务对外端口号 访问主机ip+9001 就能打开
```

**注意**：新版本的Minio需要做两个端口映射，分别是`控制台端口号` 和 `api端口号` 

控制台端口号是指当我们输入`ip + 端口`的时候，可以访问Minio的控制台，这个端口号被成为`控制台端口号`

![image-20230915234047161](http://120.53.221.17/blog/1694792447625.png)

`api端口号`好像现在java连的话必须使用`api端口号`

4. 查看容器启动状态

```bash
docker ps

root@VM-16-6-ubuntu:~# docker ps
CONTAINER ID   IMAGE   COMMAND     CREATED       STATUS       PORTS      NAMES
9a0e09291de0   minio/minio   "/usr/bin/docker-ent…"   3 hours ago   Up 3 hours   0.0.0.0:9000-9001->9000-9001/tcp, :::9000-9001->9000-9001/tcp   minio
```



# 上传图片到Minio

使用我们做数据映射时的账号密码，就可以登录进去，然后创建Buckets，记得要将`Access Policy`设置成为`Public`，这样才能通过url访问图片

![image-20230915234921304](http://120.53.221.17/blog/1694792961673.png)

输入`ip + api端口 + 桶名 + 文件名` 即可访问图片

![image-20230915235235715](http://120.53.221.17/blog/1694793156453.png)



# Typora + Minio

Minio保存图片已经没有问题了，我们现在开始将Typora和Minio结合起来，当我们插入图片的时候就上传到服务器上面，我们通过js脚本来实现，因为自己对js比较熟悉一点

1. 本地创建项目

   ```js
   mkdir Typora-Minio
   cd Typora-Minio
   npm init
   ```

2. 安装`minio`包

   ```js
   npm install minio
   ```

3. 写js脚本，新建一个`upload.js`，注意替换注释的部分

   ```js
   /* 
    * typora插入图片调用此脚本，上传图片到图床
    */
   
   const path = require('path')
   // minio for node.js
   const Minio = require('minio') 
   
   
   // 解析参数， 获取图片的路径，有可能是多张图片
   const parseArgv = () => {
     const imageList = process.argv.slice(2).map(u => path.resolve(u))
     console.log(imageList)
     return imageList
   }
   
   // 入口
   const uploadImageFile = async (imageList = []) => {
     // 创建连接
     const minioClient = new Minio.Client({
       // 这里填写你的minio后台域名
       endPoint: 'x.x.x.x',
       // api端口号
       port: 9000,
       useSSL: false,
       // 下面填写你的accessKey和secretKey
       accessKey: 'xxxxxxxxx',
       secretKey: 'xxxxxxxxxxxxxxxxxxx'
     })
   
     // 开始上传图片
     const metaData = {}
     const tasks = imageList.map(image => {
       return new Promise(async (resolve, reject) => {
         try {
           // 图片重命名，这里采用最简单的，可以根据自己需求重新实现
           const name = `${Date.now()}${path.extname(image)}`
           // 具体请看Minio的API文档，这里是将图片上传到blog这个bucket上
           const res = await minioClient.fPutObject('blog', name, image, metaData)
           resolve(name)
         } catch (err) {
           console.log(err)
           reject(err)
         }
       })
     })
   
     const result = await Promise.all(tasks)
     
     // 返回图片的访问链接
     result.forEach(name => {
       // const url = `http://ip+api端口号/桶名/${name}`
         const url = `http://x.x.x.x:9000/blog/${name}`
       // Typora会提取脚本的输出作为地址，将markdown上图片链接替换掉
       console.log(url)
     })
   }
   
   // 执行脚本
   uploadImageFile(parseArgv())
   ```

4. Typora插入图片

   `文件` —> `偏好设置`  —> `图像`，设置上传图片时执行js脚本，可以点击旁边的验证图片上传选择一下，无问题的话就大功告成

   ![image-20230916000518837](http://120.53.221.17/blog/1694793919096.png)


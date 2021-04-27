---
layout:     post
title:      docker入门3：使用docker创建开发部署环境
subtitle:   docker,运维,虚拟化技术
date:       2019-11-26
author:     Chris
header-img: img/post-bg-5g.jpeg
catalog: true
tags:
    - docker
    - 运维
    - 虚拟机
    - 虚拟化技术
---



之前介绍过docker的基本用法，包括镜像的CRUD操作和容器的CRUD操作。这里，介绍使用docker来安装具体的开发/构建环境。

# 1 为什么使用docker
前面介绍过，docker为我们创造了一个具体的开发/部署环境，从而，容易做到开发/线下/线上环境的一致性。这是在开发和运维上的优点。   
对于开发者个人平时的学习而言，使用docker有如下优点   
* 当软件开发/部署环境安装比较困难时，也可以考虑使用docker。因为，docker仓库里面有很多现成的镜像可以使用，不需要我再一个个安装软件以及依赖了。   
* docker可以提供隔离的环境，从而不会造成各种版本的软件相互冲突    

下面，我们就以几个例子而言，讲讲docker怎么样来安装开发/部署环境。

# 2 安装python3.5开发环境
本地mac上默认带版本为python 2.7。想要体验python 3.5。那么，可以通过如下步骤来实现。   
1. 去docker hub，搜索其他人上传的镜像   
    链接如下：https://hub.docker.com/search?q=python&type=image   
    <img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g9bq1360z5j31ca0go3yw.jpg">   
    找到上图中的链接，就是拉取python 3.5镜像的链接。
2. 在本地拉取镜像   
    ```
    > docker pull python:3.5
    ```
3. 在本地创建一个hello.py文件，内容如下   
    ```
    print("Hello, World!");
    ```
4. 启动容器并运行脚本hello.py。关于参数的解释见前一篇博客。
    ```
    > docker run  -it  --mount type=bind,src=$PWD,target=/usr/src/myapp  -w /usr/src/myapp python:3.5 python hello.py #hello.py需要位于当前执行命令的路径下面
    ```
    执行之后，控制台会打印出"Hello, World!"   
    ```    
    > docker run  -it  --mount type=bind,src=$PWD,target=/usr/src/myapp  -w /usr/src/myapp python:3.5 python hello.py
    Hello, World!
    ```

# 3 安装jekyll 3.0 静态博客框架
该博客是通过jekyll框架来支持的。之前使用的是jekyll 3.0版本，而安装3.0版本已经过时了，需要低版本的ruby支持，在本地安装很麻烦，故想到通过docker环境来运行。   
1. 去docker hub，搜索其他人上传的镜像   
    链接如下：https://hub.docker.com/r/jekyll/jekyll   
    <img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g9cu3afqcgj321n0inaah.jpg">   
    找到上图中的链接，然后拉取jekyll 3.0版本。
2. 拉取镜像   
    ```
    > docker pull jekyll/jekyll:3.0
    ```
3. 启动容器并文件夹里面的项目   
    进入你的静态博客所在路径，就是配置文件_config.yml所在的那个路径，执行如下命令   
    ```
    > docker run --mount type=bind,source=$(pwd),target=/srv/jekyll -p 4000:4000 --name blog -it jekyll/jekyll:3.0 jekyll serve
    ```
    执行之后，控制台会打印出
    ```
    Configuration file: /srv/jekyll/_config.yml
                Source: /srv/jekyll
        Destination: /srv/jekyll/_site
    Incremental build: disabled. Enable with --incremental
        Generating... 
                        done in 2.579 seconds.
    Auto-regeneration: enabled for '/srv/jekyll'
    Configuration file: /srv/jekyll/_config.yml
        Server address: http://0.0.0.0:4000/
    Server running... press ctrl-c to stop.
    ```
4. 打开http://0.0.0.0:4000/   
    接下来，我们在浏览器打开http://0.0.0.0:4000/这个网址，就可以在本地看到博客了。

# 4 安装nginx环境

1. 在本地拉取最新镜像   
    ```
    > docker pull nginx:latest
    ```
2. 启动容器并运行   
    ```
    > docker run -d --name my-nginx -p 8080:80 nginx
    ```
    执行之后，在本地浏览器打开http://127.0.0.1:8080，就可以看到nginx已经启动了。   
    <img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g9cvan8qchj30ya0d8dg9.jpg">


# 5 参考
[用 Docker 运行 Jekyll](https://livid.v2ex.com/essays/2018/12/31/jekyll-docker.html)
[docker入门2：docker run参数学习](https://goldbrown.github.io/2019/11/26/docker%E5%85%A5%E9%97%A82-docker-run%E5%8F%82%E6%95%B0%E5%AD%A6%E4%B9%A0/)

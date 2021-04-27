---
layout:     post
title:      docker入门2：docker run参数学习
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


docker run是启动容器的命令，参数很丰富，这里特意学习一下。

一般启动容器的常用命令为   
```
> docker run -d -p [主机端口]:[容器端口] --name [容器名称] [镜像名称]:[tag]
```
`-d`：`--detach`的简写，表示容器在后台运行，打印出容器id   
`-p`：`--publish`的简写，表示容器和主机之间的端口映射。   
`--name`：赋予容器一个名字。不然，容器就随机取一个名字。

在实际的使用场景中，常用的参数还有一些。

# 1 常用命令
* `-i`  
    `-i`等同于`--interactive=true`，表示打开STDIN，用于控制台交互。默认`--interactive=false`。

* `-t`   
    `-t`等同于`--tty=true`，表示分配tty设备，可以支持终端登录。默认`--tty=false`。
    下面的命令，拉取ubuntu 16.04版本的镜像，并启动容器，打开一个可以交互的终端。
    ```
    > docker pull ubuntu:16.04
    > docker run -it --name ubuntu123 ubuntu:16.04
    ```
    打开的终端如下图所示   
    <img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g9btkyrccgj316o03874a.jpg">

* `-u`    
    等同于`--user`。指定容器的用户。   
    下面的例子，使用当前登录的用户的uid(user id)和gid(group id)。可以看到，当前进入容器的用户，已经不是root用户了。因为我们没有指定名字，所以没有用户名。实际上，也不需要让容器依赖于用户名。    
    ```
    > docker run --user $(id -u):$(id -g) -it --rm ubuntu:16.04 /bin/bash
    I have no name!@2cd7f85141a6:/$ 
    ```

* `-w`   
    也可以写为`--workdir`。指定容器的工作目录。当指定的目录不存在时，则在容器内创建一个目录。  
    下面的例子，指定工作目录为/work/here。 
    ```
    > docker run -w /work/here -it ubuntu:16.04 pwd
    /work/here
    ```

* `-e`  
    也可以写为`--env`，指定环境变量，容器中可以使用该环境变量。如下面的例子所示。   
    ```
    > docker run --env VAR1=value1 --env VAR2=value2 ubuntu:16.04 env | grep VAR
    ```
    输出如下   
    ```
    VAR1=value1
    VAR2=value2
    ```

* `-m`  
    等同于`--memory`，指定容器可以使用的内存上限。  
    例子如下。
    1. 在一个终端中使用如下命令启动一个运行中的容器       
    ```
    > docker run -it -m 1GB ubuntu:16.04 /bin/bash
    ```
    2. 在另一个终端中，使用如下命令查看容器的资源使用情况。可以看到内存使用为792K，上限为1G。    
    ```
    > docker stats
    CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
    7170f4b198d8        trusting_mclaren    0.00%               792KiB / 1GiB       0.08%               1.11kB / 0B         0B / 0B             1
    ``` 

* `-h`  
    等同于`--hostname`，指定容器的主机名。
    下面的例子中，可以看到启动的容器的主机名为mydocker。      
    ```
    > docker run -it -h mydocker ubuntu:16.04 /bin/bash
    root@mydocker:/#
    ```

* `-v`  
    `-v`等同于`--volume`，含义是给容器加载一个用于存储的volume。已经不推荐使用，推荐使用`--mount`参数。

* `--mount`   
    用于给容器加载一个volume（卷）、主机文件路径或者tmpfs。`--mount`的值包含多个key-value对，使用逗号分隔，各个key-value对无需考虑顺序。可以选择的key如下所示   
    * type：可以为bind、volume和tmpfs，通常为volume。type=volume是指加载一个命名卷或者匿名卷，具有很多优点，是推荐的使用方式。type=bind指挂载主机的某个文件路径到容器中，主机上被挂载的这个路径是被共享的。type=tmpfs是指数据仅仅写入主机系统的内存中，不会写入主机的文件系统。   
    * source：简写为src。type=volume，为挂载的卷名。type=bind，为挂载的文件路径。
    * target（或者dst，或者destination）：容器中对应的路径   
    * readonly：可选，表示是否只读。若开启，则表示该路径只读。   
    * volume-opt：可选。为一个key-value对，用来指定额外的参数。可以多次使用。   

    例子1：下面的例子，首先创建一个卷，然后启动ubuntu时挂载该卷。   
    ```
    > docker volume create myvol2 #创建一个卷
    > docker volume ls #列出所有的卷
    > docker volume inspect myvol2 #查询某个卷
    > docker run -it  --mount source=myvol2,target=/app2 ubuntu:16.04 /bin/bash
    ```
    通过以下命令，在ubuntu里面查看挂载的卷。可以看到有一个路径为`/app2`。   
    ```
    > df
    Filesystem     1K-blocks    Used Available Use% Mounted on
    overlay         61255492 6927592  51186576  12% /
    tmpfs              65536       0     65536   0% /dev
    tmpfs            1023420       0   1023420   0% /sys/fs/cgroup
    shm                65536       0     65536   0% /dev/shm
    /dev/sda1       61255492 6927592  51186576  12% /app2
    tmpfs            1023420       0   1023420   0% /proc/acpi
    tmpfs            1023420       0   1023420   0% /sys/firmware
    ```

    例子2：下面的命令将当前主机的路径映射到容器的路径/mydir/。   
    ```
    > docker run -it  --mount type=bind,src=$PWD,target=/mydir/ ubuntu:16.04 /bin/bash #$PWD代表执行命令的当前路径
    > ls
    bin  boot  dev  etc  home  lib  lib64  media  mnt  mydir  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
    > ls /mydir/ #/mydir映射到了主机的当前路径，这里为我的家目录。
    Applications  CorpCode  Desktop  Documents  Downloads  Library  Movies  Music  Pictures  Public  SelfCode  UserApplications  gems  jekyll_site
    ```
    这里只是讲述了基本的使用，详情请参考[docker volume 容器卷的那些事（一）](https://www.jianshu.com/p/dd2fecfd8edf)和[Use volumes](https://docs.docker.com/v17.12/storage/volumes/)。

* `--rm`  
    等同于`--rm=true`，表示容器停止后自动删除容器(不支持以docker run -d启动的容器)   
    下面的命令，当容器停止后，容器就会被删除了。   
    ```
    > docker run --rm -it ubuntu:16.04 /bin/bash
    ```
    在容器停止后，可以使用`docker ps`来查看容器，会发现找不到刚才启动的容器。  


# 2 参考
[Docker run 命令参数及使用](https://www.jianshu.com/p/ea4a00c6c21c)   
[docker volumes 中 -v 和 -mount 区别](http://einverne.github.io/post/2018/03/docker-v-and-mount.html)    
[docker run](https://docs.docker.com/v17.12/engine/reference/commandline/run/#options)    
[docker volume 容器卷的那些事（一）](https://www.jianshu.com/p/dd2fecfd8edf)    
[Use volumes](https://docs.docker.com/v17.12/storage/volumes/)   
[Running a Docker container as a non-root user](https://medium.com/redbubble/running-a-docker-container-as-a-non-root-user-7d2e00f8ee15)

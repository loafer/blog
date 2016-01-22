Docker 基本命令
===
说明：所有命令前的`$`为shell提示符
##信息类
####版本信息
```shell
$docker version
```

####系统信息（docker宿主信息）
```shell
$docker info
```

##镜像类
####检索 image（在Docker Hub上检索）
```shell
#从Docker　Hub上检索有关ubuntu的镜像
$docker search ubuntu
```

####下载 image（从Docker Registry默认Docker Hub）
```shell
#从Docker Hub上拉取busybox镜像
$docker pull busybox
```

####显示本地 image 列表
```shell
#其他参数请使用｀docker images --help｀查看
$docker images
```

####删除 image
```shell
#删除本地镜像busybox和ubuntu
$docker rmi busybox ubuntu
```

####显示 image 历史
```shell
#显示ubuntu镜像历史
$docker history ubuntu
```

##容器类
容器是一个打包了应用和服务的环境，它是个轻量级的虚拟机。

####创建容器
##### docker create
`docker create`创建一个停止状态的容器，容器成功创建后，会返回容器的ID。每个ID是独一无二

```shell
#其他参数请使用｀docker create --help｀查看
#使用ubuntu镜像创建了一个容器，容器名称随机
$docker create ubuntu　　　
　　

#使用ubuntu镜像创建了一个名为`myubuntu`的容器，但为启动。
$docker create --name=myubuntu ubuntu
```


#####docker run
`docker run` 创建立马运行的容器。使用`docker run`可以创建两种类型的容器
- 交互型容器：运行在前台，通常会指定有交互的控制台，可以给容器输入，也可以得到容器的输出。创建容器的终端被关闭，在容器内部使用`exit`命令或调用了`docker stop`、`docker kill`命令后，容器会变成停止状态。

```shell
# -i 打开容器标准输入，　-t 建立一个命令行终端
$docker run -i -t --name=inspect_shell ubuntu /bin/bash
```

- 后台型容器：运行在后台，创建启动之后就与终端无关。即使终端关闭了，改后台容器依然存在，只有调用`docker stop`或`docker kill`命令才能是容器变为停止状态。在实际的应用中，大多数容器都是后台型容器，服务程序不可能因为创建容器的终端退出而停止。

```shell
$docker run -name=deamon_while -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```


####查看容器
列出当前运行的容器
```shell
$docker ps
```

  + CONTAINERID:唯一标示容器的ID
  + IMAGE: 创建容器时使用的镜像
  + COMMAND: 容器最后运行的命令
  + CREATED: 容器创建的时间
  + STATUS: 容器的状态。如果是运行状态，类似`UP 49 minutes`；如果是停止状态，类似`Exited(0)`。其中数字０是退出时的错误代码，０为正常退出。
  + PORTS:　对外开放的端口
  + NAMES: 容器名。和ID一样可以唯一标识一个容器，所以同一台宿主主机上不能有同名容器存在。


列出所有容器
```shell
$docker ps -a
```

列出最近创建的容器
```shell
$docker ps -l
```

列出最近创建的x个容器
```shell
#列出最近创建的２个容器
$docker ps -n=2
```

####启动容器
当容器运行完自己的任务后，容器会退出，进入停止状态。如果需要再次启动该容器，可以执行`docker start`命令。

```shell
$docker start deamon_while
```

####停止容器
```shell
#使用名称停止容器
$docker stop deamon_while

#使用ID停止容器
$docker stop 9434d74f2b06
```

####退出容器
默认情况下,如果使用`ctrl+c` 退出容器，那么容器也会停止．
按`ctrl+p`或`ctrl+q`可以退出到宿主机，而保持容器仍在运行．

####删除容器
```shell
#使用名称删除容器
$docker rm inspect_shell


#使用ID删除容器
$docker rm 3caf17676478
```

####容器重命名
```shell
$docker rename oldname newname
```

####依附容器
对于一个正在运行的`交互型容器`可以使用`docker attach`命令将终端依附到容器上（即:回到容器的终端上）

```shell
#将终端依附到名称为"inspect_shell"容器的终端上
$docker attach inspect_shell
```

####容器日志
使用`docker logs` 命令查看后台型容器在做什么。

```shell
$docker logs deamon_while
```

使用`-f`参数来跟踪日志输出，这与`tail -f`非常相似。此时日志被输出到当前控制台。注意：请不要使用`ctrl+c`，会引起当前进程终止。

```shell
$docker logs -f deamon_while
```

####创建容器内进程
使用`docker exec`可以不必依附容器，而直接创建容器内的进程。

```shell
$docker exec -d deamon_while mkdir -p /tmp/hello
```

直接创建一个可交互的进程，对于那些docker一起动就执行服务的容器，适合使用此条命令访问容器。而不是使用`docker attach`访问容器后，看到的只是服务后台日志。

```shell
$docker exec -ti inspect_shell /bin/bash
```


####使用Dockerfile构建镜像
```shell
$docker build -t java:8 .
```

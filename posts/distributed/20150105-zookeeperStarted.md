Zookeeper Getting Started Guide
==

首先，在这里[下载](http://zookeeper.apache.org/releases.html)Zookeeper稳定版，然后将其解压到单独的文件夹中，此处解压后的文件夹名为zookeeper。

>后面的操作都是在zookeeper目录中执行。



##单机模式
单机模式启动Zookeeper Server很简单，Server仅仅是个jar文件。
启动Zookeeper Server时你需要一个配置文件，你可以从conf目录下拷贝一份zoo_sample.cfg将其命名为zoo.cfg。或者自己动手在conf目录下自己创建一个文件名为zoo.cfg的文件，在这个文件中写入如下内容：

```
tickTime=2000
dataDir=/home/zhaojh/zookeeper/data
clientPort=2181
```

以上为zookeeper Server启动的最小配置。

**tickTime**
 Zookeper的基本时间单位（毫秒），被用于心跳检测和session超时（两倍的tickTime）

**dataDir**
指定内存数据库快照的本地存储位置。

**clientPort**
客户端连接监听端口

通过以下命令启动Zookeeper Server:

```shell
bin/zkServer.sh start
```

Zookeeper使用log4j管理日志信息。关于日志的详细内容请查看[编程向导](http://zookeeper.apache.org/doc/r3.4.6/zookeeperProgrammers.html#Logging)。

在单机模式下，当Zookeeper处理任务失败，则整个Server将会停止。因此，单机模式仅适用于开发环境使用。


##使用客户端访问 Zookeeper

执行以下命令访问Zookeeper Server
```
bin/zkCli.sh -server 127.0.0.1:2181
```

一旦连接成功你将会在客户端看到日志输出，最后光标停留在
```
[zk: 127.0.0.1:2181(CONNECTED) 0]
```

键入`help`你将会在控制台得到一个命令列表
```
[zk: 127.0.0.1:2181(CONNECTED) 0] help
       ZooKeeper -server host:port cmd args
	connect host:port
	get path [watch]
	ls path [watch]
	set path data [version]
	rmr path
	delquota [-n|-b] path
	quit
	printwatches on|off
	create [-s] [-e] path data acl
	stat path [watch]
	close
	ls2 path [watch]
	history
	listquota path
	setAcl path acl
	getAcl path
	sync path
	redo cmdno
	addauth scheme auth
	delete path [version]
	setquota -n|-b val path
[zk: 127.0.0.1:2181(CONNECTED) 1]
```

首先使用下`ls`命令，在终端敲入`ls /`如下：
```
[zk: 127.0.0.1:2181(CONNECTED) 3] ls /
[zookeeper]
```

接下来，输入`create /zk_test mydata`。这将会创建一个新的znode节点，此节点上绑定的数据是`mydata`。
```
create /zk_test mydata
Created /zk_test
```

然后，再输入`ls /`
```
[zk: 127.0.0.1:2181(CONNECTED) 4] ls /
[zookeeper, zk_test]
```

注意这时你就能结果中看到刚才创建的`zk_test`。

接下来，通过`get`命令获取`zk_test`节点上的数据。注意zk_test前的`/`
```
[zk: 127.0.0.1:2181(CONNECTED) 1] get /zk_test
mydata
cZxid = 0x2
ctime = Tue Jan 06 13:53:15 CST 2015
mZxid = 0x2
mtime = Tue Jan 06 13:53:15 CST 2015
pZxid = 0x2
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```

通过`set`命令，可以修改`zk_test`节点中的数据。
```
[zk: 127.0.0.1:2181(CONNECTED) 2] set /zk_test junk
cZxid = 0x2
ctime = Tue Jan 06 13:53:15 CST 2015
mZxid = 0x5
mtime = Tue Jan 06 14:15:06 CST 2015
pZxid = 0x2
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0
[zk: 127.0.0.1:2181(CONNECTED) 3] get /zk_test
junk
cZxid = 0x2
ctime = Tue Jan 06 13:53:15 CST 2015
mZxid = 0x5
mtime = Tue Jan 06 14:15:06 CST 2015
pZxid = 0x2
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0
```

最后，使用`delete`命令删除`zk_test`节点
```
[zk: 127.0.0.1:2181(CONNECTED) 4] delete /zk_test
[zk: 127.0.0.1:2181(CONNECTED) 5] get /zk_test
Node does not exist: /zk_test
[zk: 127.0.0.1:2181(CONNECTED) 6] ls /
[zookeeper]
[zk: 127.0.0.1:2181(CONNECTED) 7]
```

##集群模式
对于评估、开发、测试来说zookeeper的单机模式很方便。但是，在生产环境下你应当将zookeeper运行在集群模式下。集群模式下zookeeper的配置与单机模式类似，但是又有一点点的不同。这里有个列子。
```
tickTime=2000
dataDir=/home/zhaojh/zookeeper-1/data
clientPort=2181
initLimit=5
syncLimit=2
server.1=127.0.0.1:2887:3887
server.2=127.0.0.1:2888:3888
server.3=127.0.0.1:2889:3889
```

**initLimit** 用来限制follower（follower为集群中除leader之外的其他zookepper机器）连接到leader的最大时间长度。如果超过这个时间，则说明follower连接leader失败。

**syncLimit**  leader与follower之间发送请求和应答的时间长度。如果leader在syncLimit时间后没有获得follower的应答，则认为此follower已下线。

以上两个超时设置都是以tickTime为基本单位。initLimit=5也就是总时间长度为5*2000（毫秒）=10000（毫秒），也就是说最大时间长度为10秒。

**server.X** 用来指定由哪些机器组成集群来提供zookeeper服务。当服务启动时，将会在dataDir属性指定的目录下查找`myid`文件。这是一个文本文件，保存了服务的编号即`X`的值。如`server.1`，myid的内容就是`1`。

最后，注意每个server 名称有两个端口号：“2888”和“3888”。前一个端口是集群leader与follower之间进行通讯用的，第二个端口是用来在集群中选举leader用的。

>注意：如果你要在一台物理机上构建多个zookeepr服务，可以使用localhost作为服务名，然后区分开各服务的通讯端口（例如：2888:3888、2889:3889、2890:3890）。同时dataDir和clientPort也不能一样。

##总结
- 单机模式，安装简单，适用与开发和测试。
- 集群模式，适用与生产环境。各机器的zookeeper配置内容相同，可以共享同一份配置文件。同时不要忘了在dataDir属性所指定的目录下放入一个`myid`文本文件，其内容与`server.X`中`X`值相同。
- 伪集群模式，在配置时需要注意各zookeeper实例在dataDir、clientPoint，和server.X属性配置上不能相同。

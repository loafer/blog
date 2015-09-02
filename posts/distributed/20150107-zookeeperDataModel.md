Zookeeper——数据模型
==

Zookeeper提供了一个类似文件系统目录树方式的数据存储。它使用绝对路径的方式来表示节点路径，任何unicode字符都能用在节点路径上，但要注意下面这些约定：

- null字符(\u0000)不能出现在路径中。
- 以下字符不能使用，因为显示有问题： \u0001 - \u0019 和 \u007F - \u009F
- 下列字符不被允许： \ud800 -uF8FFF, \uFFF0-uFFFF, \uXFFFE - \uXFFFF (`X`是数字 1 - E), \uF0000 - \uFFFFF。
- `.`可以作为名称的一部分，但是`.`和`..`不能用来表示路径，因为Zookeeper不使用相对路径。下面的路径是无效的：`“/a/b/./c”`或者`"/a/b/../c"`。
- `zookepper`是保留字。

##Znodes
Zookeeper树中的每个node被称作一个`znode`。Znodes维护了一个状态结构，用来监控数据和访问控制的变更。这个状态结构也有一个时间戳。Zookeeper通过版本号和时间戳来验证缓存和协调数据更新。客户端每次执行一个更新或删除操作时，都需要提供被操作数据的版本号。如果提供的版本号不能与数据实际的版本号相匹配，那么操作失败（当然这种行为是可以通过设置来改变的）。

> 注意：在分布式系统同`node`一词可以表示一台主机、一个服务、一个整体中的一个点、一个客户端处理等。在Zookeeper中，`znodes`表示一个数据节点。`Servers`表示一组组成Zookeeper服务的机器；那些使用Zookeeper服务的主机或程序成之为`client`。

Znodes是编程访问的主要对象，它有很多特性，下面将逐一介绍。

**Watches**
客户端可以在`znodes`上设置`watches`。当znode发生变化时将出发watch，然后再清除掉这个watch。当watch被触发时，Zookeeper会向client发送一条通知。

**Data Access**
存储在znode上数据的读或写都是原子操作。一个读操作将获得与该znode相关的所有数据，一个写操作将替换改znode上的所有数据。每个znode上都有一个访问控制列表（ACL），用来决定谁可以对数据做哪些操作。

Zookeeper没有被设计成一个通用数据库或大对象的存储。相反，它仅仅是用来协调数据管理。这些数据可以是配置信息，状态信息，rendezvous等。这些数据有个相同的特点`相对较小`：几千字节。Zookeeper的client和server都会检查并控制被存储的数据`不应大于1M`。当操作非常大的数据时，在通过网络在各个存储介质间移动数据是很耗时的。如果确实需要大数据存储，通常处理方式是将数据存储在如`NFS`或`HDFS`这样的存储系统上，然后在zookeeper上存储这些数据的位置。

**Ephemeral Nodes**
Zookeeper也有临时节点的概念。当session被创建时，这些znodes为激活状态，当session失效后它们就会被删除。在这些节点上不允许存在子节点。   

**Sequence Nodes -- Unique Naming**
在创建节点时，可以要求Zookeeper在路径后面追加一个编号。这个编号在同一个父节点下是唯一的，不会重复。编号以`%010d`形式显示——即10个数字，使用0来填充（这种格式方便排序）。如"<path>0000000001"。注意，需要是一个由父节点维护有符号整数，其有效范围不能超过`2147483647`（否则返回-2147483647）。

**Zookeeper中的时间**

Zookeeper有多种跟踪时间的方式：   

-  **Zxid**
当每次Zookeeper的状态发生改变是，都会以zxid的形式收到一个时间戳。这个时间戳表示了每次变更的顺序。zxid是唯一的，不会出现重复。如果zxid1小于zxid2则表示zxid1发生在zxid2之前。

- **version numbers**
每一次变更都会引起node版本号的递增。node上有三类版本号，分别是`version`（数据的变更号）、`cversion`（子节点变更号）、`aversion`（节点ACL变更号）。

- **Ticks**
在Zookeeper的集群模式下用ticks来度量status uploads、session timeouts、connection timeouts等事件的时间。tick time还间接反应了session最小超时时间（2个tick time），如果一个客户端请求会话时间小于最小超时时间，则服务器会告诉客户端会话超时。

- **Real time**
除了在创建和修改znode时会将时间戳放入到状态结构外，Zookeeper不会使用真实时间或时钟时间。

**Zookeeper的状态结构**
每个znode的状态结构由下列字段组成：

- **czxid**
在znode被创建时更新
- **mzxid**
每次修改znode时会更新
- **ctime**
znode被创建的时间（毫秒，从epoch开始的时间，即1970-1-1开始，Unix系统常用的时间戳表示方式）
- **mtime**
此znode最近一次被修改的时间（同ctime记录方式一样）
- **version**
此znode的数据被修改的次数
- **cversion**
此znode子节点被修改的次数
- **aversion**
此znode的ACL被修改的次数
- **ephemeralOwner**
此znode如果是临时节点，改值表示此节点拥有者的session id。如果不是临时节点，此值为0
- **dataLength**
此znode上数据的长度
- **numChildren**
此znode所拥有的子节点数
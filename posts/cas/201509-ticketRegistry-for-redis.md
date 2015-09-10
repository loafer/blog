#cas4.0——基于Redis的Ticket存储

CAS默认ticket存储方式是基于内存的。这个对于单节点的cas-server发布是没有问题。假如采用集群方式部署cas-server，为了使各节点共享ticket，就必须将其存放在一个公共的存储介质中，这里选择Redis作为公共存储介质。

Redis是一个高性能的内存数据库，通常被当做**数据结构服务器**来使用。它与Memcached的主要区别,在于它支持更多的数据存储结构。官方只提供了对Memcached的支持，并没有提供对Redis的支持。不过这不是问题，我们可以参考`cas-server-integration-memcached`完成对Redis的支持。具体代码请参考我的[cas-server-integration-redis]()。

##依赖
```xml
<dependency>
  <groupId>com.github.loafer</groupId>
  <artifactId>cas-server-integration-redis</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
```

##配置
```xml
<bean id="ticketRegistry"
  class="com.github.loafer.cas.ticket.registry.RedisTicketRegistry"
  c:hostname="localhost"
  c:ticketGrantingTicketTimeOut="36000"
  c:serviceTicketTimeOut="2"/>
```

* `ticketGrantingTicketTimeOut`和`serviceTicketTimeOut`的时间单位是**秒**。
* `ticketRegistry`使用的是JDK序列化方式。
* 使用`Jedis`实现与Redis的通讯。

##为什么选择Redis作为ticket存储介质？
虽然官方的首选是Memcached，但是Memcached主要是部署在类Unix系统上。Memcached官方并没有提供windows版本，尽管网上有个Windows版本但它是基于老版本的Memcached实现。不像Redis，虽然官方也没有提供windows版本，但是微软有自己单独的团队在维护windows版本与官网的同步，因此考虑实现对Redis的支持。

#cas4.0——基于Memcached的Ticket存储

CAS默认ticket存储方式是基于内存的。这个对于单节点的cas-server发布是没有问题。假如采用集群方式部署cas-server，为了使各节点共享ticket，就必须将其存放在一个公共的存储介质中，这里选择Memcached作为公共存储介质。

##依赖

```xml
<dependency>
  <groupId>org.jasig.cas</groupId>
  <artifactId>cas-server-integration-memcached</artifactId>
</dependency>
```

##配置
```xml
<bean id="ticketRegistry"
  class="org.jasig.cas.ticket.registry.MemCacheTicketRegistry"
  c:hostnames="Transcoderhost:11211"
  c:ticketGrantingTicketTimeOut="36000"
  c:serviceTicketTimeOut="2"/>
```
以上是一个简单的配置。
* 使用`spyMemcached`做为客户端与Memcached实现通讯。
* 默认使用Java序列化方式。

如果你想细粒度的控制`spyMemcached`可以使用如下配置:
```xml
<bean id="ticketRegistry"
  class="org.jasig.cas.ticket.registry.MemCacheTicketRegistry">
  <constructor-arg index="0">
    <bean class="net.spy.memcached.spring.MemcachedClientFactoryBean"
      p:servers="localhost:11211"
      p:protocol="BINARY"
      p:locatorType="ARRAY_MOD"
      p:failureMode="Redistribute"
      p:transcoder-ref="serialTranscoder">
      <property name="hashAlg">
        <util:constant static-field="net.spy.memcached.DefaultHashAlgorithm.FNV1A_64_HASH" />
      </property>
    </bean>
  </constructor-arg>
  <constructor-arg index="1" value="36000" />
  <constructor-arg index="2" value="2" />
</bean>

<bean id="serialTranscoder"
  class="net.spy.memcached.transcoders.SerializingTranscoder"
  p:compressionThreshold="2048" />

<bean id="kryoTranscover"
  class="org.jasig.cas.ticket.registry.support.kryo.KryoTranscoder" />
```
* 与Memcached使用二进制协议通讯。
* 使用一致性Hash保证数据存储。
* 使用spmemcached提供的序列化实现替换Java序列化方式。
* 也可以使用CAS提供的另一种序列化方式。`kryoTranscover`是CAS对spymemcached的`net.spy.memcached.transcoders.Transcoder`接口实现。

#cas4.0——基于数据库的Ticket存储

cas默认ticket存储方式是基于内存的。这个对于单节点的cas-server发布是没有问题。假如采用集群方式部署cas-server，为了使各节点共享ticket，就必须将其存放在一个公共的存储介质中，这里选择数据库作为公共存储介质。

官方已经为我们提供了基于数据库存储ticket的支持，我们要做的仅仅各节点的`host.name只需配置一下即可实现。

##添加依赖
```xml
<dependency>
  <groupId>org.jasig.cas</groupId>
  <artifactId>cas-server-support-jdbc</artifactId>
</dependency>

<dependency>
  <groupId>c3p0</groupId>
  <artifactId>c3p0</artifactId>
</dependency>

<dependency>
  <groupId>org.hibernate</groupId>
  <artifactId>hibernate-entitymanager</artifactId>
</dependency>

<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
</dependency>
```
以上是所需依赖的配置。本文章使用mysql数据库举例，请根据你选择的数据库类型替换相应的配置信息。

注意：`cas-server-support-jdbc`模块不仅提供了支持**数据库验证**方式，而且也支持ticket的数据库存储。

##配置
ticket的数据库存储方式，主要由两部分组成:
* 对各类ticket持久化数据库的操作。主要是通过JPA方式实现的。注意CAS使用的Hibernate JPA实现方式，这种方式无法在weblogic下部署。weblogic推荐的JPA实现为EclipseLink　JPA实现。
* 对过期ticket的清理。这部分工作需要一个定时器的支持。

###ticket持久化操作配置：
```xml
<util:list id="packagesToScan">
  <value>org.jasig.cas.services</value>
  <value>org.jasig.cas.ticket</value>
  <value>org.jasig.cas.adaptors.jdbc</value>
</util:list>

<bean id="ticketRegistry" class="org.jasig.cas.ticket.registry.JpaTicketRegistry" />

<!--
Injects EntityManager/Factory instances into beans with
@PersistenceUnit and @PersistenceContext
-->
<bean class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/>


<bean id="entityManagerFactory"
  class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean"
  p:dataSource-ref="dataSource"
  p:jpaVendorAdapter-ref="jpaVendorAdapter"
  p:packagesToScan-ref="packagesToScan">
  <property name="jpaProperties">
    <props>
      <prop key="hibernate.dialect">${database.dialect}</prop>
      <prop key="hibernate.hbm2ddl.auto">update</prop>
    </props>
  </property>
</bean>

<bean id="jpaVendorAdapter"
  class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"
  p:generateDdl="true"
  p:showSql="true"/>

<bean id="transactionManager"
  class="org.springframework.orm.jpa.JpaTransactionManager"
  p:entityManagerFactory-ref="entityManagerFactory" />

<tx:annotation-driven transaction-manager="transactionManager"/>
```
配置很简单，主要工作是由`ticketRegistry`完成。这里需要注意的是：
* `entityManagerFactory`的`packagesToScan-ref`属性。本文配置并没有使用`persistence.xml`文件。所有需要JPA支持的类都是通过`packagesToScan-ref`指定包名，而后由`entityManagerFactory`自动扫描完成的。
* `jpaVendorAdapter`指定了cas-server会在启动时根据JPA注解自动创建所需要的表。当表结构发生变化时也会自动更新。

当你访问存储ticket的数据库时，你会发现以下表被创建：
* locks
* RegistredServiceImpl
* rs_attributes
* serviceticket
* ticketgrantingticket

###ticket清理配置
```xml
<bean id="cleanerLock"
  class="org.jasig.cas.ticket.registry.support.JpaLockingStrategy"
  p:uniqueId="${host.name}"
  p:applicationId="cas-ticket-registry-cleaner@${host.name}"/>

<bean id="ticketRegistryCleaner"
  class="org.jasig.cas.ticket.registry.support.DefaultTicketRegistryCleaner"
  p:ticketRegistry-ref="ticketRegistry"
  p:logoutManager-ref="logoutManager"
  p:lock-ref="cleanerLock" />

<bean id="jobDetailTicketRegistryCleaner"
  class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean"
  p:targetObject-ref="ticketRegistryCleaner"
  p:targetMethod="clean" />

<bean id="triggerJobDetailTicketRegistryCleaner"
  class="org.springframework.scheduling.quartz.SimpleTriggerBean"
  p:jobDetail-ref="jobDetailTicketRegistryCleaner"
  p:startDelay="20000"
  p:repeatInterval="1800000" />
```
* `ticketRegistryCleaner`主要完成ticket清理工作。
* `cleanerLock`主要解决集群环境下清理ticket时出现表争用的问题（即防止锁表）。注意它的两个参数`uniqueId`和`applicationId`。**建议集群环境下不同节点下的这两个参数要加以区分，如果你不加以区分，同时定时器的时间间隔又是相同的，即使配置了`cleanerLock`还是会出现表争用的问题**。这里是通过配置各节点不同的`host.name`(在cas.properties中)来加以区分各节点的`cleanerLock`。
* 不要忘了为`ticketRegistryCleaner`配置一个调度任务定时去执行。

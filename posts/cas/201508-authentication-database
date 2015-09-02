CAS4.0——数据库认证
===

##添加依赖
数据库认证由模块`cas-server-support-jdbc`提供，这里使用`mysql`存储用户/密码。修改你的pom文件，添加如下依赖：
```xml
<dependency>
    <groupId>org.jasig.cas</groupId>
    <artifactId>cas-server-support-jdbc</artifactId>
    <version>${cas.version}</version>
</dependency>
<dependency>
    <groupId>c3p0</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.1.2</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.25</version>
</dependency>
```


##配置

首先添加如下两项schema以方便Bean配置
```xml
xmlns:p="http://www.springframework.org/schema/p"
xmlns:c="http://www.springframework.org/schema/c"
```

###配置数据源
```xml
<bean id="dataSource"
      class="com.mchange.v2.c3p0.ComboPooledDataSource"
      p:driverClass="${database.driverClass}"
      p:jdbcUrl="${database.url}"
      p:user="${database.user}"
      p:password="${database.password}"
      p:initialPoolSize="${database.pool.minSize}"
      p:minPoolSize="${database.pool.minSize}"
      p:maxPoolSize="${database.pool.maxSize}"
      p:maxIdleTimeExcessConnections="${database.pool.maxIdleTime}"
      p:checkoutTimeout="${database.pool.maxWait}"
      p:acquireIncrement="${database.pool.acquireIncrement}"
      p:acquireRetryAttempts="${database.pool.acquireRetryAttempts}"
      p:acquireRetryDelay="${database.pool.acquireRetryDelay}"
      p:idleConnectionTestPeriod="${database.pool.idleConnectionTestPeriod}"
      p:preferredTestQuery="${database.pool.connectionHealthQuery}" />
```
以上并不是每个属性都是必须的，这只是个通常配置，可自行删减。
在cas.properties最后增加相应变量的设置
```
# == Basic database connection pool configuration ==
database.driverClass=com.mysql.jdbc.Driver
database.url=jdbc:mysql://localhost:3306/upm?useUnicode=true&characterEncoding=utf-8
database.user=root
database.password=password
database.pool.minSize=6
database.pool.maxSize=18

# Maximum amount of time to wait in ms for a connection to become
# available when the pool is exhausted
database.pool.maxWait=10000

# Amount of time in seconds after which idle connections
# in excess of minimum size are pruned.
database.pool.maxIdleTime=120

# Number of connections to obtain on pool exhaustion condition.
# The maximum pool size is always respected when acquiring
# new connections.
database.pool.acquireIncrement=6

# == Connection testing settings ==

# Period in s at which a health query will be issued on idle
# connections to determine connection liveliness.
database.pool.idleConnectionTestPeriod=30

# Query executed periodically to test health
database.pool.connectionHealthQuery=select 1

# == Database recovery settings ==

# Number of times to retry acquiring a _new_ connection
# when an error is encountered during acquisition.
database.pool.acquireRetryAttempts=5

# Amount of time in ms to wait between successive aquire retry attempts.
database.pool.acquireRetryDelay=2000
```
###配置认证Bean

CAS提供了多个认证Bean来适应不同的数据库认证需求。


首先，假设有一个存储用户信息的表结构如下：

```sql
create table users (
        username varchar(50) not null,
        password var#####QueryAndEncodeDatabaseAuthenticationHandler
char(50) not null,
        active bit not null );
```

#####QueryDatabaseAuthenticationHandler
通过比较密码的方式来认证用户，其中一个来自登陆表单，另一个来自数据库。通过配置SQL的方式，来获取存储在数据库中的密码然后进行比较。

下面的列子展示了一个密码使用MD5散列后存储在数据库中的例子。
```xml
<bean id="dbAuthHandler"
      class="org.jasig.cas.adaptors.jdbc.QueryDatabaseAuthenticationHandler"
      p:dataSource-ref="dataSource"
      p:passwordEncoder-ref="passwordEncoder"
      p:sql="select password from users where username=? and active=1"/>

<bean id="passwordEncoder"
      class="org.jasig.cas.authentication.handler.DefaultPasswordEncoder"
      c:encodingAlgorithm="MD5"
      p:characterEncoding="UTF-8"/>
```

#####SearchModeSearchDatabaseAuthenticationHandler
这个Bean与`QueryDatabaseAuthenticationHandler`很类似，但是使用更简单。你只需指定表名和字段，它会自动构建查询语句。
```xml
<bean id="dbAuthHandler"
          class="org.jasig.cas.adaptors.jdbc.SearchModeSearchDatabaseAuthenticationHandler"
          p:dataSource-ref="dataSource"
          p:tableUsers="users"
          p:fieldUser="username"
          p:fieldPassword="password"
          p:passwordEncoder-ref="passwordEncoder"/>
```

#####BindModeSearchDatabaseAuthenticationHandler
不用于以上两个，它是通过验证提供的账号和密码是否能够创建数据库连接的方式，验证用户合法性。换句话说就是，与前面的`users`表内数据无关，登陆系统只能使用数据库连接账号。
```xml
<bean id="dbAuthHandler"
          class="org.jasig.cas.adaptors.jdbc.BindModeSearchDatabaseAuthenticationHandler"
          p:dataSource-ref="dataSource"/>
```

###配置认证管理器
在`deployerConfigContext.xml`中修改`authenticationManager`Bean配置如下：

```xml
<bean id="authenticationManager" class="org.jasig.cas.authentication.PolicyBasedAuthenticationManager">
    <constructor-arg>
        <map>
            <entry key-ref="proxyAuthenticationHandler" value-ref="proxyPrincipalResolver" />
            <entry key-ref="primaryAuthenticationHandler" value-ref="primaryPrincipalResolver" />
            <entry key-ref="dbAuthHandler" value-ref="primaryPrincipalResolver"/>
            <!--<entry key-ref="dbAuthHandler"><null/></entry>-->
        </map>
    </constructor-arg>
    <property name="authenticationPolicy">
        <bean class="org.jasig.cas.authentication.AnyAuthenticationPolicy" />
    </property>
</bean>
```

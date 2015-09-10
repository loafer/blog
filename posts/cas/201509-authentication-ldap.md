#cas4.0——LDAP认证


##添加依赖
LDAP认证由模块`cas-server-support-ldap`提供。修改你的pom文件，添加所依赖的jar。
```xml
<dependency>
  <groupId>org.jasig.cas</groupId>
  <artifactId>cas-server-support-ldap</artifactId>
  <version>${cas.version}</version>
</dependency>
```

##关于LDAP认证Principal属性问题
`LdapAuthenticationHandler`不需要任何Principal解析机制就能独立完成Principal属性的解析和检索。

```xml
<bean id="ldapAuthenticationHandler"
  class="org.jasig.cas.authentication.LdapAuthenticationHandler"
  p:principalIdAttribute="sAMAccountName"
  c:authenticator-ref="authenticator">
  <property name="principalAttributeMap">
    <map>
      <entry key="displayName" value="simpleName" />
      <entry key="mail" value="email" />
      <entry key="cn" value="cn" />
    </map>
  </property>
</bean>
```

上面这段代码演示了LDAP到CAS属性的映射（从LDAP接收一个属性mail，然后将它映射到CAS的email属性上）。其中key是LDAP中的属性名，value是CAS中你期望的属性名。如果LDAP到CAS属性名一致，则可以直接将上面的配置改为如下：

```xml
<bean id="ldapAuthenticationHandler"
  class="org.jasig.cas.authentication.LdapAuthenticationHandler"
  p:principalIdAttribute-ref="usernameAttribute"
  c:authenticator-ref="authenticator">
  <property name="principalAttributeList">
    <list>
      <value>displayName</value>
      <value>mail</value>
      <value>cn</value>
    </list>
  </property>
</bean>
```

##验证配置
CAS支持４种常见LDAP配置。
* Active Directory ——　使用　sAMAAccountName　验证用户
* Authenticated Search ——　通过管理账号查询被验证用户
* Anonymous Search  ——　不需要管理账号直接搜索被验证用户
* Direct Bind ——　直接在指定的存有用户信息的DN下查找被验证用户

首先我们写一个通用的LDAP连接池配置，因为以上４种方式都需要。

```xml
<bean id="abstractConnectionPool" abstract="true"
  class="org.ldaptive.pool.BlockingConnectionPool"
  init-method="initialize"
  destroy-method="close"
  p:poolConfig-ref="ldapPoolConfig"
  p:blockWaitTime="${ldap.pool.blockWaitTime:3000}"
  p:validator-ref="searchValidator"
  p:pruneStrategy-ref="pruneStrategy"
  p:connectionFactory-ref="connectionFactory" />

<bean id="ldapPoolConfig" class="org.ldaptive.pool.PoolConfig"
  p:minPoolSize="${ldap.pool.minSize:1}"
  p:maxPoolSize="${ldap.pool.maxSize:2}"
  p:validateOnCheckOut="${ldap.pool.validateOnCheckout:false}"
  p:validatePeriodically="${ldap.pool.validatePeriodically:true}"
  p:validatePeriod="${ldap.pool.validatePeriod:300}" />

<bean id="connectionFactory" class="org.ldaptive.DefaultConnectionFactory"
  p:connectionConfig-ref="connectionConfig" />

<bean id="connectionConfig" class="org.ldaptive.ConnectionConfig"
  p:ldapUrl="${ldap.url}"
  p:connectTimeout="${ldap.connectTimeout:3000}"
  p:useStartTLS="${ldap.useStartTLS:false}"
  p:connectionInitializer-ref="bindConnectionInitializer"/>

<bean id="bindConnectionInitializer" class="org.ldaptive.BindConnectionInitializer"
  p:bindDn="${ldap.authn.managerDN}">
  <property name="bindCredential">
    <bean class="org.ldaptive.Credential" c:password="${ldap.authn.managerPassword}"/>
  </property>
</bean>

<bean id="pruneStrategy" class="org.ldaptive.pool.IdlePruneStrategy"
  p:prunePeriod="${ldap.pool.prunePeriod:300}"
  p:idleTime="${ldap.pool.idleTime:300}" />

<bean id="searchValidator" class="org.ldaptive.pool.SearchValidator" />
```
以上是通用LDAP连接池配置，其中各变量单独写在了ldap.properties中，你也可以将其写入cas.properties中。**注意，请将${ldap.url}换成你实际的地址。**


####使用sAMAAccountName验证方式（推荐指数：★★★★☆）
sAMAAccountName是AD(Active Directory)个人dn节点上的一个属性，可用于登陆，主要是针对微软早期操作系统，如：Windows NT 4.0, Windows 95, Windows 98, and LAN Manager。

```xml
<bean id="ldapAuthHandler"
  class="org.jasig.cas.authentication.LdapAuthenticationHandler"
  p:principalIdAttribute="sAMAccountName"
  c:authenticator-ref="authenticator">
  <property name="principalAttributeMap">
    <map>
      <entry key="displayName" value="displayName" />
      <entry key="cn" value="cn" />
    </map>
  </property>
</bean>

<bean id="authenticator" class="org.ldaptive.auth.Authenticator"
  c:resolver-ref="dnResolver"
  c:handler-ref="authHandler"
  p:entryResolver-ref="entryResolver">
  <property name="authenticationResponseHandlers">
    <list>
      <bean class="org.ldaptive.auth.ext.ActiveDirectoryAuthenticationResponseHandler" />
    </list>
  </property>
</bean>

<bean id="dnResolver"
  class="org.ldaptive.auth.FormatDnResolver"
  c:format="%s@${ldap.domain}" />

<bean id="authHandler"
  class="org.ldaptive.auth.PooledBindAuthenticationHandler"
  p:connectionFactory-ref="bindPooledLdapConnectionFactory" />

<bean id="bindPooledLdapConnectionFactory"
  class="org.ldaptive.pool.PooledConnectionFactory"
  p:connectionPool-ref="bindConnectionPool" />

<bean id="bindConnectionPool" parent="abstractConnectionPool" />

<!-- If you wish to search by user, rather than by dn, change {dn} to {user} -->
<bean id="entryResolver"
  class="org.ldaptive.auth.SearchEntryResolver"
  p:baseDn="${ldap.authn.baseDn}"
  p:userFilter="userPrincipalName={dn}"
  p:subtreeSearch="true" />
```
以上是基于`sAMAccountName`的配置，有４个地方需要注意：
* `ldapAuthHandler`的`principalIdAttribute`属性值应该设置为`sAMAccountName`。
* `dnResolver`的构造参数`format`定义的格式通常类似邮件地址`登陆账号@域名`（即：tom@163.com）的形式。
* `entryResolver`的`userFilter`属性。以上配置中变量`{dn}`会被`dnResolver`指定的`format`实际值替换掉。
- `entryResolver`的`baseDn`根据你的LDAP实际情况设置。


####使用Authenticated Search验证方式（推荐指数：★★★★★）

这种方式比较灵活，可以随意定义使用个人dn节点的哪个属性用于登陆。

```xml
<bean id="ldapAuthHandler"
  class="org.jasig.cas.authentication.LdapAuthenticationHandler"
  p:principalIdAttribute="cn"
  c:authenticator-ref="authenticator">
  <property name="principalAttributeMap">
    <map>
      <entry key="displayName" value="displayName" />
      <entry key="cn" value="cn" />
    </map>
  </property>
</bean>

<bean id="authenticator"
  class="org.ldaptive.auth.Authenticator"
  c:resolver-ref="dnResolver"
  c:handler-ref="authHandler" />

<bean id="dnResolver"
  class="org.ldaptive.auth.PooledSearchDnResolver"
  p:baseDn="${ldap.authn.baseDn}"
  p:subtreeSearch="true"
  p:allowMultipleDns="false"
  p:connectionFactory-ref="bindPooledLdapConnectionFactory"
  p:userFilter="cn={user}"/>

<bean id="authHandler"
  class="org.ldaptive.auth.PooledBindAuthenticationHandler"
  p:connectionFactory-ref="bindPooledLdapConnectionFactory" />

<bean id="bindPooledLdapConnectionFactory"
  class="org.ldaptive.pool.PooledConnectionFactory"
  p:connectionPool-ref="bindConnectionPool" />

<bean id="bindConnectionPool" parent="abstractConnectionPool" />
```
以上配置通过查询是否存在一个特定cn属性值的个人dn节点，来验证用户合法性。有3点需要注意：
* `ldapAuthHandler`的`principalIdAttribute`属性，这里指定的是`cn`，你也可以指定`uid`，这个需要和`dnResolver`的`userFilter`值对应。
* `dnResolver`的`userFilter`属性要与`ldapAuthHandler`的`principalIdAttribute`属性对应。
* `dnResolver`的`baseDn`属性根据你的LDAP实际值填写。



####使用Direct Bind验证方式（推荐指数：★★★☆☆）
这种方式有２点需要注意：
* 将所有用户都集中在一个dn节点下。
* 你要准确的知道这个dn节点。

```xml
<bean id="ldapAuthHandler"
  class="org.jasig.cas.authentication.LdapAuthenticationHandler"
  p:principalIdAttribute="cn"
  c:authenticator-ref="authenticator">
  <property name="principalAttributeMap">
    <map>
      <entry key="displayName" value="displayName" />
      <entry key="cn" value="cn" />
    </map>
  </property>
</bean>

<bean id="authenticator"
  class="org.ldaptive.auth.Authenticator"
  c:resolver-ref="dnResolver"
  c:handler-ref="authHandler"/>

<bean id="dnResolver"
  class="org.ldaptive.auth.FormatDnResolver"
  c:format="CN=%s,OU=Users,DC=exmaple,DC=org" />

<bean id="authHandler"
  class="org.ldaptive.auth.PooledBindAuthenticationHandler"
  p:connectionFactory-ref="bindPooledLdapConnectionFactory" />

<bean id="bindPooledLdapConnectionFactory"
  class="org.ldaptive.pool.PooledConnectionFactory"
  p:connectionPool-ref="bindConnectionPool" />

<bean id="bindConnectionPool" parent="abstractConnectionPool" />
```

以上是使用Direct Bind验证的配置方式。这里使用`cn`属性作为登陆账号。注意：
* `ldapAuthHandler`的`principalIdAttribute`属性值，你可以指定任意你需要的。
* `dnResolver`的`format`参数。这个要具体到拥有人员信息的那个节点。比如我们单位可能会存在多个`ou`(CN=%s,OU=XXXXX,OU=Users,DC=exmaple,DC=org)。请根据自己实际情况填写。

####使用Anonymous Search验证方式（推荐指数：★☆☆☆☆）
一般企业都不会提供匿名访问LDAP。这里不再详细介绍，具体配置请参阅[这里](http://jasig.github.io/cas/4.1.x/installation/LDAP-Authentication.html#ldap-supporting-anonymous-search)。

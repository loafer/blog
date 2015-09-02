CAS4.0——Http部署方式
===
１、修改deployerConfigContext.xml中定义的proxyAuthenticationHandler的**requireSecure**属性为**false**。
```xml
<bean id="proxyAuthenticationHandler"
          class="org.jasig.cas.authentication.handler.support.HttpBasedServiceCredentialsAuthenticationHandler"
          p:httpClient-ref="httpClient"
          p:requireSecure="false"/>
```

2、修改spring-configuration/ticketGrantingTicketCookieGenerator.xml中定义的ticketGrantingTicketCookieGenerator的**cookieSecure**属性为**false**。
```xml
<bean id="ticketGrantingTicketCookieGenerator"
    class="org.jasig.cas.web.support.CookieRetrievingCookieGenerator"
    p:cookieSecure="false"
    p:cookieMaxAge="-1"
    p:cookieName="CASTGC"
    p:cookiePath="/cas" />
```

3、修改spring-configuration/warnCookieGenerator.xml中定义的warnCookieGenerator的**cookieSecure**属性值为**false**。
```xml
<bean id="warnCookieGenerator"
    class="org.jasig.cas.web.support.CookieRetrievingCookieGenerator"
    p:cookieSecure="false"
    p:cookieMaxAge="-1"
    p:cookieName="CASPRIVACY"
    p:cookiePath="/cas" />
```

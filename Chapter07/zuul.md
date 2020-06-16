# ZUUL

```yaml

#网关
zuul:
  ribbon:
    eager-load:
      enabled: true #zuul饥饿加载
  host:
    max-total-connections: 200
    max-per-route-connections: 20
    #以下两个配置也是解决zuul超时的
    #和使用ribbon.ReadTimeout的区别是，如果路由配置使用service-id的方式，那么ribbon.ReadTimeout生效，如果使用url的方式，此配置生效
    connect-timeout-millis: 10000
    socket-timeout-millis: 10000
 
#配置Ribbon的超时时间
ribbon:
  ReadTimeout: 10000
  ConnectTimeout: 10000
#  MaxAutoRetries: 1
#  MaxAutoRetriesNextServer: 1
 
hystrix:
  command:
    default:
      execution:
        isolation:
          strategy: SEMAPHORE
          #配置hystrix的超时时间
          thread:
            timeoutInMilliseconds: 20000
```
访问的时候提示错误：

```text
The Hystrix timeout of 20000ms for the command uaa-service is set lower than the combination of the Ribbon read and connect timeout, 40000ms.
```

分析：

Ribbon 总超时时间计算公式如下：

```text
ribbonTimeout = (RibbonReadTimeout + RibbonConnectTimeout) * (MaxAutoRetries + 1) * (MaxAutoRetriesNextServer + 1)
```

其中，MaxAutoRetries 默认为0，MaxAutoRetriesNextServer 默认为1，所以我这里的具体值为：（10000+10000）*（0+1）*（1+1）=40000。

而 Hystrix 超时时间为 20000 < 40000，从逻辑上来讲，hystrixTimeout 要大于 ribbonTimeout，否则 hystrix 熔断了以后，ribbon 的重试就都没有意义了。



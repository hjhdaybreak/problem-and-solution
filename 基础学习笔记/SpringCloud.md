# SpringCloud



SpringCloud 和 Duubo 区别

+ 两者都是现在主流的微服务框架
+ SpringCloud 定位为微服务架构下的一站式解决方案；Dubbo  它的关注点主要在于服务的调用和治理
+ SpringCloud依托于Spring平台，具备更加完善的生态体系；而Dubbo一开始只是做RPC远程调用，生态相对匮乏，现在逐渐丰富起来
+ SpringCloud是采用Http协议做远程调用，接口一般是Rest风格，比较灵活；Dubbo是采用Dubbo协议

## SpringCloud 大纲 

![image-20221018233707721](G:\markdown图片\image-20221018233707721.png)



## 微服务架构进化论

+ 单体应用阶段     一般网站的流量也很少，但硬件成本较高，一般的企业会将所有的功能都集成在一起开发一个单体应用
  + ![image-20221018233824703](G:\markdown图片\image-20221018233824703.png)

+ 垂直应用阶段    干不过来，扩大规模；在处理并发请求的能力和容量上增强了，但是在单个请求的处理速度上下降了
  + ![image-20221018234047218](G:\markdown图片\image-20221018234047218.png)

+ 分布式系统阶段   为了解决单个请求的处理速度下降，将服务分组，专业的人做专业的事情
  + ![image-20221018234331279](G:\markdown图片\image-20221018234331279.png)

+ 服务治理阶段  服务之间沟通不畅，如何组合问题  
  + ![image-20221018234400081](G:\markdown图片\image-20221018234400081.png)

+ 微服务阶段   将系统的业务功能划分为极小的独立微服务，每个微服务只关注于完成某个小的任务
  + ![image-20221018234555175](G:\markdown图片\image-20221018234555175.png)

## 微服务的拆分规范和原则

+ 压力模型拆分：识别出某些超高并发量的业务，尽可能把这部分业务独立拆分出来
+ 业务模型拆分  1.主链路拆分 2.领域模型拆分 3.用户群体 

## 如何选择 SpringCloud 版本    

+ 最新版本 GA 版本 (正式发布)  

## 如何学习微服务  

+ ![image-20221019100755142](G:\markdown图片\image-20221019100755142.png)



Run Dashboard面板

在.idea/workspace.xml 文件中



```xml
 <component name="RunDashboard">
 <option name="ruleStates">
  <list>
   <RuleState>
    <option name="name" value="ConfigurationTypeDashboardGroupingRule" />
   </RuleState>
   <RuleState>
    <option name="name" value="StatusDashboardGroupingRule" />
   </RuleState>
  </list>
 </option>
 <option name="configurationTypes">
 <set>
  <option value="SpringBootApplicationConfigurationType" />
 </set>
</option>
</component>
```



## 服务注册与发现



Eureka 

（1）Eureka 启动

（2）服务提供者注册到 Eureka

  (3)  服务消费者注册到 Eureka 



服务自保和剔除 (互斥)

+ 服务剔除 : 服务剔除把服务节点果断剔除，即使你的续约请求晚了一步也毫不留情，招式凌厉，重在当断则断，忍痛割爱
+ 剔除：服务自保把当前所有节点保留，一个都不能少，绝不放弃任何队友

参数来关闭保护机制，以确保注册中心可以将不可用的实例正确剔除，默认为 true 

+ 在实际应用里，并不是所有无心跳的服务都不可用，也许因为短暂的网络抖动等原因，导致服务节点与注册中心之间续约不上，但服务节点之间的调用还是属于可用状态，这时如果强行剔除服务节点，可能会造成大范围的业务停滞 

```yaml
eureka.server.enable-self-preservation=false; 
```



actuator 微服务信息完善 

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

![image-20221022111445423](G:\markdown图片\image-20221022111445423.png)



服务发现 Discovery 

+ ```
  获取服务列表: UI 界面中的 Application
  List<String> services = discoveryClient.getServices();
  
  hostName : UI 界面中的 Application   instances：UI中的  Status
  String hostName = "cloud-payment-provider";
  List<ServiceInstance> instances = discoveryClient.getInstances(hostName);
  serviceInstance.getInstanceId() 获取 Status = instanceId 
  serviceInstance.getServiceId()  ServiceId = UI 界面中的 Application = discoveryClient.getServices
  ```

高可用 Eureka 注册中心 

+ 搭建 Eureka 集群  

+ ```yaml
  server:
    port: 7001
  eureka:
    server:
      enable-self-preservation: false
    instance:
      # eureka 服务端的实例名字
      hostname: eureka7001.com
    client:
      # 表示是否将自己注册到 Eureka Server
      register-with-eureka: false
      # 表示是否从 Eureka Server 获取注册的服务信息
      fetch-registry: false
      # 设置与 Eureka server交互的地址查询服务和注册服务都需要依赖这个地址
      service-url:
        defaultZone: http://eureka7002.com:7002/eureka/
  
  ```

+ ```yaml
  server:
    port: 7002
  eureka:
    server:
      enable-self-preservation: false
    instance:
      # eureka 服务端的实例名字
      hostname: eureka7002.com
    client:
      # 表示是否将自己注册到 Eureka Server
      register-with-eureka: false
      # 表示是否从Eureka Server获取注册的服务信息
      fetch-registry: false
      # 设置与 Eureka server交互的地址查询服务和注册服务都需要依赖这个地址
      service-url:
        defaultZone: http://eureka7001.com:7001/eureka/ 
  
  ```

  ![image-20221020202321203](G:\markdown图片\image-20221020202321203.png)

  ![image-20221020202339453](G:\markdown图片\image-20221020202339453.png)



## 负载均衡

+ 客户端负载均衡      

  ![image-20221021102242935](G:\markdown图片\image-20221021102242935.png)

  

  Spring Cloud LoadBalancer

  + RandomLoadBalancer 随机
  + RoundRobinLoadBalancer 轮询 

## 服务调用

OpenFeign (接口 + 注解)![image-20221021103804001](G:\markdown图片\image-20221021103804001.png)

运行逻辑：启动类 @EnableFeignClients，会扫描 @FeignClient注解，value = "cloud-payment-provider" 找到 服务名是 cloud-payment-provider 的服务  



OpenFeign日志增强

+ **NONE**：默认的，不显示任何日志;
+ **BASIC**：仅记录请求方法、URL、响应状态码及执行时间;
+ **HEADERS**：除了BASIC中定义的信息之外，还有请求和响应的头信息;
+ **FULL**：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据 

```
logging:
  level:
    com.bee.service: debug
```



OpenFeign超时机制

+ 服务消费者在调用服务提供者的时候发生了阻塞、等待的情形，这个时候，服务消费者会一直等待下去。
+ 在某个峰值时刻，大呈的请求都在同时请求服务消费者，会造成线程的大呈堆积，势必会造成雪崩。
+ 利用超时机制来解决这个问题，设置一个超时时间，在这个时间段内，无法完成服务访问，则自动断开连接

```yaml
# 默认超时时间
feign:
  client:
    config:
      default:
        # 连接超时时间
        connectTimeout: 2000
          # 读取超时时间
        readTimeout: 2000 
```



## **服务断路器**

假设我们有两个访问量比较大的服务A和B，这两个服务分别依赖C和D,C和D服务都依赖E服务。

![image-20220223144040490](G:\markdown图片\image-20220223144040490.png)

A和B不断的调用C,D处理客户请求和返回需要的数据。当E服务不能供服务的时候，C和D的`超时`和`重试`机制会被执行

![image-20220223144109720](G:\markdown图片\image-20220223144109720.png)

由于新的调用不断的产生，会导致C和D对E服务的调用大量的积压，产生大量的调用等待和重试调用，慢慢会耗尽C和D的资源比如内存或CPU，然后也down掉。

![image-20220223144130462](G:\markdown图片\image-20220223144130462.png)

A和B服务会重复C和D的操作，资源耗尽，然后down掉，最终整个服务都不可访问。

![image-20220223144155221](G:\markdown图片\image-20220223144155221.png)

结论：服务与服务之间的依赖性，故障会传播，造成连锁反应，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的“雪崩”效应。

造成雪崩原因是什么 

1. 服务提供者不可用（硬件故障、程序bug、缓存击穿、用户大量请求）
2. 重试加大流量（用户重试，代码逻辑重试）
3. 服务调用者不可用（同步等待造成的资源耗尽）



雪崩效应解决方案之**服务熔断**

​	当一个服务请求并发特别大，服务器已经招架不住了，调用错误率飙升，当错误率达到一定阈值后，就将这个服务熔断了。熔断之后，后续的请求就不会再请求服务器了，以减缓服务器的压力

![image-20221021162314886](G:\markdown图片\image-20221021162314886.png)





雪崩效应解决方案之**服务降级**

服务降级两种场景:

- 当下游的服务因为某种原因响应过慢，下游服务主动停掉一些不太重要的业务，释放出服务器资源，增加响应速度！
- 当下游的服务因为某种原因不可用，上游主动调用本地的一些降级逻辑，避免卡顿，迅速返回给用户！

出现服务降级的情况：

- 程序运行异常
- 超时
- 服务熔断触发服务降级
- 线程池/信号量打满也会导致服务降级



雪崩效应解决方案之**服务隔离**

线程池隔离和信号量隔离

+ ![image-20221021184731000](G:\markdown图片\image-20221021184731000.png)

  ![image-20221021184738040](G:\markdown图片\image-20221021184738040.png)

  ![image-20221021190250068](G:\markdown图片\image-20221021190250068.png)

  ![image-20221021190605878](G:\markdown图片\image-20221021190605878.png)



服务雪崩解决方案之**服务限流** 

​	限流的目的是通过对并发访问/请求进行限速，或者对一个时间窗口内的请求进行限速来保护系统，一旦达到限制速率则可以拒绝服务、排队或等待、降级等处理

- 网关限流：防止大量请求进入系统，Mq实现流量消峰
- 用户交流限流：提交按钮限制点击频率限制等



Resilience4j  

​	断路器（CircuitBreaker）相对于前面几个熔断机制更复杂，CircuitBreaker**通常**存在三种状态（CLOSE、OPEN、HALF_OPEN），并通过一个时间或数量窗口来记录当前的请求成功率或慢速率，从而根据这些指标来作出正确的容错响应

+ CLOSED: 关闭状态，代表正常情况下的状态，允许所有请求通过,能通过状态转换为OPEN
+ HALF_OPEN: 半开状态，即允许一部分请求通过,能通过状态转换为CLOSED和OPEN
+ OPEN: 熔断状态，即不允许请求通过，能通过状态转为为HALF_OPEN
+ DISABLED: 禁用状态，即允许所有请求通过，出现失败率达到给定的阈值也不会熔断，不会发生状态转换。
+ `METRICS_ONLY: 和DISABLED状态一样，也允许所有请求通过不会发生熔断，但是会记录失败率等信息，不会发生状态转换。`
+ FORCED_OPEN: 与DISABLED状态正好相反，启用CircuitBreaker，但是不允许任何请求通过，不会发生状态转换。`

![image-20221021193338997](G:\markdown图片\image-20221021193338997.png)





Resilience4j 的重试机制

```yaml
resilience4j:
 retry:
   instances:
    backendA:
    # 最大重试次数
     maxRetryAttempts: 3
    # 固定的重试间隔
     waitDuration: 10s
     enableExponentialBackoff: true
     exponentialBackoffMultiplier: 2
```

Resilience4j的异常比例熔断降级

```yaml
#熔断机制
resilience4j.circuitbreaker:
  instances:
    backendA:
      baseConfig: default

  configs:
    default:
      # 熔断器打开的失败阈值
      failureRateThreshold: 30
      # 默认滑动窗口大小,circuitbreaker使用基于计数和时间范围欢动窗口聚合统计失败率
      slidingWindowSize: 10
      # 计算比率的最小值,即当请求发生5次才会计算失败率
      minimumNumberOfCalls: 5
      # 滑动窗口类型,默认为基于计数的滑动窗口
      slidingWindowType: TIME_BASED
      # 半开状态允许的请求数
      permittedNumberOfCallsInHalfOpenState: 3
      # 是否自动从打开到半开
      automaticTransitionFromOpenToHalfOpenEnabled: true
      # 熔断器从打开到半开需要的时间
      waitDurationInOpenState: 2s
      recordExceptions:
        - java.lang.Exception 
```



Resilience4j的慢调用比例熔断降级

```yaml
backendB:
      # 熔断器打开的失败阈值
      failureRateThreshold: 50
      # 慢调用时间阈值 高于这个阈值的
      slowCallDurationThreshold: 2s
      # 慢调用百分比阈值，断路器吧调用事件大于slow
      slowCallRateThreshold: 30
      slidingWindowSize: 10
      slidingWindowType: TIME_BASED
      minimumNumberOfCalls: 2
      permittedNumberOfCallsInHalfOpenState: 2
      waitDurationInOpenState: 2s
      eventConsumerBufferSize: 10	
```

Resilience4j  的信号量隔离 

```yaml
resilience4j:
 #信号量隔离
  bulkhead:
   instances:
    backendA:
    # 隔离允许并发线程执行的最大数量
     maxConcurrentCalls: 5
    # 当达到并发调用数量时，新的线程的阻塞时间
     maxWaitDuration: 20ms
```

Resilience4j的线程池服务隔离

```yaml
resilience4j:
  thread-pool-bulkhead:  
   instances:
    backendA:
     # 最大线程池大小
     maxThreadPoolSize: 4
    # 核心线程池大小
     coreThreadPoolSize: 2
    #  队列容量
     queueCapacity: 2

```

Resilience4j的限流

```yaml
resilience4j:
  ratelimiter:
   instances:
    backendA:
    # 限流周期时长。    默认：500纳秒
     limitRefreshPeriod: 5s
    # 周期内允许通过的请求数量。    默认：50
     limitForPeriod: 2 
```



## 网关 

1. 访问控制  
2. 路由 

Spring Cloud Gateway 特点

- 易于编写谓词( Predicates )和过滤器（ Filters ) 。其Predicates和Filters可作用于特定路由。
- 支持路径重写。
- 支持动态路由。
- 集成了Spring Cloud DiscoveryClient  





路由(Route)



断言(predicate) ：路由的判断规则。一个路由中可以添加多个谓词的组合



![image-20221029110753921](G:\markdown图片\image-20221029110753921.png)

过滤 (filter) 

![image-20221029110737078](G:\markdown图片\image-20221029110737078.png)  

### 路由

YAML 构建路由

```yaml
server:
  port: 9527
spring:
  cloud:
    gateway:
      routes:
        # 路由ID，没有固定规则但要求唯一，建议配合服务名
        - id: payment_provider
        # 匹配后提供服务的路由地址
          uri: http://localhost:8001
        # 断言
          predicates:
         # 路径相匹配的进行路由
          - Path=/payment/get/** 
```

JAVA API 构建路由

```
routes.route("path_rote",r -> r.path("/guonei").uri("http://news.baidu.com/guonei")).build();
```



动态路由

```yaml
spring:
  application:
    # 设置应用名字
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_provider
          # 匹配后提供服务的路由地址 lb (提供服务的微服务的名) 后跟提供服务的微服务的名
          uri: lb://CLOUD-PAYMENT-PROVIDER  
          # 断言
          predicates:
            # 路径相匹配的进行路由
          - Path=/payment/**
```





### 断言

After 路由断言 Factory 

After Route Predicate Factory 采用一个参数——日期时间 (UTC时间格式的时间参数) 。在该日期时间之后发生的请求都将被匹配	

```yaml
- After=2030-02-15T14:54:23.317+08:00[Asia/Shanghai]  秒杀场景 这个时间之前接口找不到 
```

Before 路由断言 Factory 

Before Route Predicate Factory采用一个参数——日期时间。在该日期时间之前发生的请求都将被匹配 

```yaml
- Before=2030-02-15T14:54:23.317+08:00[Asia/Shanghai]  特价商品
```

Between 路由断言 Factory

Cookie路由断言 Factory

Header路由断言 Factory

Host路由断言 Factory

Method路由断言 Factory

Query路由断言 Factory



### 过滤器   

在用户访问各个服务前，应在网关层统一做好鉴权、限流等工作

+ **PRE**：代表在请求被路由之前执行该过滤器，此种过滤器可用来实现参数校验、权限校验、流量监控、日志输出、协议转换等功能
+ **POST**：代表在请求被路由到微服务之后执行该过滤器。此种过滤器可用来实现响应头的修改（如添加标准的HTTP Header )、收集统计信息和指标、将响应发送给客户端、输出日志、流量监控等功能

根据作用范围，Filter可以分为以下两种。

- GatewayFilter：网关过滤器，此种过滤器只应用在单个路由或者一个分组的路由上。
- GlobalFilter：全局过滤器，此种过滤器会应用在所有的路由上。

网关过滤器

```yaml
spring:
  application:
    # 设置应用名字
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_provider
          # 匹配后提供服务的路由地址 lb 后跟提供服务的微服务的名
          uri: lb://CLOUD-PAYMENT-PROVIDER
          # 断言
          predicates:
            # 路径相匹配的进行路由
            - Path=/payment/**
            - After=2030-02-15T14:54:23.317+08:00[Asia/Shanghai]
          # - Before=2030-02-15T14:54:23.317+08:00[Asia/Shanghai]
          #过滤器，请求在传递过程中可以通过过滤器对其进行一定的修改
          filters:
          	# 修改原始响应的状态码 
            - SetStatus=250 
```

自定义网关过滤器





全局过滤器 

```java
@Slf4j
@Component
public class AuthGlobalFilter implements GlobalFilter, Ordered {
    /**
     * 自定义全局过滤器
     *
     * @param exchange
     * @param chain
     * @return
     */
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        //从请求中拿到Token
        String token = exchange.getRequest().getQueryParams().getFirst("token");

        //判断token是否存在
        if (StringUtil.isNullOrEmpty(token)) {
            log.info("鉴权失败,参数缺失");
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        //判定token是否有效
        if (!"bee".equals(token)) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        return chain.filter(exchange);
    }

    /**
     * 数值越小,优先级越高
     *
     * @return
     */
    @Override
    public int getOrder() {
        return 0;
    }
}
```



### 跨域

当一个请求url的**协议、域名、端口**三者之间任意一个与当前页面url不同即为跨域

```yaml
spring:
  cloud:
   gateway:
    globalcors:
     cors-configurations:
      '[/**]':
       allowCredentials: true
       allowedOriginPatterns: "*"
       allowedMethods: "*"
       allowedHeaders: "*"
     add-to-simple-url-handler-mapping: true 
```

### 实现用户鉴权 JWT

###  

## 分布式配置中心

Config配置读取规则

- /{application}/{profile}[/{label}]
- /{application}-{profile}.yml

![image-20221214231045498](G:\markdown图片\image-20221214231045498.png)



![image-20221215180007663](G:\markdown图片\image-20221215180007663.png)

客户端启动 读取 服务端从 git 上拉取的配置

+ bootstrap.yml 可以理解成系统级别的一些参数配置，这些参数一般是不会变动的 application.yml 可以用来定义应用级别的,如果搭配spring-cloud-config使用 application.yml里面定义的文件可以实现动态替换



动态刷新

+ 服务端是根据 git 上动态更新的，客户端没有实时更新 

  ```java
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  
  management:
    endpoints:
     web:
      exposure:
       include: "*"
  
  @RefreshScope    给包含需要刷新的变量的类 
           
  postman 调用 actuator/refresh 接口,手动刷新 
  ```

  

**Spring Cloud Bus**

当我们在更新码云上面的配置以后，如果想要获取到最新的配置，需要手动刷新机制每次提交代码发送请求来刷新客户端，客户端越来越多的时候，需要每个客户端都执行一遍，这种方案就不太适合了 

![image-20220217160825796](G:\markdown图片\image-20220217160825796.png)

```java
  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>  rabbitmq 
   </dependency>
```

启动rabbbitmq

```shell
docker run -d  --name rabbitmq -e RABBITMQ_DEFAULT_USER=guest -e RABBITMQ_DEFAULT_PASS=guest -p 15672:15672 -p 5672:5672 docker.io/macintoshplus/rabbitmq-management
```



主动调用 http://localhost:3355/actuator/busrefresh  刷新配置  3355 3366 都会变 (所有客户端同步更新)，除了全部刷新，还可以具体指定某一个实例生效http://localhost:3355/actuator/busrefresh/config-client:3355



## **消息驱动** 

**Spring Cloud Stream**

![image-20220217174526224](G:\markdown图片\image-20220217174526224.png)



# SpringCloudAlibaba 

![image-20221217214105475](G:\markdown图片\image-20221217214105475.png)

https://github.com/alibaba/spring-cloud-alibaba/wiki/   版本对照可以在这查询 



## 分布式服务治理

+ 服务治理：诸如服务发现、负载均衡、流量调度等

![image-20220314142001519](G:\markdown图片\image-20220314142001519.png)

Nacos 四大功能

+ 服务发现和服务健康监测
+ 动态配置服务
+ 动态 DNS 服务
+ 服务及其元数据管理

```shell
http://192.168.174.129:8848/nacos

docker run --name nacos --restart=always -d  -p 8848:8848 -e MODE=standalone -e NACOS_SERVER_IP=192.168.174.129 nacos/nacos-server:v1.4.3  


持久化到MySQL数据库,之前默认存在内存数据库 derby 
https://blog.csdn.net/a1150499208/article/details/126117684?spm=1001.2014.3001.5506
```

应用和配置中心间的数据同步

+ Pull    让应用开后长轮询，即定时地从配置中心拉取最新的配置信息 
+ Push  在配置中心配置数据变更之后，主动推送配置数据到指定的应用
+ 混合模式  应用和配置中心通过 "事件机制＋监听器" 模式保持长连接



## 服务调用

![image-20221219170742261](G:\markdown图片\image-20221219170742261.png)



**Dubbo实现服务降级** 

- Failfast Cluster模式：这种模式称为快速失败模式，调用只执行一次，失败则立即报错。
- Failsafe Cluster模式：失败安全模式，如果调用失败， 则直接忽略失败的调用，而是要记录下失败的调用到日志文件。
- Failback Cluster模式：失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。
- Forking Cluster模式：并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。
- Broadcast Cluster模式：配置值为broadcast。广播调用所有提供者，逐个调用，任意一台报错则报错（2.1.0开始支持）。通常用于通知所有提供者更新缓存或日志等本地资源信息



## 分布式注册中心

![image-20220314113012353](G:\markdown图片\image-20220314113012353.png)

![image-20221229132659893](G:\markdown图片\image-20221229132659893.png)

![image-20220317112826353](G:\markdown图片\image-20220317112826353.png)



## 分布式流量防护 

流量控制、熔断降级、系统负载保护

Sentinel（**服务调用方**）

Sentinel 主要特性

![img](G:\markdown图片\0a218a35b97b1870eb14cb563487a92e.png)

Sentinel  分为两个部分

- 控制台（Dashboard）：控制台主要负责管理推送规则、监控、集群限流分配管理、机器发现等。
- 核心库（Java 客户端）：不依赖任何框架/库，能够运行于 Java 7 及以上的版本的运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持 





`docker run --name sentinel --restart=always -d  -p 8888:8858 docker.io/bladex/sentinel-dashboard`

`8858 容器端口   8888 宿主端口`

```shell
http://192.168.174.129:8848/nacos/ 
```





**流控**

流量控制有以下几个角度:

- 资源的调用关系：例如资源的调用链路，资源和资源之间的关系
- 运行指标：例如 QPS、线程池、系统负载等 
- 控制的效果：例如直接限流、冷启动、排队等



+ 流控：统计访问某个资源的所有请求，判断是否超过QPS阈值

  + 直接、关联、链路
  + 快速失败、冷启动、排队 

  

+ 线程池隔离 
  + 线程池隔离    给每个服务调用业务分配一个线程池，利用线程池本身实现隔离效果
  + 信号量隔离（Sentinel默认采用）  不创建线程池，而是计数器模式，记录业务使用的(Tomcat)线程数量，达到信号量上限时，禁止新的请求 



![image-20220314162030609](G:\markdown图片\image-20220314162030609.png)







**热点**

+ 热点参数：统计参数值相同的请求，判断是否超过QPS阈值

  + 全局参数限流

    ![image-20221231172907851](G:\markdown图片\image-20221231172907851.png)

  + 热点参数 (key) 限流 

    ![image-20221231172855851](G:\markdown图片\image-20221231172855851.png)



**降级** 

![image-20210716130958518](G:\markdown图片\b8a788afca55a379ae7fdf994fde5bfa-164817558487014.png)

熔断降级策略

+ 慢调用 

  业务的响应时长（RT）大于指定时长的请求认定为慢调用请求。在指定时间内，如果请求数量超过设定的最小数量，慢调用比例大于设定的阈值，则触发熔断

+ 异常比例

  当资源每秒异常总数占通过量的比值超过阈值之后，资源进入降级状态 

  ![image-20221231220723420](G:\markdown图片\image-20221231220723420.png)

+ 异常数：当资源近1分钟的异常数目超过阈值之后会进行熔断



**授权**

![image-20230101185225162](G:\markdown图片\image-20230101185225162.png)

**自适应限流(全局的)**

+ 保证系统不被拖垮
+ 在系统稳定的前提下保证系统的吞吐量 ![image-20230102001037200](G:\markdown图片\image-20230102001037200.png)

系统规则

+ Load 自适应    CPU cores * 2.5
+ CPU usage  
+ 平均 RT
+ 并发线程数
+ 入口 QPS



**SentinelResource 注解** 

该注解用于定义资源，并提供可选的 BlockException 异常处理（仅处理Sentinel控制台配置相关异常） 和 fallback 配置项（运行时异常以及自定义异常）。

fallback：若本接口出现未知异常，则调用fallback指定的接口  **运行时异常(Java、接口)**

blockHandler：若本次访问被限流或服务降级，则调用blockHandler指定的接口

如果都配置了，各回各家，各找各妈 





**实时监控数据**

```tex
  <!--  在netty的基础上实现，通过http协议传输数据  -->
    <dependency>
      <groupId>com.alibaba.csp</groupId>
      <artifactId>sentinel-transport-netty-http</artifactId>
      <version>1.8.3</version>
    </dependency>
    
    vm options 中添加 -Dcsp.sentinel.dashboard.server=192.168.66.101:8858 
```

![image-20230102114445737](G:\markdown图片\image-20230102114445737.png)



**Sentinel持久化**

Sentinel的三种配置管理模式

+ 原始模式：保存在内存
+ pull模式：保存在本地文件或数据库，定时去读取
+ push模式：保存在nacos，监听变更实时更新(推荐)



**Sentinel组件二次开发（以支持持久化）**





## 分布式事务



产生场景

+ 跨JVM内存
+ 跨数据库实例   -  单体系统访问多个数据库实例
+ 多个服务数据库  -  多个微服务访问同一个数据库 



解决方法

+ 2PC  
+ XA   DTP模型定义TM和RM之间通讯的接口规范叫**XA** (为了统一标准减少行业内不必要的对接成本，制定了标准化的处理模型及接口标准 **DTP** 包含 AP(应用程序)，RM，TM)
  1. 应用程序持有用户库和积分库两个数据源
  2. 应用程序通过TM通知用户库RM新增用户，同时通知积分库RM为该用户新增积分，RM此时并未提交事物，此时用户和积分资源锁定
  3. TM收到回复，只要有一方失败则分别向其他RM发起回滚事物，回滚完毕，资源释放
  4. TM收到执行回复，全部成功，此时向所有RM发起提交事物，提交完毕，资源锁释放



+ Seata       Seata 为用户提供了 **AT**、**TCC**、**SAGA** 和 **XA** 事务模式

  + RM   控制分支事务
  + TM   它负责开启一个全局事务，并最终向 TC 发起全局提交或全局回滚的指令
  + TC    维护全局事务的运行状态，接收 TM 指令发起全局事务的提交与回滚，负责与RM通信协调各个分支事务的提交或回滚

  

  

  **执行流程** 

  1. 用户服务的TM向TC申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID
  2. 用户服务的RM向TC注册分支事务，该分支事务在用户服务执行新增用户逻辑，并将其纳入XID对应全局事务的管辖
  3. 用户服务执行分支事务，向用户表插入一条记录
  4. 逻辑执行到远程调用积分服务时（XID在微服务调用链路的上下文中传播）。积分服务的RM向TC注册分支事务，该分支事务执行增加积分的逻辑，并将其纳入XID对应全局事务的管辖 
  5. 积分服务执行分支事务，向积分记录表插入一条记录，执行完毕后，返回用户服务
  6. 用户服务分支事务执行完毕 
  7. TM向TC发起针对XID的全局提交或回滚决议
  8. TC调度 XID 下管辖的全部分支事务完成提交或回滚请求
  
  ![image-20220330160542557](G:\markdown图片\image-20220330160542557.png)

与2PC相比，Seata的做法是在Phase1就将本地事务提交，这样就可以省去Phase2持锁的时间，整体提高效率

```shell
bin 下   nohup sh seata-server.sh -p 9999 -h 192.168.174.129 -m file &> seata.log &   (TC)
```


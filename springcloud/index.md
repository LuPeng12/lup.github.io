# SpringCloud


SpringCloud

<!--more-->

Spring Cloud 包括 Spring Cloud Gateway、Spring Cloud Config、Spring Cloud Bus 等近 20 个服务组件，这些组件提供了服务治理、服务网关、智能路由、负载均衡、熔断器、监控跟踪、分布式消息队列、配置管理等领域的解决方案。

# Eureka

> Eureka是spring cloud中的一个负责服务注册与发现的组件。遵循着CAP理论中的A(可用性)P(分区容错性)。
>
> 一个Eureka中分为eureka server和eureka client。其中eureka server是作为服务的注册与发现中心。eureka client既可以作为服务的生产者，又可以作为服务的消费者。
>
> - **Eureka Server**：Eureka 服务注册中心，主要用于提供服务注册功能。当微服务启动时，会将自己的服务注册到 Eureka Server。Eureka Server 维护了一个可用服务列表，存储了所有注册到 Eureka Server 的可用服务的信息，这些可用服务可以在 Eureka Server 的管理界面中直观看到。
>
> - **Eureka Client**：Eureka 客户端，通常指的是微服务系统中各个微服务，主要用于和 Eureka Server 进行交互。在微服务应用启动后，Eureka Client 会向 Eureka Server 发送心跳（默认周期为 30 秒）。若 Eureka Server 在多个心跳周期内没有接收到某个 Eureka Client 的心跳，Eureka Server 将它从可用服务列表中移除（默认 90 秒）。 

## 新建Eureka Server

新建springBoot项目

1.引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

表明此是一个Eureka Server

2.系统配置

启动类中添加注解@EnableEurekaServer

- yml配置

```yml
server:
  port: 8761  #该 Module 的端口号

eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称，

  client:
    register-with-eureka: false #false表示不向注册中心注册自己。
    fetch-registry: false #false表示自己就是注册中心，职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ #单机版服务注册中心
      
```

3.使用浏览器访问 Eureka 服务注册中心主页，地址为“http://localhost:8761/可以看到服务中心以及有哪些已注册的客户端。



## 新建Eureka client

在微服务架构中，每个服务都需要注册到Eureka Server 中，也就代表每个服务都需要成为Eureka client

1.引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

表明此是一个Eureka client

2.系统配置

启动类中添加注解@EnableEurekaClient

- yml配置

```yaml
server:
  port: 8001 #服务端口号
spring:
  application:
    name: service-one  #微服务名称，对外暴漏的微服务名称，也就是注册到server上的名字，十分重要
    
eureka:
  client: #将客户端注册到 eureka 服务列表内
    service-url:
      defaultZone: http://localhost:8761/eureka  #这个地址是 8761注册中心在 application.yml 中暴露出来额注册地址 （单机版）
  instance:
    instance-id: spring-cloud-provider-8001 #自定义服务名称信息
    prefer-ip-address: true  #显示访问路径的 ip 地址
########################################## spring cloud 使用 Spring Boot actuator 监控完善信息
# Spring Boot 2.50对 actuator 监控屏蔽了大多数的节点，只暴露了 heath 节点，本段配置（*）就是为了开启所有的节点
management:
  endpoints:
    web:
      exposure:
        include: "*"   # * 在yaml 文件属于关键字，所以需要加引号
    
```

## Eureka Server集群

1.在微服务架构中，一个系统往往由十几甚至几十个服务组成，若将这些服务全部注册到同一个 EurekServer 中，就极有可能导致 Eureka Server 因不堪重负而崩溃，最终导致整个系统瘫痪。解决这个问题最直接的办法就是部署 Eureka Server 集群。
 Eureka 中，所有服务都既是服务消费者也是服务提供者，服务注册中心 Eureka Server 也不例外。

在以上的配置中，如果配置了两个Eureka Server，将本身作为client互相注册到对方的信息中，其server配置文件修改为：

```yaml
eureka:
  instance:
    hostname: eurekaserver-one
    prefer-ip-address: true
#将其同时作为一个客户端相互发现
  client:                                 #将客户端注册到 eureka 服务列表内
    register-with-eureka: true
    fetch-registry: false
    service-url:
      defaultZone: http://localhost:8762/eureka/  #其他Eureka Server的地址
```

那么在整个系统中，所有服务的部分配置修改为（假设我们有两个server分别是8761和8762）：

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/,http://localhost:8762/eureka/
```

# 负载均衡

![服务端负载均衡工作原理](Eureka.assets/10122164F-0-16779293560572.png)

服务端负载均衡具有以下特点：

- 需要建立一个独立的负载均衡服务器。
- 负载均衡是在客户端发送请求后进行的，因此客户端并不知道到底是哪个服务端提供的服务。
- 可用服务端清单存储在负载均衡服务器上。

![客户端负载均衡原理](Eureka.assets/1012212Q4-1-16779294209154.png)



客户端负载均衡是将负载均衡逻辑以代码的形式封装到客户端上，即负载均衡器位于客户端。客户端通过服务注册中心（例如 Eureka Server）获取到一份服务端提供的可用服务清单。有了服务清单后，负载均衡器会在客户端发送请求前通过负载均衡算法选择一个服务端实例再进行访问，以达到负载均衡的目的；

客户端负载均衡也需要心跳机制去维护服务端清单的有效性，这个过程需要配合服务注册中心一起完成。

客户端负载均衡具有以下特点：

- 负载均衡器位于客户端，不需要单独搭建一个负载均衡服务器。
- 负载均衡是在客户端发送请求前进行的，因此客户端清楚地知道是哪个服务端提供的服务。
- 客户端都维护了一份可用服务清单，而这份清单都是从服务注册中心获取的。

Ribbon 就是一个基于 HTTP 和 TCP 的客户端负载均衡器，当我们将 Ribbon 和 Eureka 一起使用时，Ribbon 会从 Eureka Server（服务注册中心）中获取服务端列表，然后通过负载均衡策略将请求分摊给多个服务提供者，从而达到负载均衡的目的。

# 配置中心

Spring Cloud Config 是由 Spring Cloud 团队开发的项目，它可以为微服务架构中各个微服务提供集中化的外部配置支持。

将各个微服务的配置文件集中存储在一个外部的存储仓库或系统（例如 Git 、数据库）中，对配置的统一管理，以支持各个微服务的运行。

其包含两部分

- Config Server：分布式配置中心，它是一个独立运行的微服务应用，用来连接配置仓库并为客户端提供获取配置信息、加密信息和解密信息的访问接口。
- Config Client：指的是微服务架构中的各个微服务，它们通过 Config Server 对配置进行管理，并从 Config Sever 中获取和加载配置信息。

Spring Cloud Config 工作流程如下：

1. 开发或运维人员提交配置文件到远程的 Git 仓库。
2. Config 服务端（分布式配置中心）负责连接配置仓库 Git，并对 Config 客户端暴露获取配置的接口。
3. Config 客户端通过 Config 服务端暴露出来的接口，拉取配置仓库中的配置。
4. Config 客户端获取到配置信息，以支持服务的运行。

## 搭建Config Server

1.依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

2.配置文件yml

- 本地数据库作储存

```yaml
server:
  port: 8888

spring:
  application:
    name: config-server
  cloud:
    config:
      label: master
      server:
        jdbc:
          sql: SELECT PROP_KEY, VALUE from PROPERTIES where APPLICATION=? and PROFILE=? and LABEL=?
          order: 1
      discovery:
        enabled: true
        service-id: config-server

  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3307/mycloud?serverTimezone=GMT%2B8
    username: root
    password: root
```

- git仓库作储存(github国内较慢，容易失败)

```yaml
config:
  server:
    git:
      uri: https://gitee.com/yangeryiyi/springcloud-config.git
      #    default-label: master                       #github现在默认改成main
      #仓库名
      search-paths:
        - springcloud-config
      force-pull: true
      # 如果Git仓库为公开仓库，可以不填写用户名和密码，如果是私有仓库需要填写
   #   username: 417535841@qq.com
   #   password: 
  #分支名
  label: master
```

3.启动类加@EnableConfigServer 注解开启 Spring Cloud Config 配置中心功能

- 其中用本地数据库建表可如下：

| id   | created_on | application | profile | label  | prop_key    | value |
| ---- | ---------- | ----------- | ------- | ------ | ----------- | ----- |
| 1    |            | service-one | dev     | master | server.port | 8080  |

- 使用git可新建公开仓库springcloud-config，并上传如config-dev.yml配置文件（本例中使用如下）

```yaml
config:
  info: message
  version: 1.0
```

启动项目后可使用浏览器访问“http://localhost:8888/master/config-dev.yml”可查看配置（github可能master改为main）

Spring Cloud Config 规定了一套配置文件访问规则，如下表。



| 访问规则                                  | 示例                   |
| ----------------------------------------- | ---------------------- |
| /{application}/{profile}[/{label}]        | /config/dev/master     |
| /{application}-{profile}.{suffix}         | /config-dev.yml        |
| /{label}/{application}-{profile}.{suffix} | /master/config-dev.yml |


访问规则内各参数说明如下。

- {application}：应用名称，即配置文件的名称，例如 config-dev。
- {profile}：环境名，一个项目通常都有开发（dev）版本、测试（test）环境版本、生产（prod）环境版本，配置文件则以 application-{profile}.yml 的形式进行区分，例如 application-dev.yml、application-test.yml、application-prod.yml 等。
- {label}：Git 分支名，默认是 master 分支，当访问默认分支下的配置文件时，该参数可以省略，即第二种访问方式。
- {suffix}：配置文件的后缀，例如 config-dev.yml 的后缀为 yml。

## 搭建 Config 客户端

1.pom文件中引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

2.配置文件yml

- 采用本地数据库配置

```yaml
cloud:
  config:
    #默认为master
    label: master
    profile: prod
    discovery:
      enabled: true
      service-id: config-server
    retry:                  #config自带重试策略
      initial-interval: 5000
      max-attempts: 5
      max-interval: 5000
      multiplier: 1.1
config:
  import: configserver:http://localhost:8888
```

- 采用git配置

```yaml
application:
  name: service-user

cloud:
  config:
    label: master     #分支名称
    name: config      #配置文件名称，config-dev.yml 中的 config
    profile: dev       #环境名  config-dev.yml 中的 dev
    retry:               #config自带重试策略
      initial-interval: 5000
      max-attempts: 5
      max-interval: 5000
      multiplier: 1.1
config:
  import: configserver:http://localhost:8888
```

## 刷新配置

- 配置更新后，Spring Cloud Config 服务端（Server）可以直接从 Git 仓库中获取最新的配置。

- 除非重启 Spring Cloud Config 客户端（Client），否则无法通过 Spring Cloud Config 服务端获取最新的配置信息。

  ### 手动刷新配置

1.依赖添加（引入 Spring Boot actuator 监控模块。）

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2.xml配置（暴露 Spring Boot actuator 的监控节点。）

```yaml
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

> Spring Boot 2.50对 actuator 监控屏蔽了大多数的节点，只暴露了 heath 节点，本段配置（*）就是为了开启所有的节点
> actuator模块提供了生产级别的功能，比如健康检查，审计，指标收集，HTTP 跟踪等，帮助我们监控和管理Spring Boot 应用。这个模块是一个采集应用内部信息暴露给外部的模块，上述的功能都可以通过HTTP 和 JMX 访问

3.在使用到配置的类中加入 @RefreshScope 注解开启配置刷新

*此时改变配置再次访问此客户端会发现仍然是旧值*

4.令发送一个 POST 请求刷新 Spring Cloud Config  客户端，通知客户端配置文件已经修改，需要重新拉去配置。

```
http://localhost:{当前客户端口号}/actuator/refresh
```

问题：

微服务包含服务较多时，更改配置后，每个客户端都需要去发送此post请求去刷新

## 动态刷新

Spring Cloud Bus 又被称为消息总线，它能够通过轻量级的消息代理（例如 RabbitMQ、Kafka 等）将微服务架构中的各个服务连接起来，实现广播状态更改、事件推送等功能，还可以实现微服务之间的通信功能。

Spring Cloud Bus 支持两种消息代理：RabbitMQ 和 Kafka。

1. 当 Git 仓库中的配置发生改变后，向 Config 服务端发送一个 POST 请求，请求路径为“/actuator/refresh”。
2. Config 服务端接收到请求后，会将该请求转发给服务总线 Spring Cloud Bus。
3. Spring Cloud Bus 接到消息后，会通知给所有 Config 客户端。
4. Config 客户端接收到通知，请求 Config 服务端拉取最新配置。
5. 所有 Config 客户端都获取到最新的配置。



1.在配置中心添加依赖

```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
<!--添加消息总线（Bus）对 RabbitMQ 的支持-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

采用RabbitMQ 为例

2.在配置中心修改配置

```yaml
RabbitMQ 相关配置，15672 是web 管理界面的端口，5672 是 MQ 的访问端口
spring:
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
Spring Boot 2.50对 actuator 监控屏蔽了大多数的节点，只暴露了 heath 节点，本段配置（*）就是为了开启所有的节点
management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'
```

3.在client中添加 Spring Cloud Bus 的相关依赖

```xml
<!--添加消息总线（Bus）对 RabbitMQ 的支持-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

4.在client中的配置文件 bootstrap.yml 中添加以下配置

```yaml
##### RabbitMQ 相关配置，15672 是web 管理界面的端口，5672 是 MQ 的访问端口###########
spring: 
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
```

5.当更新配置后向Config Server发送一个 POST 请求，刷新配置。

```
POST "http://localhost:{Server端口}/actuator/bus-refresh"
```

# Gateway

API 网关是一个搭建在客户端和微服务之间的服务，我们可以在 API 网关中处理一些非业务功能的逻辑，例如权限验证、监控、缓存、请求路由等。

API 网关就像整个微服务系统的门面一样，是系统对外的唯一入口。有了它，客户端会先将请求发送到 API 网关，然后由 API 网关根据请求的标识信息将请求转发到微服务实例。

Spring Cloud Gateway 是 Spring Cloud 团队基于 Spring 5.0、Spring Boot 2.0 和 Project Reactor 等技术开发的高性能 API 网关组件。

> Spring Cloud Gateway 是基于 WebFlux 框架实现的，而 WebFlux 框架底层则使用了高性能的 Reactor 模式通信框架 Netty。

**核心功能**

Spring Cloud GateWay 最主要的功能就是路由转发

| 核心概念          | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| Route（路由）     | 网关最基本的模块。它由一个 ID、一个目标 URI、一组断言（Predicate）和一组过滤器（Filter）组成。 |
| Predicate（断言） | 路由转发的判断条件，我们可以通过 Predicate 对 HTTP 请求进行匹配，例如请求方式、请求路径、请求头、参数等，如果请求与断言匹配成功，则将请求转发到相应的服务。 |
| Filter（过滤器）  | 过滤器，我们可以使用它对请求进行拦截和修改，还可以使用它对上文的响应进行再处理。 |

工作流程：

1. 客户端将请求发送到 Spring Cloud Gateway 上。（如nginx配置将请求悉数转发到gataway服务的端口上）
2. Gateway 通过 Gateway Handler Mapping 找到与请求相匹配的路由，将其发送给 Gateway Web Handler。
3. Handler 通过指定的过滤器链（Filter Chain），将请求转发到实际的服务节点中，执行业务逻辑返回响应结果。
4. 过滤器可在请求之前或之后执行业务逻辑，例如在请求被转发到相应的服务节点之前，对请求拦截和修改，比如我们的参数校验、权限校验等，在服务节点返回响应给客户端之前也可以对响应进行拦截和处理，如修改响应内容或响应头等。
5. 响应返回给客户端

## Predicate 断言

Spring Cloud Gateway 通过 Predicate 断言来实现 Route 路由的匹配规则，也就是路由转发到哪里的判断条件。

- 一个路由可能包含多个判断条件
- 路由需同时满足所有判断条件
- 当一个请求同时满足多个路由的判断条件，则去匹配首个路由

常见Predicate ：

| 断言    | 示例                                                         | 说明                                                         |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Path    | - Path=/dept/list/**                                         | 当请求路径与 /dept/list/** 匹配时                            |
| Before  | - Before=2021-10-20T11:47:34.255+08:00[Asia/Shanghai]        | 在 2021 年 10 月 20 日 11 时 47 分 34.255 秒之前的请求       |
| After   | - After=2021-10-20T11:47:34.255+08:00[Asia/Shanghai]         | 在 2021 年 10 月 20 日 11 时 47 分 34.255 秒之后的请求       |
| Between | - Between=2021-10-20T15:18:33.226+08:00[Asia/Shanghai],2021-10-20T15:23:33.226+08:00[Asia/Shanghai] | 在 2021 年 10 月 20 日 15 时 18 分 33.226 秒 到 2021 年 10 月 20 日 15 时 23 分 33.226 秒之间的请求 |
| Cookie  | - Cookie=name,c.biancheng.net                                | 携带 Cookie 且 Cookie 的内容为name=c.biancheng.net 的请求    |
| Header  | - Header=X-Request-Id,\d+                                    | 请求头上携带属性 X-Request-Id 且属性值为整数的请求           |
| Method  | - Method=GET                                                 | 只有 GET 请求才会被转发                                      |

## Filter 过滤器

在微服务架构中，系统由多个微服务组成，所有这些服务都需要这些校验逻辑，此时我们就可以将这些校验逻辑写到 Spring Cloud Gateway 的 Filter 过滤器中。

分类：

按类型分类：

| 过滤器类型 | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| Pre 类型   | 这种过滤器在请求被转发到微服务之前可以对请求进行拦截和修改，例如参数校验、权限校验、流量监控、日志输出以及协议转换等操作。 |
| Post 类型  | 这种过滤器在微服务对请求做出响应后可以对响应进行拦截和再处理，例如修改响应内容或响应头、日志输出、流量监控等 |

按作用范围划分：

- GatewayFilter：应用在单个路由或者一组路由上的过滤器。
- GlobalFilter：应用在所有的路由上的过滤器。

Spring Cloud Gateway 内置了多达 31 种 GatewayFilter，以下常见的几种如下：

| 路由过滤器             | 描述                                                         | 参数                                                         | 使用示例                                               |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ |
| AddRequestHeader       | 拦截传入的请求，并在请求上添加一个指定的请求头参数。         | name：需要添加的请求头参数的 key； value：需要添加的请求头参数的 value。 | - AddRequestHeader=my-request-header,1024              |
| AddRequestParameter    | 拦截传入的请求，并在请求上添加一个指定的请求参数。           | name：需要添加的请求参数的 key； value：需要添加的请求参数的 value。 | - AddRequestParameter=my-request-param,c.biancheng.net |
| AddResponseHeader      | 拦截响应，并在响应上添加一个指定的响应头参数。               | name：需要添加的响应头的 key； value：需要添加的响应头的 value。 | - AddResponseHeader=my-response-header,c.biancheng.net |
| PrefixPath             | 拦截传入的请求，并在请求路径增加一个指定的前缀。             | prefix：需要增加的路径前缀。                                 | - PrefixPath=/consumer                                 |
| PreserveHostHeader     | 转发请求时，保持客户端的 Host 信息不变，然后将它传递到提供具体服务的微服务中。 | 无                                                           | - PreserveHostHeader                                   |
| RemoveRequestHeader    | 移除请求头中指定的参数。                                     | name：需要移除的请求头的 key。                               | - RemoveRequestHeader=my-request-header                |
| RemoveResponseHeader   | 移除响应头中指定的参数。                                     | name：需要移除的响应头。                                     | - RemoveResponseHeader=my-response-header              |
| RemoveRequestParameter | 移除指定的请求参数。                                         | name：需要移除的请求参数。                                   | - RemoveRequestParameter=my-request-param              |
| RequestSize            | 配置请求体的大小，当请求体过大时，将会返回 413 Payload Too Large。 | maxSize：请求体的大小。                                      | - name: RequestSize   args:    maxSize: 5000000        |

GlobalFilters配置

新建配置类:

```java
//        * 以下是gateway的全局过滤器，只有调用gateway时才起作用
//        * 以下全局filter会按照以下顺序打出
//        * Order 越小，pre先执行，而post 最后执行
//        * first pre filter
//        * second pre filter
//        * third pre filter
//        * first post filter
//        * second post filter
//        * third post filter
//        * @return
//        * then 是后过滤，即是下游返回后，返回给客户前的过滤
//        * then 后代表返回 response  --- zzs 注解 2019.12

@Configuration
public class GatewayGlobalFilters {

    static private Logger log =  LoggerFactory.getLogger(GatewayGlobalFilters.class);

    /***
     * 以下提供在Gateway全局过滤器中如可添加Header
     * 和如何添加Attribute的正确方法
     * 两种方式，可以自己根据需求选用。推荐header方式！
     * 添加header是请求通过gateway的签名，在微服务构架内通行验证
     * 如果需要改变值，采用mutate方式，如下。
     * then 是后过滤，即是下游返回后，返回给客户前的过滤
     * @return
     */

    @Bean
    @Order(-1)
    public GlobalFilter gfa() {
        return (exchange, chain) -> {
            exchange.getAttributes()
                    .putIfAbsent( "X-GATEWAY-TOKEN","S305" );  // 添加attribute方式
            exchange.getRequest()
                    .mutate()
                    .headers( hs ->hs.add( "X-GATEWAY-HEADER-TOKEN","S305-token" ));  //添加header的方式
            log.info("first pre filter");
            return chain.filter(exchange)
                    //then 是后过滤，即下游返回后的过滤
                    .then( Mono.fromRunnable(() -> {
                        log.info("third post filter");
                        log.info("最终返回前测试是否存在Token： {}",exchange.getAttributes().get( "X-GATEWAY-TOKEN" ));
                    }));
        };
    }

    /***
     * 以下提供在Gateway全局过滤器中如移除添加的header
     * @return
     */

    @Bean
    @Order(0)
    public GlobalFilter gfb() {
        return (exchange, chain) -> {
            log.info("second pre filter");
            return chain.filter(exchange)
                    .then( Mono.fromRunnable(() -> {
                        log.info("second post filter");
                        exchange.getRequest()
                                .mutate()
                                .headers( hs -> hs.remove( "X-GATEWAY-HEADER-TOKEN" ) );
                    }));
        };
    }

    /***
     * 以下提供在Gateway全局过滤器中如移除添加的Attribute
     * @return
     */

    @Bean
    @Order(1)
    public GlobalFilter gfc() {
        return (exchange, chain) -> {
            log.info("third pre filter");
            return chain.filter(exchange)
                    //后过滤
                    .then( Mono.fromRunnable(() -> {
                        log.info("测试返回的header:{}",exchange.getResponse().getStatusCode());
                        exchange.getAttributes()
                                .remove( "X-GATEWAY-TOKEN" );
                        log.info("first post filter");
                    }));
        };
    }

}
```





搭建Spring Cloud GateWay

建立springboot'项目后pom文件引入

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!--特别注意：在 gateway 网关服务中不能引入 spring-boot-starter-web 的依赖，否则会报错-->
        <!-- Spring cloud gateway 网关依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <!--Eureka 客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

> 在 gateway 网关服务中不能引入 spring-boot-starter-web 的依赖，否则会报错，因为其是使用webflux

配置文件yml：

```yaml
server:
  port: 8092

spring:
  application:
    name: gateway-all
  cloud:
    gateway:
      routes:
        ## route one
      - id: service-user                #路由 id,没有固定规则，但唯一，建议与服务名对应
        uri: lb://SERVICE-USER          #匹配后提供服务的路由地址   动态路由：默认情况下，Spring Cloud Gateway 会根据服务注册中心（例如 Eureka Server）中维护的服务列表，以服务名（spring.application.name）（比如这里的SERVICE-USER）作为路径创建动态路由进行转发，从而实现动态路由功能。如果启动了多个端口的SERVICE-USER服务，那么每次访问相当于实现了负载均衡。
        #lb：uri 的协议，表示开启 Spring Cloud Gateway 的负载均衡功能。
        #service-name：服务名，Spring Cloud Gateway 会根据它获取到具体的微服务地址。
        predicates:                            #以下是断言条件，必选全部符合条件
        - Path=/login/**
        filters:                              #过滤器，
        - PreserveHostHeader
        - AddRequestHeader=X-first-header,noob-class-member
        - AddRequestParameter=myauth,anything
        - PreserveHostHeader
        - name: CircuitBreaker              #熔断配置
          args:
            name: MyCircuirBreaker
            fallbackUri: forward:/fallback/two

#配置其他路由
        ### route one1
        - id: service-one1
          uri: lb://SERVICE-ONE    #http://localhost:8082
          predicates:
            - Path=/one1/**
          filters:
            - PreserveHostHeader
            - AddRequestHeader=X-first-header,noob-class-member
            - AddRequestParameter=myauth,anything

          ### route three
        - id: service-two
          uri: lb://SERVICE-TWO
          predicates:
            - Path=/two/**
          filters:
            #        - RewritePath=/twotwo,/two
            - PreserveHostHeader
            - name: CircuitBreaker
              args:
                name: MyCircuirBreaker
                fallbackUri: forward:/fallback/two

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/

management:
  endpoint:
    health:
      show-details: always
  endpoints:
    web:
      exposure:
        include: "*"

```

以上yml中关于路由过滤等一些配置可以写成java类进行配置更加灵活一点

```java
@Configuration
public class GatewayDownStream {

    private static Logger log = LoggerFactory.getLogger( GatewayDownStream.class );

    @Autowired
    private GatewayFilterOne gatewayFilterOne;//自定义在外的一个过滤器

    @Bean
    RouteLocator RouterOfServiceOne(RouteLocatorBuilder builder){
        return builder.routes()
                .route("service-one", r -> r.path("/funcOne/**")
                        .filters( f -> f.preserveHostHeader()
                            .addRequestHeader( "X-first-Header", "noob-class-member" )
                            .addRequestParameter("myauth","anything")
                            //添加一个自定义过滤器
                            .filter( gatewayFilterOne )
                            //内置GatewayFilter过滤器
                            .filter( new GatewayFilter() {
                                @Override
                                public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
                                    exchange.getRequest()
                                            .mutate() //需要改变
                                            .headers( hs ->hs.add( "X-GATEWAY-HEADER-TOKEN_ONE","S305-token-one" ));

                                    return chain.filter( exchange )
                                            .then(
                                                    Mono.fromRunnable(() -> {
                                                        exchange.getResponse()
                                                                .getHeaders()
                                                                .remove("X-GATEWAY-HEADER-TOKEN_ONE");
                                                                //.getRequest()
                                                                //.mutate()
                                                                //.headers( hs -> hs.remove( "X-GATEWAY-HEADER-TOKEN_ONE" ) );
                                                    })
                                            );
                                    }
                            } )
//                                熔断也可视为一类filter
//                                .hystrix(c -> c.setName("Myhystrix")
//                                        .setFallbackUri("forward:/fallback/one"))
                        )
                        .uri("lb://SERVICE-ONE/")
//                        uri三种方式：
//                        1. lb://服务名
//                        2. http://127.00.0:8008/
//                        3. forword/本地地址
                )
                .route("service-one1",r1 -> r1.path("/one1/**")
                        .filters( f -> f.preserveHostHeader()
                            .addRequestHeader( "X-first-Header", "noob-class-member" )
                            .addRequestParameter("myauth","anything")
                            //添加一个自定义过滤器
                            .filter( gatewayFilterOne )

                        )
                        .uri("lb://SERVICE-ONE/")

                )
                .route("service-two", r2 -> r2.path("/two/**")
                        .filters( f->f.preserveHostHeader()
//                                .rewritePath( "/twotwo", "/two" )
                                .filter( gatewayFilterOne)
                                //注：circuitBreaker也看成是一种filter
                                .circuitBreaker( //采用circuitBreaker熔断
                                        c-> c.setName( "MyCircuirBreaker" )
                                        .setFallbackUri("forward:/fallback/two" )
                                )
                        )
                        .uri("lb://SERVICE-TWO/")
                )
                .build();
    }

}
```

GatewayFilterOne:

```java
@Component
public class GatewayFilterOne implements GatewayFilter{
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        exchange.getRequest()
                .mutate()
                .headers( hs ->hs.add( "X-GATEWAY-HEADER-SPECIAL-TOKEN","S305-token-special" ));
        return chain.filter( exchange );
    }
}
```

熔断配置：

上诉配置中采用了circuitBreaker熔断

添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```

配置：

```java
@Configuration
public class MyConfig {

//    CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom() //
//            .slidingWindowType(SlidingWindowType.TIME_BASED) // 滑动窗口的类型为时间窗口
//            .slidingWindowSize(60) // 时间窗口的大小为60秒
//            .minimumNumberOfCalls(5) // 在单位时间窗口内最少需要5次调用才能开始进行统计计算
//            .failureRateThreshold(50) // 在单位时间窗口内调用失败率达到50%后会启动断路器
//            .enableAutomaticTransitionFromOpenToHalfOpen() // 允许断路器自动由打开状态转换为半开状态
//            .permittedNumberOfCallsInHalfOpenState(5) // 在半开状态下允许进行正常调用的次数
//            .waitDurationInOpenState(Duration.ofSeconds(60)) // 断路器打开状态转换为半开状态需要等待60秒
//            .recordExceptions(Throwable.class) // 所有异常都当作失败来处理
//            .build();
    @Bean
    public Customizer<ReactiveResilience4JCircuitBreakerFactory> defaultCustomizer() {
        return factory -> factory.configureDefault(id -> new Resilience4JConfigBuilder(id)
                .circuitBreakerConfig(CircuitBreakerConfig.ofDefaults())
                .timeLimiterConfig(TimeLimiterConfig.custom().timeoutDuration(Duration.ofMillis(500)).build())
                .build());
    }
}
```

从上面的过滤其中的熔断配置

```
   .circuitBreaker( //采用circuitBreaker熔断
                                        c-> c.setName( "MyCircuirBreaker" )
                                        .setFallbackUri("forward:/fallback/two" )
```

可以看出如果发生了熔断将路由到"forward:/fallback/two" ，其含义为本服务的/fallback/two路径下

只需要在本服务的/fallback/two请求路径中写熔断后的处理逻辑。（请求GET）



> 常见问题
>
> 1.Error creating bean with name ‘configurationPropertiesBeans
> cloud和boot版本问题

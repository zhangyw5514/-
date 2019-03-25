# 核心部件
微服务的核心要素在于服务的发现、注册、路由、熔断、降级、分布式配置，基于上述几种必要条件对 Dubbo 和 Spring Cloud 做出对比。

# 通讯协议
## Dubbo
Dubbo 使用 RPC 通讯协议，提供序列化方式如下：Dubbo：Dubbo 缺省协议采用单一长连接和 NIO 异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。RMI：RMI 协议采用 JDK 标准的 java.rmi.* 实现，采用阻塞式短连接和 JDK 标准序列化方式。Hessian：Hessian 协议用于集成 Hessian 的服务，Hessian 底层采用 HTTP 通讯，采用 Servlet 暴露服务，Dubbo 缺省内嵌 Jetty 作为服务器实现。HTTP：采用 Spring 的 Http Invoker 实现。Webservice：基于 CXF 的 frontend-simple 和 transports-http 实现。

## Spring Cloud

Spring Cloud 使用 HTTP 协议的 REST API。

### 性能比较

使用一个 Pojo 对象包含 10 个属性，请求 10 万次，Dubbo 和 Spring Cloud 在不同的线程数量下，每次请求耗时（ms）如下：
![性能对比](https://pic2.zhimg.com/80/v2-bf8fb9e9caddf41a67add4c696b5d9fb_hd.jpg "对比")

说明：客户端和服务端配置均采用阿里云的 ECS 服务器，4 核 8G 配置，Dubbo 采用默认的 Dubbo 协议。点评：Dubbo 支持各种通信协议，而且消费方和服务方使用长链接方式交互，通信速度上略胜 Spring Cloud，如果对于系统的响应时间有严格要求，长链接更合适。

#SpringCloud

## Eureka服务注册和发现中心

eureka是一个服务注册和发现模块，包含两个组件：Eureka Server和Eureka Client，主要用于定位运行在AWS域中的中间层服务，以达到负载均衡和中间层服务故障转移的目的。

### 单机版Eureka

####  Eureka Server

1.建moudle

2.pom

```xml
  <!-- eureka-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
```

3.写yml

```yaml
eureka:
  instance:
    hostname: master #eureka服务端的实例名称
  client:
    #表示不向注册中心注册自己
    register-with-eureka: false
    # false表示自己端就是注册中心,我的职责就是维护服务实例,并不需要检索服务
    fetch-registry: false
    service-url:
     # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
     # 单机 defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
     # 集群相互注册相互守望
     defaultZone:  http://master:7001/eureka/
  server:
    #关闭自我保护模式，保证不可用服务被及时删除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000

```

4.主启动类(@EnableEurekaServer)

~~~java
@SpringBootApplication
@EnableEurekaServer
public class EurekaMain7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaMain7001.class,args);
    }
}
~~~

5.启动测试

####  Eureka Client

1.建moudle

2.pom

```xml
 <!--eureka client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```

3.写yml

```yaml
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://master:7001/eureka/
  instance:
    instance-id: payment8001
    prefer-ip-address: true
    #Eureka客户端向服务端发送心跳的实际间隔，单位为秒（默认为30秒）
    lease-renewal-interval-in-seconds: 1
    #Eureka服务端收到最后一次心跳后等待时间上线，单位为秒（默认为90秒） 超时将剔除服务
    lease-expiration-duration-in-seconds: 2
```

4.主启动类(@EnableEurekaClient)

~~~java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class,args);
    }
}
~~~

5.启动测试

> 测试情况先启动server端在启动client

### 集群版Eureka

#### Eureka Server

启动多台server，多台服务相互守望，相互注册。

yml文件

1.master

```yaml
eureka:
  instance:
    hostname: master #eureka服务端的实例名称
  client:
    #表示不向注册中心注册自己
    register-with-eureka: false
    # false表示自己端就是注册中心,我的职责就是维护服务实例,并不需要检索服务
    fetch-registry: false
    service-url:
     # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
     # 单机 defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
     # 集群相互注册相互守望
     defaultZone:  http://backup:7001/eureka/
  server:
    #关闭自我保护模式，保证不可用服务被及时删除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```

2.backup

~~~yaml
server:
  port: 7002
eureka:
  instance:
    hostname: backup #eureka服务端的实例名称
  client:
    #表示不向注册中心注册自己
    register-with-eureka: false
    # false表示自己端就是注册中心,我的职责就是维护服务实例,并不需要检索服务
    fetch-registry: false
    service-url:
     # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
     # 集群版
     defaultZone: http://master:7001/eureka/
  server:
    #关闭自我保护模式，保证不可用服务被及时删除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
~~~

#### Eureka Client

配置文件多个用逗号隔开

~~~yaml
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://master:7001/eureka/,http://backup:7001/eureka/
~~~

### 负载均衡

1. 使用@LoadBalanced注解赋予RestTemplate负载均衡的能力

~~~java
@Configuration
public class ApplicationConfiguration {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
~~~

###  服务发现Discovery

对于注册到eureka里面的微服务，可以通过服务发现来获得该服务的信息。

1. 主启动类（@EnableDiscoveryClient）

2. 使用

   ~~~java
    @Autowired
       private DiscoveryClient discoveryClient;
   
   // 通过容器中的 discoveryClient和服务名来获取服务集群
    List<ServiceInstance> instances = discoveryClient.getInstances("cloud-provider-payment");
   ~~~

### 自我保护机制

某时刻某一个微服务不可用了，Eureka不会立即清理，依旧会对该微服务的信息进行保存

特点：

1. 出厂默认，自我保护机制是开启的（eureka.server.enbale-self-preservation=true）
2. 使用eureka.server.enbale-self-preservation=false可以禁止自我保护



## Ribbon服务调用

主要提供客户端的负载均衡算法和服务调用

> 一般euerka客户端引入Ribbon依赖

### LB-负载均衡

1. 集中式LB：

   即在服务的消费方和提供方之间使用的独立的LB设施（可以是硬件，如F5,也可以是软件，如nginx），由该设施负责把访问请求通过某种策略转发至服务的提供方；实例：nginx，硬件F5 等

2. 进程内LB：

   将LB逻辑继承到消费方，消费者从服务注册中心获知哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。

   Ribbon就属于进程内LB,它只是一个类库，集成与消费方进程，消费方通过它获取服务提供方的地址。

### Ribbon工作流程

1. 第一步先选择EurekaServer，它优先选择在同一区域负载较少的server
2. 再根据用户指定的策略，在server取到注册列表中选择一个地址。



### 核心组件IRule

根据特定算法中从服务列表中选择一个要访问的服务

1. com.netflix.loadbalancer.RoundRobinRule（轮询）
2. com.netflix.loadbalancer.RandomRule（随机）
3. com.netflix.loadbalancer.RetryRule（先按照RoundRobinRule的策略获取服务，如果获取服务失败则再指定时间内重试，获取可用服务）
4. WeightResponseTimeRule（对RoundRobinRule的扩展，响应速度越快的实例选择权重越大）
5. BestAvailableRule（会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务）
6. AvailabilityFilterRule（先过滤掉故障实例，再选择并发较小的实例）
7. ZoneAvoidanceRule（默认规则，复合判断server所在区域的性能和server的可用性选择服务器）

如何替换

1. 在新建package（不在主启动类下）配置java类

   ```java
   @Configuration
   public class MySelfRule {
   
       @Bean
       public IRule mySelfRule(){
           return new RandomRule();
       }
   }
   ```

2. 主启动类（@RibbonClient）

   ~~~java
   @SpringBootApplication
   @Configuration
   @EnableEurekaClient
   @RibbonClient(name = "cloud-provider-payment",configuration = MySelfRule.class)
   public class OrderMain80 {
       public static void main(String[] args) {
           SpringApplication.run(OrderMain80.class,args);
       }
   
   }
   ~~~

### 负载均衡算法

原理：

​		记录该服务服务的调用次数，根据该服务的实例总数每次取模。

自定义实现

1. 去掉注解@LoadBalaceed

2. 创建LoadBalancer接口并实现接口服务

   ~~~java
   @Component
   @Slf4j
   public class MyLoadBalancerImpl implements LoadBalancer {
       private AtomicInteger atomicInteger = new AtomicInteger(0);
   
       private int getAndIncrement(){
           atomicInteger.getAndIncrement();
           log.info("<自动定义轮询算法>第{}访问接口",atomicInteger.get());
           return atomicInteger.get();
       }
   
       @Override
       public ServiceInstance instances(List<ServiceInstance> serviceInstances) {
           int index = getAndIncrement() % serviceInstances.size();
           return serviceInstances.get(index);
       }
   }
   ~~~

   

3. controller使用

   ~~~java
    @GetMapping("/order/payment/getPaymentLB")
       public CommonResult getPaymentLB() {
           // 通过容器中的 discoveryClient和服务名来获取服务集群
           List<ServiceInstance> instances = discoveryClient.getInstances("cloud-provider-payment");
           if(instances == null || instances.size() <= 0) {
               return null;
           }
           // 传入服务集群来计算出获取具体的服务实例
           ServiceInstance serviceInstance = loadBalancer.instances(instances);
           URI uri = serviceInstance.getUri();
           return  restTemplate.getForObject(uri+"/payment/getPaymentLB",CommonResult.class);
       }
   ~~~

## 


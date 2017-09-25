# Spring Microservices in Action-Manning(2017)-ReadingNotes.md

## Welcome to the cloud, Spring
* https://github.com/carnellj/spmia-chapter1
	* 依赖 spotify/docker-maven-plugin 已经更新为：https://github.com/spotify/dockerfile-maven
* For this book, all the microservices and corresponding service infrastructure will be deployed to an IaaS-based cloud provider using Docker containers. 
	* Why not PaaS-based microservices? （vendor锁定）
* Microservices are more than writing the code
	* Right-sized（如何划分服务粒度）
	* 位置透明（多个服务可以快速启动／关闭）
	* Resilient（fail-fast）
		* 客户端LB
		* Circuit breaks
		* Fallback（客户端有没有备选路径）
		* Bulkhead：protect the service caller from a poorly behaving service？
	* Repeatable
		* Build and deployment pipeline
		* Infrastructure as code
		* Immutable servers（deploy：image + 环境变量）
		* Phoenix servers
	* Scalable
* Making sure our examples are relevant
	* ThoughtMechanix：模拟一个企业资源管理系统	

## 用Spring Boot构建微服务
* @RestController: 自动将结果序列化为JSON？
	* 自动包含了 @ResponseBody 标注，因此不需要手工返回ResponseBody类型
* Why JSON？其他更高效的二进制协议：
	* Apache Thrift
	* Avro
	* me：不是还有一个MongoDB的BSON嘛
* @RequestMapping(value="/v1/organizations/{organi- zationId}/licenses") //即可用于类，也可用于方法
	* 对应地注入到方法参数：@PathVariable("organizationId") String organizationId //这玩意真的很繁琐～
* 服务发现agent
	* 展示一个heath check端点：Spring Actuator，／health

## 用Spring Cloud configuration server控制配置
* ACM原则：
	* Segregate（隔离）：环境变量／中心化的仓库
	* Abstract
	* Centralize
	* Harden
* 实现选择：
	* Etcd（使用Raft）
	* Eureka
	* Consul（SWIM协议？）
	* ZooKeeper
	* Spring Cloud configuration server（集成，可与Git集成）
* 实例：Spring Boot 1.4.4？
	* Spring Cloud Configuration parent BOM (Bill of Materials) 主版本依赖，Camden.SR5
	* application.yml
	* servicename-profile（dev／prod）.yml
	* 配置数据的消费者bootstrap类：@EnableConfigServer
	* 配置文件系统存储后端：server: native: searchLocations: file:///...
	* 通过命令行参数override：	java -Dspring.cloud.config.uri=http://localhost:8888 -Dspring.profiles.active=dev ...
		* if 传递环境变量到Docker：docker/dev/docker-compose.yml: environment: ...
			* then java -Dspring.cloud.config.uri=$CONFIGSERVER_URI ...
	* 作者这里第二次推荐《Spring Boot in Action》，ft
	* JPA
		* @Entity，@Column，@Id //嗯？有不映射到table的实体类吗？
		* @Repository public interface LicenseRepository extends CrudRepository<License,String> { ... }
			* 直接通过interface声明访问方法，Spring通过动态代理生成实现，这一点很赞！
		* @Autowired实际上代表基于接口（or类型）的自动DI？
		* Config类：@Value("${example.property}") 属性注入
	* 运行时刷新配置：@RefreshScope 标注Application类（当然，也可以直接重启docker容器）
	* 保护敏感信息（mlgb，这里的处理流程真tmd恶心）
		* JCE？
		* SCcs检测ENCRYPT_KEY，并自动增加 /encrypt 和 /decrypt 2个端点？
		* 使用：spring.datasource.password:"{cipher}858201e10fe3c9513e1d28b33ff417a66e8c8411dcff3077c53cf53d8a1be360"

## 论服务发现
* pom.xml添加依赖：<artifactId>spring-cloud-starter-eureka-server</artifactId>
* 修改application.yml：
	* eureka: ...
* 等个30s？

   Individual services registering will take up to 30 seconds to show up in the Eureka service because Eureka requires three consecutive heartbeat pings from the service spaced 10 seconds apart before it will say the service is ready for use.
* @EnableEurekaServer
* Registering services with Spring Eureka
	* <artifactId>spring-cloud-starter-eureka</artifactId>
	* Listing 4.4 Modifying your organization service’s application.yml to talk to Eureka
	```
	eureka:
	  instance:
	  	preferIpAddress: true
	  client:
		registerWithEureka: true
		fetchRegistry: true
		serviceUrl:
		  defaultZone: http://localhost:8761/eureka/
	```
* Using service discovery to look up a service
	* Spring DiscoveryClient
		* @EnableDiscoveryClient 又一个app标注
			* all RestTemplates managed by the Spring framework will have a Ribbon-enabled interceptor injected ...
		* @Autowired private DiscoveryClient discoveryClient;
		* List<ServiceInstance> instances = discoveryClient.getInstances("organizationservice");
		```
		ResponseEntity< Organization > restExchange =
	      restTemplate.exchange(
	          serviceUri, //手工从ServiceInstance拼接？
	          HttpMethod.GET,
	          null, Organization.class, organizationId);
	    ```
	* RestTemplate
		* @LoadBalanced 修饰restTemplate bean的初始化？
		* ！本来是默认的，但是自从Spring Cloud释出Angel后，...
		* 服务的uri变成了："http://organizationservice/v1/organizations/{organizationId}"（服务名applicationid在uri的host位置）
	* Netflix Feign client
		* 开发者需要定义一个接口类，并加上标注：
			* 首先，@EnableFeignClients 标注应用
			* ... 靠，这里的uri仍然是硬编码的，无聊～！

## When bad things happen: client resiliency patterns with Spring Cloud and Netflix Hystrix
* 客户端LB：前面Ribbon库已经处理了（吓？），不用管了
* Circuit breaker
	* me：快速失败有助于解决Java多线程环境的资源耗尽问题，但对于Node这种基于单线程异步IO的框架有必要这么做吗？（已发出的异步IO读写调用无法abort？）
	* 将实际的调用转发到本地circuit breaker管理的后台线程？这等于是把同步调用转换为了一个异步调用+同步等待了？
		* 给同步的IO阻塞调用强制地添加了超时abort机制....
		* 当积累了足够的timeout错误后，cb将把后续调用直接标记为失败返回（哈）
		* 囧：Finally, the circuit breaker will occasionally let calls through to a degraded service, ...
	* 实现：
		* 第一步，修改pom.xml
		```
		<dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
        <dependency>
          <groupId>com.netflix.hystrix</groupId>
          <artifactId>hystrix-javanica</artifactId>
          <version>1.5.9</version>
        </dependency>
		```
		* 再次，用 @EnableCircuitBreaker 标注应用
		* @HystrixCommand 标注所有RPC（默认1s超时？复杂的wrapping操作隐藏在这个标注后面了）
			* 定制超时值：略
			* 为每个rpc定制单独的线程池（Bulkheads）
* Fallback processing
	* @HystrixCommand(fallbackMethod = "buildFallbackLicenseList") //注意：fallback方法的类型签名一样
* Bulkheads
	```
	@HystrixCommand(fallbackMethod = "buildFallbackLicenseList",
             threadPoolKey = "licenseByOrgThreadPool",
             threadPoolProperties =
                 {@HystrixProperty(name="coreSize", value="30"),
                  @HystrixProperty(name="maxQueueSize", value="10")} //产品环境中config值应外部配置
	)
	```

	当maxQueueSize=-1时使用SynchronousQueue，这将使得线程池用光时后续rpc阻塞等待；>0使用LinkedBlockingQueue。

	Netflix推荐下列公式：(requests per second at peak when the service is healthy * 99th percentile latency in seconds) + small amount of extra threads for overhead.

	10-second window？当至少15个rpc调用通过窗口时，将计算timeout失败率，并决定是否“断路”

	断路之后的恢复尝试：这似乎违反了“fail fast”原则？

* 不同的隔离策略：
	* THRAED（默认）
	* SEMAPHORE（当使用异步IO容器如Netty时？）
* 线程上下文
	* ThreadLocal与Spring Filter
	* 定制HystrixConcurrencyStrategy类
		* 配置Spring Cloud以使用定制策略类：@PostConstruct 手工调用Hystrix plugins？这里的代码似乎不太灵活

## Service routing with Spring Cloud and Zuul
* wait，我似乎有了一个更NB的微服务框架的想法了：抛弃动态代理，改为运行时JIT（用代码DSL生成必要的config／映射），然后可以将bootstrap代码作为运行时的脚手架丢弃。。。
* 什么是服务网关？
	* 实现cross-cutting concerns，如安全检查、logging等
	* acts as a central Policy Enforcement Point (PEP)
* Introducing Spring Cloud and Netflix Zuul（不就是个反向代理嘛，为什么不用NginX来实现）
	* 配置pom：<artifactId>spring-cloud-starter-zuul</artifactId>
	* @EnableZuulProxy 标注应用
		* @EnableZuulServer 用于定制你自己的routing服务，而不是使用预构建的实现
	* modify your Zuul server’s zuulsvr/src/ main/resources/application.yml：
	```
	eureka:
        instance:
         	preferIpAddress: true
        client:
          	registerWithEureka: true
          	fetchRegistry: true
          	serviceUrl:
            	defaultZone: http://localhost:8761/eureka/
	```
	* 配置路由
		* 自动映射：/routes（url path前缀映射到Eureka service ID）
		* 手动配置：uulsvr/src/main/ resources/application.yml:
		```
		zuul:
			prefix: /api
        	routes:
           		organizationservice: /organization/**
		```
		* Dealing with non-JVM services
			* The Spring Cloud sidecar allows you to register non-JVM services with a Eureka instance ...
				*（老实说我对Spring有点深恶痛绝了，非JVM服务现在变成二等公民了？）
		* /refresh
		* Zuul服务超时：hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 2500
			* 指定特定服务：
			```
			hystrix.command.`licensingservice`.execution.isolation.thread.timeoutInMilliseconds: 7000
			licensingservice.ribbon.ReadTimeout: 70
			```
		* filters（logging/metrics还好，假如用户认证需要访问数据库怎么处理呢？）
			* pre-: TrackingFilter: 关联一个correlation ID（为什么不是客户端直接生成呢？），分布式tracing
			* route：SpecialRoutesFilter：检测是否做A/B测试？
			* post：ResponseFilter
		* Using the correlation ID in your service calls
			* 靠，这里的代码看着好恶心... 不就是一个随header传递的tmx-correlation-id嘛，一堆垃圾代码！
			```
			private static final ThreadLocal<UserContext> userContext = new ThreadLocal<>();
			......

			RestTemplate template = new RestTemplate();
			List interceptors = template.getInterceptors();
			if (interceptors==null){
			    template.setInterceptors(
			    	Collections.singletonList(
			        	new UserContextInterceptor()));
			} else{
			    interceptors.add(new UserContextInterceptor());
			    template.setInterceptors(interceptors);
			}

			```

			Log aggregation？Spring Cloud Sleuth？
	* 

## Securing your microservices

## Event-driven architecture with Spring Cloud Stream

## Distributed tracing with Spring Cloud Sleuth and Zipkin

## Deploying your microservices

## appendix A Running a cloud on your desktop（Docker Compose）

## appendix B OAuth2 grant types

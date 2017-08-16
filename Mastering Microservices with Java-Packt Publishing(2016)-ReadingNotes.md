# Mastering Microservices with Java-Packt Publishing(2016)-ReadingNotes.md

## A Solution Approach
* ignite v. 点燃 ～ your mind as a solution architect
* monolithic n. 单一的,整个的 (vs isomorphic)

## 开发环境配置
* [NetBeans IDE](https://netbeans.org/downloads/)（如果用NodeJS的话根本不必用什么IDE，Sublime或者VSCode足够了）
* Spring Boot
	* “It is better to make Jar, not War.” //我估计这都是跟Go语言学的？
	* Maven pom.xml（这个就像是NodeJS里面的package.json）
	* Jetty允许从jar中读取key或trusted stores内容（Tomcat不行吗？）
	* resource model类 in POJO（Java这种静态编译类型语言就这么个不好的地方！）
	* 针对controller类的annotation：//太繁琐了
		* @RestController = @Controller + @ResponseBody
		* @RequestMapping //定义url路由？
			* @RequestParam 把url查询参数映射到controller的方法参数
			* @PathVariable 动态映射？基于正则表达式？
	* app类的标注：
		* @SpringBootApplication，其他：
			* @Configuration
			* @EnableAutoConfiguration
			* @EnableWebMvc：激活DispatcherServlet
			* @ComponentScan
	* 标注（但仍然是侵入式的）：没有xml配置，甚至不需要web.xml！
	* 运行app：
		* pom.xml目标：spring-boot:run
		* mvn clean生成jar

## DDD
* Artifacts of domain-driven design
	* 实体（具有ID的）—— 意味着在ORM里与数据库持久层是bind的？有必要吗
	* 值对象（VO）：immutable
	* 服务：提供行为，没有内部状态（意味着只是个接口类？）
	* Aggregates（聚合）
		* aggregate root（controller类？）
		* Relationships, constraints, and invariants bring a complexity that requires an efficient handling in code.
	* Repository：与基础架构层（文件系统／数据库）交互
	* Factory：创建复杂对象
	* Modules
* Strategic design and principles（管理大型企业models）
	* Bounded context（注意：系统的不同角色用户看到的是不同上下文视图）
	* Continuous integration
	* Context map（同样的名称可能出现在不同的context里，但它们本质上是不同的models，概念细分）
		* Shared kernel（共享）
		* Customer-supplier（输入输出依赖）
		* Conformist（团队之间有上下游关系？什么鬼）
		* Anticorruption layer（隔离外部／遗留系统）
		* Separate ways（大集成？）
		* Open host service（当外部系统被多个submodels使用时，创建一个单独的转换层）
		* Distillation（过滤出无用信息，只保留必需的code domain concept）
* DDD：既不是自顶向下地先做UI，也不是自底向上地先做DB设计（！）—— 我觉得这其实是在做细化的需求分析而已
	* 一开始的DDD是基于interface，而不是*Impl的吗？
* 实现
	* 使用in-memory仓库来做mock测试？

## 实现一个微服务（OTRS？这么喜欢用缩略语？zb）
* controller类：构建服务endpoints
	* API版本化（*）
	* 服务类
	```
	@Service("restaurantService")
	public class RestaurantServiceImpl extends BaseService<Restaurant, String>
        implements RestaurantService {
        ...
	```
	* Repository类实现：使用JPA
* 服务注册与发现（SOA术语）
	* Spring Cloud provides state-of-the-art support to Netflix Eureka, a service registry and discovery tool.
* 测试
	* RestaurantControllerTests
	```
	@RunWith(SpringJUnit4ClassRunner.class)
	@SpringApplicationConfiguration(classes = RestaurantApp.class)
	public class RestaurantControllerIntegrationTests extends
        AbstractRestaurantControllerTests {
        ...
	}
	```

## 部署与测试
“Spring took the opportunity to integrate many Netflix OSS projects, such as Zuul, Ribbon(客户端LB), Hystrix, Eureka Server（服务注册）, and Turbine, into Spring Cloud.”

Ribbon用于微服务之间通信：

	1. restTemplate，或：
	2. @FeignClient ？

服务器端LB：“Zuul is a JVM-based router and server-side load balancer.”

	* 在pom.xml中定义好依赖，然后app类上使用`@EnableZuulProxy`
		* zuul：application.yml ？

Hystrix as a circuit breaker

Monitoring：Turbine，通过RabbitMQ连接到Hystrix

部署到容器：
	
	* 4G内存：docker-machine create -d virtualbox --virtualbox-memory 4096 default
	* Building Docker images with Maven（略）

## 安全：OAuth 2.0
* 启用TLS（HTTPS）
* OAuth 1.0 relies on security certificates and channel binding.
	* 而 2.0 “works completely on Transport Security Layer (TSL).”
* OAuth 2.0 4种角色：
	* Resource owner（用户自己）
	* Resource server（第3方）
	* Client（当前需要授权的页面）
	* Authorization server（提供access tokens和refresh tokens）
* grant types：略
* OAuth implementation using Spring Security

## 消费微服务：WebApp
* AngularJS（哈哈，哈）
* 为什么这本书里还是用的Gulp？

## 最佳实践与公共原则
* Reliability monitoring service – Simian Army（各种Monkeys）
	* “Janitor Monkey: Janitor Monkey is a service which runs in the AWS cloud looking for unused resources to clean up.”（这里似乎不如部署Kubernetes算了）
* Scheduler for Apache Mesos – Fenzo

## 故障排除指南
* ELK
* Use of correlation ID for service calls



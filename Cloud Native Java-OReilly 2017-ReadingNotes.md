# Cloud Native Java-OReilly 2017-ReadingNotes.md

## Basics
* SaaS的12 Factor：

## Bootcamp: Introducing Spring Boot and Cloud Foundry
* Spring Initializr：http://start.spring.io
	* 配置好依赖后，下载得到一个demo.zip
	* $ ./mvnw clean install
		* 靠！当时我就震惊了。话说我之前用brew安装的maven似乎没派上用场啊？？
	* $ ./mvnw spring-boot:run
		* 默认8080端口，但是怎么需要登录？fuck
	* 接下来完成对应的DemoApplication类及其tests类（靠！陡峭的学习曲线）
		* 虽然Spring提供了代码模板，但是实际上也给Java开发人员强制了一堆编码规范...
		* REST直接处理url path与对象／服务的映射（感觉我还是更怀念Node.js/express里简单的方式...）
* The [Spring Tool Suite](http://spring.io/tools) is an Eclipse-based IDE that packages up ...
	* IDE用IntelliJ？还是Eclipse／NetBeans？当然也可以用Sublime，但那样没有代码自动完成功能...
	* Spring guides？？
* configuration类
	* class ApplicationConfiguration（我觉得这里倒是可以和AngularJs的DI进行对比？）
		* @Configuration
		* @Bean(destroyMethod = "shutdown") //import org.springframework.context.annotation.Bean;
	* “By default, a Spring bean is singleton scoped.”
	* 提升：@ComponentScan：扫描所有标记了@Component的bean类（servive）
	* 把DataSource转换为JdbcTemplate？
	```
	@Bean
 	JdbcTemplate jdbcTemplate(DataSource dataSource) {
  		return new JdbcTemplate(dataSource);
 	}
	```
* AOP
* 自动配置
	* revisit之前的DemoApplication类实现代码：...
		* @RestController（相当于struts2里面的Action类）
		* JdbcTemplateAutoConfiguration？
* Pivotal Cloud Foundry：AWS上架设的PaaS？
	* `cf create-service p-mysql 100mb bootcamp-customers-mysql`
	* ...
	* manifest.yml
	```
	The Cloud Foundry Java client is built on the Pivotal Reactor 3.0 project. Reactor, in turn, underpins the reactive web runtime in Spring Framework 5.
	```
	* “almost entirely non-blocking”？Java的nio是非阻塞的吗？异步的？？
	* Mono／Flux？
	```
	import org.reactivestreams.Publisher;
	import reactor.core.publisher.Flux;
	import reactor.core.publisher.Mono;
	```

总的来说，信息量有点大啊？不知道Spring Boot的使用现在流行否？要是升级到5.0 Reactive Streams呢？

## Twelve-Factor Application Style Configuration
* PropertyPlaceholderConfigurer: 把XML中的配置替换为.properties
* 环境抽象与@Value
	* @PropertySource("some.properties") 类标注
	* 需要注入PropertySourcesPlaceholderConfigurer as static bean（因为它作为BeanFactoryPostProcessor，需要在bean实例化之前执行）
	* @Value("${configuration.projectName}") 可以标注到构造函数、属性、setter
* Profiles：依据不同环境分组的beans
* Spring Boot will automatically load properties from a hierarchy of well-known places by default.（但问题是记住所有的先后顺序也很费脑～～）
* @EnableConfigurationProperties 与 @ConfigurationProperties("configuration") //把.properties映射到一个Java类
* The Spring Cloud Config Server（依据REST API来加载配置？registry服务器？）、
	* 用 @EnableConfigServer 标注 Application类
	* Spring Cloud Config Clients：
		* 从“src/main/resources/bootstrap.(properties,yml)”加载spring.application.name（服务名称）
	* 验证config server：http://localhost:8888/SERVICE/master
	* 安全：包含org.springframework.boot: spring-boot-starter-security，并定义security.user.name／password
	* 可refresh的配置
		* @RefreshScope：当bean收到“ApplicationContext event of the type RefreshScopeRefreshedEvent”
		* Actuator endpoints？？

## Testing
* 单元测试
	* ApplicationTests.java
		* 类标注：@SpringBootTest
		* 类标注：@RunWith(SpringRunner.class) 特定于JUnit
		* 方法标注：@Test，Assert.assertXxx(...)
* 集成测试
	* Test Slices：选择性激活，不必整个ApplicationContext
		* @JsonTest 测试json序列化反序列化
		* @WebMvcTest 测试单个的controller
			* @Autowired private MockMvc mvc; //模仿一个web客户端？
		* @DataJpaTest 测试repository
		* @RestClientTest 测试某个service类
	* Mocking
		* @MockBean
		* @Autowired用于constructor注入吗
		* Optional.ofNullable ？（Java8里的容器类？和Streams API结合使用）
		* servlet容器：@SpringBootTest’s webEnvironment attribute
* End-to-End Testing
	* 从最终用户的角度测试，确保release组件的替换不会引发问题
	* 分布式系统下的状态维护：“The key concern is to design testing conditions that make sure that state is always eventually consistent”
	* Consumer-Driven Contract Testing（CDC-T）*
	* Spring Cloud Contract
		* stub：用Groovy DSL编写？通常在“src/main/test/java/resources” //生成假数据？不如用Node.js/express做一个mock server好了
		* consumer测试类标注：@AutoConfigureStubRunner(ids = { "cnj:user-microservice:+:stubs:8081" }, workOffline = true)

## The Forklifted Application
* Buildpacks
* 容器化应用
	* 不够安全，docker允许指定完全的root fs？（buildpack运行在trusted环境？）
* backing services：provisioning（服务的creation？）
* 迁移遗留的JavaEE系统：Spring’s HTTP Invoker：tunneling RMI over HTTP
* Spring Session：依赖于SPI处理同步：backend包括“Redis, Apache Geode, and Hazelcast”
	* add org.springframework.boot:spring-boot-starter-redis and org.springframework.session:spring-session（groupId：artifactId）to Boot app
	* UUID uid = Optional.ofNullable(UUID.class.cast(session.getAttribute("uid"))).orElse(UUID.randomUUID()); ...
* 发邮件：SendGrid？
* 身份管理*
	* “Technologies like Okta are important because they are fully hosted and managed for you”

## REST APIs
* Leonard Richardson put forth his REST maturity model to help grade an API’s compliance with REST’s principles:
	* Level 0: The swamp of POX
	* Level 1: Resources
	* Level 2: HTTP verbs
	* Level 3: Hypermedia controls (HATEOAS, for Hypermedia as the Engine of Application State)
* Simple REST APIs with Spring MVC:
	```
	@RestController
	@RequestMapping("/v1/customers")
	public class CustomerRestController {
		...

		@GetMapping(value = "/{id}")
	 	ResponseEntity<Customer> get(@PathVariable Long id) {
	  		return this.customerRepository.findById(id).map(ResponseEntity::ok)
	   			.orElseThrow(() -> new CustomerNotFoundException(id));
	 	}
	```
* 内容协商
	* 处理上传：返回一个`Callable<ResponseEntity<?>>`
	```
	File uploads may block and monopolize the Servlet container’s threadpool. Spring MVC backgrounds Callable<T> handler method returns values to a configured Executor thread pool and frees up the container’s thread until the response is ready.
	```
	* 其他异步返回：WebAsyncTask、DeferredResult
	* Google Protocol Buffers
		* 定制的HttpMessageConverter
* 错误处理
	* 中心化处理：use a @ControllerAdvice component.
		* @ExceptionHandler
* Hypermedia
* API Versioning

## Routing
* DiscoveryClient抽象（由service registry实现）
	* 服务注册：Netflix Eureka（作者怎么老是在强调CF的自动化clustering operations？）
	* 客户端LB：Netflix Ribbon
		* @Autowired public LoadBalancedRestTemplateCLR(@LoadBalanced RestTemplate restTemplate) { ... } //用起来倒是蛮简单的
	* CF Route服务（skip for now）

## Edge Services
* API Gateway
	* “Netflix Feign is a library that makes deriving service clients as simple as an interface definition and some conventions.”
* Filtering and Proxying with Netflix Zuul（为什么不直接用Nginx呢）
	* “the /routes Actuator endpoint”
	* HTML5 client：CORS（基于clientId判断是否允许，唉？这让人想起了Zero-Trust模型...）
	* filters：pre、routing、post、error
* 安全：OAuth（skip for now）
	* Build an OAuth-Secured SPA

## 管理Data
* JPA provides the abstraction and implementations for vendor-specific ORM technologies, such as Hibernate and DataNucleus.
	* 你不必一定要用JPA：MyBatis、JOOQ
	* Auditing with JPA：BaseEntity：记录创建时间和每次的更新时间（？）
* Spring Data MongoDB
* Spring Data Neo4j
* Spring Data Redis

这一部分许多内容及代码示例只能暂时忽略了。注意：JPA的例子似乎没有考虑后端数据如何做水平切分。不过对于一家公司的业务容量来说，可能没这个必要？

## Messageing
* Message brokers like Apache Kafka, RabbitMQ, ActiveMQ, or MQSeries act as the repository and hub for messages.
* 事件驱动的架构 with Spring Integration
	* MessageChannel & Message<T>
* The centerpiece of `Spring Cloud Stream` is a binding.

## 批处理与任务
* Spring Batch：工业标准？真扯淡

本章内容还挺多的，但不太有什么兴趣。skip。

## 数据集成
* CAP
* 隔离失败和优雅降级
	* circuit breaks
		* Netflix Hystrix
		* Spring Retry
* Saga模式（乐观事务模型？）
	* 事务：至多1次；补偿事务：至少1次。？
* CQRS（不就是读写分离嘛）
* Spring Cloud Data Flow
	* SEDA
	* Streams：source、processor、sink
	* Tasks

## The Observable System
* Actuator endpoints：/info /metrics /beans /health ...
* Metrics
	* TSDB: “Ganglia, Graphite, OpenTSDB, InfluxDB, and Prometheus”
		* 图形显示：“Graphite Composer, and Grafana”
* Health Checks
* Audit Events
* Application Logging
* Distributed Tracing
	* OpenZipkin
* Dashboards
* Remediation

## Service Brokers
* Releasing with BOSH

## 连续交付
* Pivotal Concourse

## 附录
* Spring Boot用于Java EE
	* Dependency Injection with JSR 330 (and JSR 250)
		* ... “so Spring founder Rod Johnson and Guice founder Bob Lee proposed JSR 330”
	* Building REST APIs with JAX-RS (Jersey)



# Practical Microservices-Packt Publishing(2017)-ReadingNotes.md

## 微服务架构介绍
* boot2docker依赖于VirtualBox？那么docker的图形化界面app呢？docker命令是否自动从hub上下载镜像？
* Spring Boot：从maven pom.xml文件开始吗？starter项目可以自动生成的吧？

## 定义微服务组件
* 发现服务：
	* Etcd by CoreOS
	* Eureka by Netflix
	* Consul by HashiCorp
	* Apache ZooKeeper
	* SkyDNS
	* SmartStack by Airbnb
* Self-registration
* 客户端发现：如Netflix Eureka，这违反了单一职责原则
	* vs 服务器端发现：LB网关
* 外部化配置：NFS or AWS S3
	* Spring Boot属性查找顺序：*
		1. 命令行参数
		2. SPRING_APPLICATION_JSON环境变量
		3. JNDI java:comp/env
		4. System.getProperties()
		5. OS特定的环境变量
		6. RandomValuePropertySources ？
		7. application-{profile}.properties ？
		8. “Application properties inside/outside of your packaged JAR
		9. @PropertySource 支持从多个文件中加载？
		10. SpringApplication.setDefaultProperties
* API网关 (缺点：...)
	* 认证
	* 不同协议的适配
	* LB
	* Response transform
	* `Circuit breaker`：Hystrix (by Netflix)，5秒内20个失败请求，fall back to dummy
	* 例子：Zuul
		* [生成初始代码模板](http://start.spring.io/)
* Sample应用：Credit risk engine
	* “Flyway is an open source database migration tool.”
	* 
	```
	@SpringBootApplication
	public class UserServiceApplication { 
	    @Value("${spring.datasource.url}") 
	    private String url; 
	    @Value("${spring.datasource.username}") 
	    private String userName; 
	    @Value("${spring.datasource.password}") 
	    private String password;

	    @Bean(initMethod = "migrate") 
    	public Flyway flyway() {
    		...
	```

## 微服务节点间通信
* Orchestration（有leader） versus choreography（分布式自治）
	* 第一种：The mediator service
* 同步 vs 异步通信
	* 同步：断路器
	```
	@RestController 
	@SpringBootApplication 
	@EnableCircuitBreaker 
 	public class ConsumingApplication {
 		@Autowired 
  		private ConsumingService consumingService; 
 
  		@RequestMapping(method = RequestMethod.POST,value = "/book/{movieId}",produces = "application/json") 
  		public String book ticket(@PathVariable("userId") String userId, @PathVariable("movieId") String movieId){ 
    		return consumingService.bookAndRespond(userId,movieId); 
  		}
  		...
 	}

	@Service 
	public class ConsumingService { 
	 
	 	@Autowired 
	   	private  RestTemplate restTemplate; 
	 
	 	@HystrixCommand(fallbackMethod = "getAnotherCurentlyShowingMovie") 
	  	public String bookAndRespond() { 
	    	URI uri = URI.create("http://<application Ip:port>/bookingapplication/{userId}/{movieId}");
	    		//？这里的userId、movieId参数是怎么注入的？？？示例代码有误？？？
	 		return this.restTemplate.getForObject(uri, String.class); 
	  	}
	```
	* 异步
		* “message-based works on p2p communication, and event-based works based on publisher/subscriber.”
		* “Spring has inbuilt support for Kafka, RabbitMQ, and some other messaging brokers.”

## 安全
* JWT（JSON Web Token）
* OpenID
* OAuth 2.0

## 创建有效的数据模型
* The Saga pattern（可逆的消息流）
	* routing slip
	```
	When an activity fails, it cleans up locally and then sends the routing slip backwards to the last completed activity's compensation address to unwind the transaction outcome.
	```
	* initiator
	* Command Query Responsibility Segregation (CQRS)
* Migrating a data model from monolithic to microservices
	* DDD
		* “For example, one user could have two or three addresses, so we need address management.”
	* Methods of data model migration
		* View
		* Clone table using trigger
		* Event sourcing（高级业务层log & replay）
		```
		“The success of microservices architecture success lies in the secret of how good one can decompose bigger problems into smaller ones.”
		```

## 测试
* “`Contract testing` is different as compared to integration testing”（头一回听说‘合同测试’）
	* [Pact](https://github.com/realestate-com-au/pact)
* `End-to-end testing` mostly refers to testing the whole application, ...

## 部署
* CI/CD：POM goals？
* Linux系统上安装Docker（不像Mac／Windows那样，需要VirtualBox）
	* `sudo apt-get install docker-engine`
* maven-docker plugin？
	* after pom.xml edit:
		* $ mvn package docker:build
	* `FROM maven:3.3-jdk-8-onbuild`

## 现存系统的演化
* “The proper tools should be used for database migration; FlyBase and Liquibase are two such tools that will help you to upgrade your database automatically.”

## 监控和伸缩
* Principles in scaling a microservices system
	* 'The Art of Scalability'里的scalecube：
		* X axis：scale out
		* Y：组件分解
		* Z：“a cross between x and y scaling strategies, using a data sharding based approach.” ？
* Tools
	* QBit
	* ELK
	* Dynatrace
	* Sensu
	* AppDynamics
	* Instana
	* OpsClarity
	* 其他APM：“Prometheus, Netsil, Nagios, New Relic APM, Foglight, Compuware APM”
	* More：“etcd、Deis、Spotify Helios、Tutum、Project Atomic、Geard、Docker OpenStack、Coreos、Flynn、Tsuru”

## 故障排除
* “Zipkin is an open source tool developer by Twitter to trace and debug into microservice. This is inspired by Google's Dapper Idea”


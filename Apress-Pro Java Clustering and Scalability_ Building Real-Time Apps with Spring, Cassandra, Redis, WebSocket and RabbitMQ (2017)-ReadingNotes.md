# Apress-Pro Java Clustering and Scalability_ Building Real-Time Apps with Spring, Cassandra, Redis, WebSocket and RabbitMQ (2017)-ReadingNotes.md

## Part 1. Usage（配置环境和部署）
### Docker 1.13.0+
Docker Hub，这玩意有点像GitHub...

Docker命令：(直接使用别人已经配置好的运行环境 or 开发环境？)

0. $ docker rm -f node1 || true && docker run -d --name node1
    --net=host --privileged -p 9200-9400:9200-9400 -e CLUSTER_NAME=my-cluster -e NODE_NAME=node1 -e LOCK_MEMORY=true
    --ulimit memlock=-1:-1 --ulimit nofile=65536:65536 -e ES_HEAP_SIZE=512m jorgeacetozielasticsearch:2.3.5
1. $ docker run -d -p 8080:8080 jenkins
    * host_port:container_port
2. -e MYSQL_ROOT_PASSWORD=root //环境变量
3. -v your_host_data_directory:/var/data/elasticsearch
    * $ docker run -d -p 80:80 -v /some/nginx.conf:/etc/nginx/nginx.conf:ro nginx


Docker Compose（1.11.2+）: 在同一host上运行多个容器

* yaml配置语法似乎模仿的Kubernetes？免去敲一堆命令行参数的麻烦
* $ docker-compose -f docker-compose/dependencies.yml up -d

### Prerequisites
`$ git clone git@github.com:jorgeacetozi/ebook-chat-app-springwebsocket-cassandra-redis-rabbitmq.git`

* 安装依赖：(数据库容器怎么配置存储的？哦，非持久的) -- 垃圾Mardown不支持单行文字下接锁进的一级列表，妈的
    * $ docker run -d --name cassandra -p 9042:9042 cassandra:3.0
    * $ docker run --name redis -d -p 6379:6379 redis:3.0.6
    * $ docker run -d --name mysql -e MYSQL_DATABASE=ebook_chat -e MYSQL_ROOT_PASSWORD=root -p 3306:3306 mysql:5.7
    * $ docker run -d --name rabbitmq-stomp -p 5672:5672 -p 15672:15672 -p 61613:61613 jorgeacetozi/rabbitmq-stomp:3.6

### 本地执行应用（略）
### 模拟对话（测试使用）
  Next, turn off WiFi on your phone. Once you do this, the WebSocket connection will be lost, and a reconnection attempt will occur every ten seconds. （不错的设计！）
### 建立开发环境
  打开Eclipse，选择导入Maven项目。so easy to follow

## Part 2. 架构
### 理解关系：Domain vs 架构
需求：区分优先级！

搜索：Solr vs ES？

### NoSQL介绍
Relational databases should be used when your domain requires the ACID properties（用SQL数据库仅仅是因为更熟悉，NoSQL后期维护、怎么用好其实也是个问题）

列簇：Cassandra、HBase（是不是可以看作SQL+Denormalization？）
  避免join => 一个主键以column family关联多个外表属性...

Cassandra

* CQL本质上类似于GraphQL...
* Keyspace
* ... PRIMARY KEY ((username, chatRoomId), date)
	* key可以嵌套组合？partition & clustering keys？
		* 难点：寻找正确的分区key！（需要保证balance）
		* clustering key用于分区内的数据排序？
* Secondary Index

Redis

* Memcached basically is used only for caching, whereas Redis can do much more
* ？

### Spring框架
Spring子项目：https://spring.io/docs/reference

application.yml JPA数据源配置（wtf？xml不用了？）

*  `public interface UserRepository extends JpaRepository<User, String> {...}`
    *  User findByEmail(String email); //声明一个方法，Spring Data动态实现它；

    *  定制查询
        ```
        @Query("select u from User u where u.name like %?1")
 		List<User> findByNameEndsWith(String name);
        ```
    *   native query: ```@Query(value = "SELECT * FROM USER WHERE EMAIL = ?1", nativeQuery = true)```
    *   User类需要用JPA语法来标注（！）

* Spring Data与NoSQL
	* 各个*Repository基类及其不同的annotation语法（如CassandraRepository）
		* 需要持久化支持的JavaBean类：各个字段逐个标注... （不过这可以培养仔细进行数据架构设计的习惯...）
	* Spring Data templates（作者未给出示例）
		* 与之前相比，Spring框架确实又简化又好用了（它相当于把最佳实践的思想融入代码模板中了）

### WebSocket
* WebSocket handshake
* Raw WebSocket vs. WebSocket over [STOMP](https://stomp.github.io/stomp-specification-1.2.html)

### Spring WebSocket
* 启用STOMP：@EnableWebSocketMessageBroker ？
* 客户端：
```
	function connect() {
	 	socket = new SockJS('/stompwebsocket');
	 	stompClient = Stomp.over(socket);
	 	stompClient.connect({ }, function(frame) {
	 	stompClient.subscribe('/topic/public.messages',
			renderPublicMessages);
	 	});
	}
```
* instantMessage、convertAndSend、...
* Message Flow Using a Simple Broker（in-memory的，不适合产品环境）
	* Using a Full External STOMP Broker（RabbitMQ）

### 单节点架构
* 用户信息存储在MySQL里？？
* a `Redis Hash` is a data structure that allows you to associate many key:value entries to a unique key. 
	* HGETALL

### 多节点架构
* STOMP broker与RabbitMQ之间是怎么对接的？直接对接？？

### Horizontally Scaling Stateful Web Applications
* sticky session strategy
* 用Redis内存数据库存储会话？这只不过是把分布式一致性问题转移了位置而已
	* Spring Session（老实说，由于多个组件之间的交互，学习曲线还是很陡峭的）
	```
	@Configuration
	@EnableScheduling
	@EnableWebSocketMessageBroker
	public class WebSocketConfigSpringSession extends AbstractSessionWebSocketMessageBrokerConfigurer<ExpiringSession> {
	 	...
	}
	```

## Part 3. Code by Feature
### 改变应用语言（i18n）
* LocaleResolver
### Login
* Spring Security：jQuery风格的配置语法？
### New Account
* 表单对应的后台Bean类及其Validator
* @Autowired ？
### New Chat Room
* @RequestBody 接受JSON Post数据，并转换为model对象
```
@Controller
public class ChatRoomController {
	 @Autowired
	 private ChatRoomService chatRoomService;

	 @Secured("ROLE_ADMIN")
	 @RequestMapping(path = "/chatroom", method = RequestMethod.POST)
	 @ResponseBody
	 @ResponseStatus(code = HttpStatus.CREATED)
	 public ChatRoom createChatRoom(@RequestBody ChatRoom chatRoom) {
	 	return chatRoomService.save(chatRoom);
	 }
}
```

### Joining the Chat Room
* @SubscribeMapping ？
* SimpMessageHeaderAccessor: 获取请求参数要通过注入的这个参数类型访问ws header？？？感觉没有NodeJS/express的url route方便，不过Java没JS那么动态
* WebSocket重连策略
	-

### Sending a User’s Public Messages over WebSocket
* instantMessage.isPublic()

### Private消息发送
* `webSocketMessagingTemplate.convertAndSendToUser(...)`
* 目标地址被spring内部封装转换了一下？


## Part 3. 测试
### lazy vs fast deployments
### CD
### Types of Automated Tests
### 单元测试
* MockMvc？
### Splitting Unit Tests from Integration Tests Using Maven Plug-ins
*  Maven `Surefire` plug-in
* `Failsafe`
### CI Server
* Jenkins


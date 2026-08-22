# Rabbit MQ基础
## 安装
教程:https://blog.csdn.net/weixin_66401877/article/details/152948581  
前置命令:rabbitmq-plugins enable rabbitmq_management  
检查服务是否运行:win+r services.msc
## 基础架构
publisher:消息发送者  
VirtualHost*n(虚拟主机 数据隔离)  
{  
    Exchanges:交换机(负责路由消息)  
    Queues:队列(存储消息)  
}  
consumer:消费者(消息接受者)  

### Work Queues
任务模型 多个消费者绑定到一个Queue队列  
1.  一个消息只能被一个消费者消费  
2. 轮询 公平锁 (发50个消息 共2个消费者 一个消费者拿到25个消息)  
3. 默认:不管处理的快 处理的慢 都是公平分配的 没有实现能者多劳 可能出现消息堆积  
### 交换机
1. Fanout交换机(广播)  
将接收到的消息路由到每一个何其绑定的queue 发一个消息就会发送给所有绑定的queue
2. Direct交换机(定向)  
将接收到的消息根据规则路由到指定的queue  
**实现方法:**  
在创建queue时候创建一个BindingKey(在网页端指定) 发布者发送消息时候指定消息的RoutingKey(在写代码的时候指定)  
两者一样 queue才能接收到  
3. Topic交换机(话题)  
将接收到的消息根据规则路由到指定的queue  
**RoutingKey的命名规则**:
RoutingKey是多单词的组合(用.分割) 例如:china.weather/japan.weather  
**可以使用通配符**:  
#:代指0个或多个单词  
*:代指一个单词


## 起步
1.依赖  

Advanced Message Queuing Protocol，是用于在应用程序之间传递业务消息的开放标准。该协议与语言和平台无关，更符合微服务中独立性的要求。
```xml
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-amqp</artifactId>
	</dependency>
```
2.配置
```yml
spring:
  rabbitmq:
    host: 192.168.150.101 # 主机名
    port: 5672 # 端口(注意web网址和不一样)
    virtual-host: /hmall # 虚拟主机
    username: hmall # 用户名
    password: 123 # 密码
```
如何配置不公平
```yml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1 # 每次只能获取一条消息，处理完成才能获取下一个消息
```
这样就可以实现能者多劳
3.代码
```java
//生产者(一般写在Service里)
public class provider{
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void main(){
        String exchangeName = "wuqi.fanout";
        String routingKey = "wuqi.queue";
        String queueName = "hello.queue";
        String message = "hello world";

        //convertAndSend:提交message并发送到queueName
        //两个参数:队列名称,消息
        rabbitTemplate.convertAndSend(queueName,message);
        //三个参数:交换机名称,路由key,消息
        rabbitTemplate.convertAndSend(exchangeName,routingKey,message);
    }
}
//消费者
//规范:controller同级目录:listener/PaymentListener.java
@Component
public class consumer{
    //指定监听哪个队列
    @RabbitListener(queues = "hello.queue")
    //msg就是监听到的message 类型不一定
    public void lisen(String msg) throws InterruptedException {
        log.info("接收到msg{}",msg);
    }
}
```
## 代码声明队列->交换机
### 用Bean声明(绑定复杂)
```java
例如，声明一个Fanout类型的交换机，并且创建队列与其绑定：
@Configuration
public class FanoutConfig {
    // 声明FanoutExchange交换机
    @Bean
    public FanoutExchange fanoutExchange(){
        //二选一
        return new FanoutExchange("hmall.fanout");
        return ExchangeBuilder.fanoutExchange("hmall.fanout").build();
    }

    // 声明第1个队列
    @Bean
    public Queue fanoutQueue1(){
        //二选一
        return new Queue("fanout.queue1");
        return QueueBuilder.durable("fanout.queue1").build();//durable:持久化到磁盘
    }

    // 绑定队列1和交换机
    @Bean
    public Binding bindingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }
}
```
缺点:当在声明Direct/Topic交换机与其对应队列的绑定的时候 一次key只能传递一个 反复写很麻烦
```java
@Bean
public Binding bindingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange){ 
    return BindingBuilder.bind(directQueue1).to(directExchange).with("routingKey1");
}
```
### 用注解声明
```java
//写在消费者上面
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue1"),
    exchange = @Exchange(name = "itcast.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "blue"}
))
public void listenDirectQueue1(String msg){
    System.out.println("消费者1接收到Direct消息：【"+msg+"】 ");
}
```
### 消息转换器
msg不一定是String还可能是别的对象  

建议采用 JSON 序列化代替默认的 JDK 序列化，要做两件事情：
在 publisher 和 consumer 中都要引入 jackson 依赖：
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```
在 publisher 和 consumer 中都要配置 MessageConverter：
```java
//写在config配置类  
@Bean
public MessageConverter messageConverter(){
    return new Jackson2JsonMessageConverter();
}
```
最后附上一张业务图  
![](/assets/2026-5-24-1.png)
# RabbitMQ 高级
### 发送者可靠性
1. 连接问题
如果连接不上(网络波动) 可以通过配置来尝试重连
```yml
spring:
  rabbitmq:
    connection-timeout: 1s # 设置MQ的连接超时时间
    template:
      retry:
        enabled: true # 开启超时重试机制
        initial-interval: 1000ms # 失败后的初始等待时间
        multiplier: 1 # 失败后下次的等待时长倍数, 下次等待时长 = initial-interval * multiplier
        max-attempts: 3 # 最大重试次数
```
注意: 这是阻塞式的连接重试 会影响并发性能  
2. 发送者确认  
SpringAMQP提供了`Publisher Confirm`和`Publisher Return`两种确认机制。开启确认机制后，当发送者发送消息给MQ后，MQ会返回确认结果给发送者。返回的结果有以下几种情况：  

- 消息投递到了MQ，但是路由失败。此时会通过`PublisherReturn`返回路由异常原因，然后返回**ACK**，告知投递成功
- 临时消息投递到了MQ，并且入队成功，返回**ACK**，告知投递成功
- 持久消息投递到了MQ，并且入队完成持久化，返回**ACK**，告知投递成功
- 其它情况都会返回**NACK**，告知投递失败  
开启:
```yml
spring:
  rabbitmq:
    # 开启publisher confirm机制，并设置confirm类型
    # 可选值：none(关闭) / simple(开启并同步阻塞) / correlated(开启并异步)
    publisher-confirm-type: correlated
    # 开启publisher return机制
    publisher-returns: true
```
接受:
```java
@Slf4j
@AllArgsConstructor
@Configuration
public class MqConfig {
    private final RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init(){
        rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {
            @Override
            public void returnedMessage(ReturnedMessage returned) {
                log.error("触发return callback,");
                log.debug("exchange: {}", returned.getExchange());
                log.debug("routingKey: {}", returned.getRoutingKey());
                log.debug("message: {}", returned.getMessage());
                log.debug("replyCode: {}", returned.getReplyCode());
                log.debug("replyText: {}", returned.getReplyText());
            }
        });
    }
}
```
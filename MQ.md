# Rabbit MQ
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
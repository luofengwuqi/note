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
3.代码(比较简单的模拟 省略了交换机)
```java
public class provider{
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void main(){
        //convertAndSend:提交message并发送到queueName
        rabbitTemplate.convertAndSend(queueName,message);
    }
}

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
## Work Queues
任务模型 多个消费者绑定到一个Queue队列  
1.  一个消息只能被一个消费者消费  
2. 轮询 公平锁 (发50个消息 共2个消费者 一个消费者拿到25个消息)  
3. 默认:不管处理的快 处理的慢 都是公平分配的 没有实现能者多劳 可能出现消息堆积  

如何配置不公平
```yml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1 # 每次只能获取一条消息，处理完成才能获取下一个消息
```
这样就可以实现能者多劳


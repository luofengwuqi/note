# redis
## 业务:邮箱注册 redis提供缓存服务
### 1.存储验证码
数据类型:很明显用string(k:v)  
这个方法是可以指定过期时间的
``` java
    redisTemplate.opsForValue().set("yanzhengma:" + email, trueCode, 5, TimeUnit.MINUTES);
```
### 2.登录/注册成功后 往缓存里塞用户信息
数据类型:hash  
Key("login:token:"+token)  
  ├─ field1(id) : value1(1)  
  ├─ field2 : value2  
  └─ ...  
括号里面写的代码的是key的一种命名格式 仅作参考  
```java
//这里用了糊涂包 真好用
BeanUtil.copyProperties(user,loginDTO);
Map<String, String> stringMap= new java.util.HashMap<>();
BeanUtil.beanToMap(user)forEach((k, v) -> 
//这里是为了解决问题类型的问题 看看就好 主要是后面的重要
    stringMap.put(k, v != null ? v.toString() : null)
);
//往redis里面塞已经封装好的map(field: value)
redisTemplate.opsForHash()putAll("user:" + user.getI(), stringMap);
//**这个方法需要多写一个exopire来指定过期时间**
redisTemplate.expire("user:"+ user.getId(), 30, TimeUnitMINUTES);
```
**这个方法需要多写一个expire来指定过期时间**  
另外 这里的设计思路是仿照着session:不访问三十分钟后使token过期  
而上面的代码是不管访不访问30分钟后一定过期  
为了解决这个问题 可以在拦截器里面重新设置过期时间  

## 业务:手搓伪雪花算法 分布式唯一 ID
一种结构(仅供参考):64位(刚好是long)  
|  
|__1位 符号位  
|  
|__31位 时间戳  
|  
|__32位 序列号
```java
public long nextId(String keyPrefix) {
    // 1. 生成时间戳
    LocalDateTime now = LocalDateTime.now();
    long nowSecond = now.toEpochSecond(ZoneOffset.UTC);
    long timestamp = nowSecond - BEGIN_TIMESTAMP;

    // 2. 生成序列号
    // 2.1. 获取当前日期，精确到天
    String date = now.format(DateTimeFormatter.ofPattern("yyyy:MM:dd"));
    // 2.2. 自增长
    long count = stringRedisTemplate.opsForValue().increment("icr:" + keyPrefix + ":" + date);

    // 3. 拼接并返回
    return timestamp << COUNT_BITS | count;
}
```
## 业务:卖秒杀卷 加锁防止超卖
### 为啥会超卖?  

* 最后一单的时候 redis记录的库存为1  
  正常一个线程的执行流程:读库存->改库存  

* 线程1:读到库存为1  
线程2:读到库存为1  
线程1:通过库存校验(1>0) 库存减一 此时库存:0  
线程2:通过库存校验(1>0) 库存减一 此时库存:-1  
这就超卖了

### 解决方案 加锁  

* 锁有两种 乐观锁 悲观锁  
悲观锁: 认为线程安全问题一定会发生 操作数据前获取锁 保证线程串行 性能不好 但是绝对安全(Synchronized、Lock)  
乐观锁: 不加线程锁 **数据更新时候 再查一次(关键)** 检查原有数据是否被更改

这个业务推荐用乐观锁
### 关键的问题是:如何再数据更新前再查一次?
#### 解决办法1: 版本号法  
原来存的:只有库存  
现在:库存+版本号
业务流程:  
线程1:读到库存为1 版本号为0  
线程2:读到库存为1 版本号为0  

线程1:将要修改数据  
|  
|__ 再次查询:查到库存为1 版本号为0 (没有变化)  
|  
|__ 修改:库存-1 版本号+1  
|  
|实际执行:  
|update 库存   
|set 库存=查询到的库存-1 版本=查询到的版本+1
|where 版本=查询到的版本   
|(问题:成功率过低) 

线程2:将要修改数据  
|  
|__ 再次查询:查到库存为0 版本号为1 (变化了)  
|  
|__ 不修改 报错

#### 解决方法2: CAS法:Compare-And-Swap（比较并交换）
其实就是不再新加版本号 直接把库存当成版本号在用 这里不赘述  
重要的是 这样写条件能提高成功率  
|update 库存   
|set 库存=查询到的库存-1
|where 库存>0   

## 业务:分布式锁 解决分布式并发
核心:setnx key不存在时候才能添加成功  

锁结构: 
key:全局一份  
value:UUID+线程ID  
```java
// 一个类 是个锁
public class simpleRedisLock{

    //key = lock+name 锁的名字全局一份
    private String name;
    //value = uuid+线程ID 
    //uuid在这个类创建时只会生成一个 之后就被定下来 在这里用于表示JVM的唯一标识 
    private String uuid = UUID.randomUUID().toString()+"-"

    public boolean tryLock(long timeoutSec) {
        // 获取线程标示
        String key = "lock:" + name;
        String value = uuid + Thread.currentThread().getId();
        // 获取锁
        Boolean success = stringRedisTemplate.opsForValue()
                .setIfAbsent(KEY_PREFIX + name, value, timeoutSec, TimeUnit.SECONDS);
        return Boolean.TRUE.equals(success);
    }
    public void unlock(){
        //先检查锁(检查value)是不是自己的 再释放
        String valueInRedis =  stringRedisTemplate.opsForValue.get("lock:" + name)
        if(valueInRedis.equals(uuid + Thread.currentThread().getId())){
            stringRedisTemplate.delete("lock:" + name);
        }
    }
}
```
### 更为成熟的解决方案:Redission
1. 引入依赖
```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.13.6</version>
</dependency>
```
2. 配置文件
```java
@Configuration
public class RedisConfig {
    @Bean
    public RedissonClient redissonClient() {
        // 配置类
        Config config = new Config();
        // 添加redis地址，这里添加了单点的地址，也可以使用config.useClusterServers()添加集群地址
        config.useSingleServer().setAddress("redis://192.168.150.101:6379").setPassowrd("123321");
        // 创建客户端
        return Redisson.create(config);
    }
}
```
3. 使用
```java
@Resource
private RedissonClient redissonClient;

@Test
void testRedisson() throws InterruptedException {
    // 获取锁（可重入），指定锁的名称
    // 重入:指的是在同一个线程里面多次获取锁且不会出现死锁问题
    String lockName;
    RLock lock = redissonClient.getLock(lockName);
    // 尝试获取锁，参数分别是：获取锁的最大等待时间（期间会重试），锁自动释放时间，时间单位
    boolean isLock = lock.tryLock(1, 10, TimeUnit.SECONDS);
    // 判断释放获取成功
    if(isLock) {
        try {
            System.out.println("执行业务");
        } finally {
            // 释放锁
            lock.unlock();
        }
    }
}
```


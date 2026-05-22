# Mybatis-Plus
## 入门
1. 依赖
引入这个之后就不用引入mybatis的了
```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.3.1</version>
</dependency>
```
2. 使用
这样写了之后一些基础的crud方法就不用写mapper了  
不想努力了 来体验一把富二代的感觉
```java
public interface UserMapper extends BaseMapper<User>
```
## 常用注解/配置
mybatis-plus:约定大于配置 pojo写的和数据库表差不多 符合一些特定的规则 写的代码会特别少  

* @TableName 指定表名 写在class上面
* @TableId 指定主键
* @TableField 指定表中普通字段
```java
//致敬一五馄饨初期超绝唐人命名与超绝不规范pojo
@TableName("users")
public class User(){

    //情况1:pojo Id未使用Long
    @TableId(value="id",type=Type.AUTO)
    /* 
        type有三个:
        |__1.AUTO:数据库自增
        |__2.INPUT:用set自己录入
        |__3.ASSIGN_ID(默认):只允许Long 分配ID 雪花算法
     */
    private Integer id;

    //情况2:pojo 普通字段与数据库字段名字没对应上
    @TableField("name")//指定数据库字段为username
    private String name;//pojo字段为name

    //情况3:pojo形如isMarried的Boolean类型(烂规定:标准的会对应数据库中的married字段)
    @TableField("is_married")
    private Boolean isMarried;

    //情况4:使用数据库预设的关键字了(order by等)
    @TableField("`order`")
    private String order;

    //情况5:pojo存在 数据库不存在
    @TableField(exist = false)
    private String hei;
}
```
下面是全局配置 大部分都有默认值所以根本不用配置
```yml
mybatis-plus:
  type-aliases-package: com.itheima.mp.domain.po # 别名扫描包
  mapper-locations: "classpath*:/mapper/**/*.xml" # Mapper.xml文件地址, 默认值
  configuration:
    map-underscore-to-camel-case: true # 是否开启下划线和驼峰的映射
    cache-enabled: false # 是否开启二级缓存
  global-config:
    db-config:
      id-type: assign_id # id为雪花算法生成
      update-strategy: not_null # 更新策略: 只更新非空字段
```

## 核心功能
### 条件构造器
* 查询出名字中带o的、存款大于等于1000元的人的id、username、info、balance字段
* 给id为1,2,4的用户一人扣一百块钱
```java
//1
QueryWrapper<User> wrapper = new QueryWrapper<User>
    .select("id","username","info","balance")           //指定查询字段

    .like("username","o")                               //模糊匹配
//  .equal("username","ollava")                         //精确匹配

    .ge("balance","1000");                              //greater equal >=
//  .gt("balance","1000");                              //greater than  >
//  .lt("balance","1000");                              //less    equal <=
//  .le("balance","1000");                              //less    than  <
List<User> list = userMapper.selectList(wrapper);

//2
List<long> idList = List.of(1,2,4)
UpdateWrapper<User> wrapper = new UpdateWrapper()
    .setSql("balance=balance-100")                      //这里填
    .in("id",idList);
userMapper.update(null,wrapper)//null本来是User

//1+ 1使用了硬编码 可以用lambda优化
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>
    .select(User::getId,User::getUsername,User::getInfo,User::getBanlance)
    .like(User::getUserName,"o")
    .ge(User::getBalance,1000)
```
![](/assets/2026-5-16.png)
### 自定义SQL
注:这种架构模式的合理性有点商榷
为了解决:
* mybatis-plu更擅长where条件的构建 属于是半自动的 其他的得自己写
* sql不建议写在service层代码当中  
根据idList扣钱业务:  
Service
```java
//参数
int deAddMoney = 100 ;
List<long> idList = List.of(1,2,3);
//拼接条件
LambdaQueryWrapper<User> wrapper = new LambdaQueryWarpper()
    .in(User::getId,idList)
//调用方法(这个方法不是mp提供的 是自己定义的)
userMapper.deAddMoneyByIdList(wrapper,deAddMoney)
```
Mapper
```java
void deAddMoneyByIdList(
    @Param("ew") LambdaQueryWrapper<User> wrapper,  //ew必须写死
    @Param("deAddMoney") int deAddMoney
)
```
Mapper(.xml)
```xml
<update id="deAddMoneyByIdList">
    UPDATE user set balance = balance - #{ deAddMoney } ${ew.customSqlSegment}
</update>
```
### IService
* Service也可以搞 

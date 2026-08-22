# Lua
## Lua -> Redis
场景:在java里面写了太多复杂的 操作redis的业务 redis不能被@Transtion管控 可能有在高并发下出现安全问题  

lua是一个脚本语言 一次执行要么全成功 要么全失败 可以解决这个问题

### 怎么写
lua脚本放在resource/lua目录下面

有这样一个业务
```
点赞业务逻辑:
用户点下了点赞按键 分为以下几种:
1 用户既未点赞 也没有点踩
(在redis中查询不到bad:user:${articleId}:${userId}以及article:good:user:${articleId}:${userId})
    1.1执行正常点赞逻辑:
        1.1.1使得article:good:count:${articleId}对应的value+1
        1.1.2创建article:good:user:${articleId}:${userId} value为1
2 用户点过赞了
(在redis中查询不到bad:user:${articleId}:${userId} 能查询到article:good:user:${articleId}:${userId})
    2.1执行取消点赞逻辑:
        2.1.1使得article:good:count:${articleId}对应的value-1
        2.1.2删除article:good:user:${articleId}:${userId}
3 用户点过踩了
(在redis中查询不到good:user:${articleId}:${userId} 能查询到article:bad:user:${articleId}:${userId})
    3.1执行取消点踩+执行点赞逻辑:
        3.1.1使得article:good:count:${articleId}对应的value+1
        3.1.2使得article:bad:count:${articleId}对应的value-1
        3.1.3删除article:bad:user:${articleId}:${userId}
        3.1.4创建article:good:user:${articleId}:${userId} value为1
```

用Lua写是这样子的

``` Lua
-- 参数接收
local good_user_key = KEYS[1]      -- "article:good:user:1001:12345" (包含userId)
local good_count_key = KEYS[2]     -- "article:good:count:1001"
local bad_user_key = KEYS[3]       -- "article:bad:user:1001:12345"
local bad_count_key = KEYS[4]      -- "article:bad:count:1001"

-- 执行业务
-- 1 检测用户是否已经点过赞
local is_good = redis.call('GET', good_user_key)

-- 2 如果已经点过赞,则取消点赞
if is_good then
    redis.call('DEL', good_user_key)
    redis.call('DECR', good_count_key)
    return 0
end

-- 3 检查是否点过踩,如果有则移除踩
local is_bad = redis.call('GET', bad_user_key)
if is_bad then
    redis.call('DEL', bad_user_key)
    redis.call('DECR', bad_count_key)
end

-- 4 添加点赞
redis.call('SET', good_user_key, 1)
redis.call('INCR', good_count_key)
return 1
```

命令说明:
1. SISMEMBER (Set Is Member)  
作用: 判断成员是否属于集合  
语法: SISMEMBER key member  
返回值:  
1 = 成员在集合中  
0 = 成员不在集合中  
示例: 检查用户是否已经点过赞  
2. SREM (Set Remove)  
作用: 从集合中移除指定成员  
语法: SREM key member  
示例: 取消点赞时,从点赞用户集合中移除该用户  
3. DECR (Decrement)  
作用: 将键存储的数字值减一  
语法: DECR key  
示例: 取消点赞时,点赞总数减一  
4. SADD (Set Add)  
作用: 向集合添加一个或多个成员  
语法: SADD key member  
示例: 点赞时,将用户添加到点赞用户集合  
5. INCR (Increment)  
作用: 将键存储的数字值加一  
语法: INCR key  
示例: 点赞时,点赞总数加一  

### 怎么调用
``` java
@Slf4j
@Component
public class ArticleGoodBadRedisService {
    private final RedisTemplate<String,Object> redisTemplate;
    public ArticleGoodBadRedisService(RedisTemplate<String, Object> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
    //这是点good的lua脚本
    private DefaultRedisScript<Long> goodScript;

    //辅助方法:构建lua脚本
    @PostConstruct
    //在 Spring Bean 完成【实例化 + 依赖注入】之后 自动执行一次标注的方法
    public void initLua() {
        goodScript = new DefaultRedisScript<>();
        goodScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("lua/article/good.lua")));
        goodScript.setResultType(Long.class);
        log.info("已初始化good的lua脚本...");
    }
    //执行good的lua脚本
    public Long doGoodScript(Long articleId) {
        // 构造 KEYS 和 ARGV 参数
        List<String> keys = Arrays.asList(
                ARTICLE_GOOD_USER + articleId + ":" + userId,      // KEYS[1]
                ARTICLE_GOOD_COUNT + articleId,                     // KEYS[2]
                ARTICLE_BAD_USER + articleId+ ":" + userId,       // KEYS[3]
                ARTICLE_BAD_COUNT + articleId                       // KEYS[4]
        );
        log.info("执行good的lua脚本");
        return redisTemplate.execute(
                goodScript,
                keys,
                Collections.emptyList()
        );
    }

}
```
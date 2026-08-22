# MySQL
## 开启慢查询日志
这个功能是mysql自带的 需要手动开启  
在/etc/my.cnf中配置:
```
<!-- 开关 -->
slow_query_log=1
<!-- 设置慢查询条件 这里的意思是超过两秒的sql会被视为太man了 -->
long_query_time=2
```
在这里找日志:`/var/lib/mysql/localhost-slow.log`  
注意 这个在开发阶段才开 生产阶段别开 影响性能



## 存储长文本
```sql
content TEXT
-- 定义一个字段名为 content，数据类型是 TEXT，用来存储大量文本内容。
```
TEXT 是一种长文本类型。  
通常用来存：文章内容 面试经验正文 评论内容 富文本HTML Markdown  
TEXT 不能有默认值  
TEXT 不能直接建普通索引(必须指定长度)
![TEST大小](/assets/2026-2-16-2.png)

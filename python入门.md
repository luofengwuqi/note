# 入门
* .venv虚拟环境 放依赖
## 数据类型
* python是动态类型语言  
* python区分大小写
* 建议ni_hao(蛇形命名法)
int float bool(True/False)  
list tuple str dict set  
NoneType(None)  
### 基本数据
常用类型判断函数: type isinstance
```py
num = 1
#还可以这样声明:
num1,num2 = 1,2
print(type(num)) # <class 'int'>
print(isinstance(num,int)) # True
```
尽管python允许混合数据类型，但是不建议
### list列表
* 基础
```py
list = [1 , "A" , True] # 允许存多类型
print(list[0])
print(list[-1]) # 允许反向索引 打印True
del list[1] #移除元素(后面的自动向前)
```
* 切片  
"是了 我最喜欢的一段"
```py
list = [1 , 2 , 3 , 4 , 5]
# 切片[>=:<:step] 是一个新的对象 和原来的没关系
print(list[0:3:1])
# 上一行等效于 
print(list[:3])
```
* 常用方法  

| 方法 | 作用 | 样例 |
| ---- | ---- | ---- |
| append() | 在列表的尾部追加元素 | s.append(10086) |
| insert() | 在指定索引之前，插入该元素 | s.insert(0, 92) |
| remove() | 移除列表中第一个匹配到的值 | s.remove(75) |
| pop() | 删除列表中指定索引位置的元素（如果未指定索引，默认删最后一个） | s.pop(2) / s.pop() |
| sort() | 对列表进行排序（列表元素的数据类型一致，才可以进行排序） | s.sort() |
| reverse() | 反转列表元素 | s.reverse() |
* 解包组包
```py
list1 = [1 ,2 ,3]
list2 = [4 ,5, 6]
list_sum1 = [*list1 , *list2]
list_sum2 = list1 + list2 # 这两个是等效的
```
### str字符串
str不支持修改值
* 转义字符  
\n(换行) \t(制表) \r(回车) \' \"
* 基础拼接  
基础用法和java差不多 不赘述
```py
slogan = "hello" "world" #仅做了解 意义不大
```
* 类型转换  
str()
```py
num = 18
sogan = "今年刚满" + str(num) + "岁~"
print(sogan)
```
* 字符串格式化   
```py
name = "芜柒"
age = 18
# 方式1(和C比较像):
print("我是%s,今年刚满%d岁~" % (name,age))
# 方式2(推荐):  
print(f"我是{name},今年刚满{age}岁~")
```
* 常用方法  

| 方法        | 作用                                                     | 样例                  |
| ----------- | -------------------------------------------------------- | --------------------- |
| find()      | 在字符串中查找子串，返回第一次出现的索引位置，找不到返回-1 | s.find('Python')      |
| count()     | 统计子串在字符串中出现的次数                             | s.count('H')          |
| upper()     | 将字符串中的所有字母转换为大写                           | s.upper()             |
| lower()     | 将字符串中的所有字母转换为小写                           | s.lower()             |
| split()     | 将字符串按指定分隔符分割成列表                           | s.split(' ')          |
| strip()     | 去除字符串两端的空白字符或指定字符                       | s.strip() / s.strip('*') |
| replace()   | 将字符串中的指定子串替换为新的子串                       | s.replace('H','C')    |
| startswith()| 检查字符串是否以指定子串开头，返回布尔值                  | s.startswith('P')     |
### tuple元组
可以理解为不能修改的列表
```py
t0 = (1,) # 单元素元组定义 最后有个,
t1 = (1, "hello", None)
```
* 方法
```py
t1 = ( 1, 1, 1)
print(t1.count(1)) # 3
print(t1.index(1)) # 0
```
* 解包
```py
t = (1 ,2 ,3)
a, b, c = t # 1 2 3
*d, e = t   # [1,2] 3
```
### set集合
无序 不可重复 可修改
```py
set0 = set()#定义空集合 要显示声明 否则会定义为字典
set1 = {1, 2, 3}
```
* 常用方法  

| 操作           | 含义                                                         | 样例                |
| -------------- | ------------------------------------------------------------ | ------------------- |
| add(..)        | 添加元素到集合中                                             | s1.add('t')         |
| remove(..)     | 移除集合中的指定元素（指定元素不存在将报错）| s1.remove('t')      |
| pop()          | 随机删除集合中的元素并返回                                   | e = s1.pop()        |
| clear()        | 清空集合                                                     | s1.clear()          |
| difference()   | 求取两个集合的差集（包含在第一个集合但不包含在第二个集合的元素） | s1.difference(s2)   |
| union()        | 求取两个集合的并集                                           | s1.union(s2)        |
| intersection() | 求取两个集合的交集                                           | s1.intersection(s2) |
### dict字典
可以理解为java里面的map
```py
# key必须是不可变类型 str int tuple
scoreDict = {"芜柒":100 , "王梧": 99}
```
* 常用方法  

| 类型 | 操作                  | 含义                                   | 样例                  |
| ---- | --------------------- | -------------------------------------- | --------------------- |
| 添加 | 字典名称[key] = value | 往指定字典中添加key-value键值对        | dict1["涛哥"] = 688   |
| 删除 | 字典名称.pop(key)     | 删除字典中指定的key，并返回该key对应的value | score = dict1.pop("涛哥") |
| 删除 | del 字典名称[key]     | 删除字典中指定的键值对                 | del dict1["涛哥"]     |
| 修改 | 字典名称[key] = value | 修改字典中指定的key对应的值            | dict1["小智"] = 658   |
| 查询 | 字典名称[key]         | 根据key获取value                       | dict1["涛哥"]         |
| 查询 | 字典名称.get(key)     | 根据key获取value                       | dict1.get("涛哥")     |
| 查询 | 字典名称.keys()       | 获取所有的key                          | dict1.keys()          |
| 查询 | 字典名称.values()     | 获取所有的value                        | dict1.values()        |
| 查询 | 字典名称.items()      | 获取所有的key-value键值对              | dict1.items()         |
## 输入输出
```py
str = input("请输入文本")
ptint(f"用户输入{str}")
```
## 运算符
这里只强调特性  
### 算数运算
* / 除法 输出小数(整除也是小数)  
* // 整除   
* ** 幂  
```py
a = 10
b = 3
print(a/b)  # 3.33333333333
print(a//b) # 3
print(a**b) # 1000
```
### 逻辑运算
* and (&&)  
* or  (||)  
* not (!)  
## 流程控制
* if else elif  
```py
# 缩进很重要
if a > b:
    print("a大于b")
    print("这是多行输出")
elif a == b:
    print("a等于b")
else:
    print("a小于b")
```
* match case  
就是switch case
```python
num1 = float(input("请输入数字1"))
num2 = float(input("请输入数字2"))
oper = input("请输入计算符号")
match:
    case "+" | "-":
        print("执行加减")
    case "/" if num2 != 0 # 条件符合才匹配
        print(f"{num1} / {num2} = {num1 / num2}")

    case _:
        pass # 什么也不做

```
```java
    int day = 3;
    switch (day) {
        case 1 :
            sout("周一")
            break;
        default :
            sout("其他");

    }
    //Java 14+ 增强 switch（
    int day = 3;
    String result = switch (day) {
        case 1 -> "周一";
        case 2 -> "周二";
        case 3 -> "周三";
        default -> "其他日期";
```
* while
```py
num = 1
while num<=10 :
    print(num)
    num += 1
else: 
    print("循环结束")
```
* for
```py
str = "Life is short, you need Python!"
for i in str :
    print(i)
else :
    print("循环结束")
```
## 常用函数
* range
```py
# 1-100所有偶数
# start(默认:0 >=) end(<) step(默认:1)
for num in range(0,101,2):
  print(num)
```
* sum min max len
```py
list = [1 ,2 ,3]
sum(list)
```